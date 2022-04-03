#Docker-Jenkins配置
## 1.jenkins安装与配置
### (1)镜像github官网
```shell
https://github.com/jenkinsci/docker
```
### (2)配置jenkins专用网络,如果重复了就替换一个
```shell
docker network create -d bridge --subnet 172.19.0.0/16 --gateway 172.19.0.1 jenkin
```
### (3) 安装一个本地maven
下载修改地址
```shell
wget https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz 
tar zxvf apache-maven-3.8.5-bin.tar.gz
mv apache-maven-3.8.5 /usr/local/maven3
```
配置aliyun镜像 vim /usr/local/maven3/conf/settings.xml
```xml
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

### (3)安装镜像
```shell
mkdir -p /var/jenkins_home
chmod 777 /var/jenkins_home
docker run -d --name mJenkins -p 10240:8080 -p 10241:50000 --network jenkins \
    -e JAVA_OPTS=-Duser.timezone=Asia/Shanghai \
    -v /etc/localtime:/etc/localtime \
    -v /usr/bin/docker:/usr/bin/docker \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /var/jenkins_home:/var/jenkins_home  \
    -v /usr/local/maven3:/usr/local/maven3 \
    jenkins/jenkins:lts-jdk11
# -v /var/jenkins_home:/var/jenkins_home目录为容器jenkins工作目录，我们将硬盘上的一个目录挂载到这个位置，方便后续更新镜像后继续使用原来的工作目录。这里我们设置的就是上面我们创建的 /var/jenkins_home目录
# -v /etc/localtime:/etc/localtime让容器使用和服务器同样的时间设置。
# -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock   设置docker
# 不需要使用 默认自带jdk11 -v /usr/bin/mv:/usr/bin/mv -v /usr/local/java/jdk1.8.0_271/bin:/usr/local/java/jdk1.8.0_271/bin 设置jdk
# -v /usr/local/maven3.6:/usr/local/maven3.6  设置maven

```
————————————————
版权声明：本文为CSDN博主「梦想俱乐部」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_49724150/article/details/115613101
### (4)切换国内jenkins镜像
因为国外镜像经常出现下载失败的问题，所以切换为国内镜像。 下载速度嗖嗖的!
```shell
cd /var/jenkins_home
# 将镜像地址改为 https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
sed -i 's/https:\/\/updates.jenkins.io\/update-center.json/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins\/updates\/update-center.json/g' /var/jenkins_home/hudson.model.UpdateCenter.xml
# 修改updates文件夹中的 default.json文件
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /var/jenkins_home/updates/default.json
sed -i 's/http:\/\/www.google.com/http:\/\/www.baidu.com/g' /var/jenkins_home/updates/default.json

# 重载配置
http:ip:10240/reload
# 或者 
http:ip:10240/restart
```


### (5)访问jenkins
```shell
http://ip:10240
# 查看第一次登录密码
docker exec -it mJenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### (6) 常见问题
#### 1 进入容器后发现无法使用docker
```shell
docker exec -it mJenkins /bin/bash
docker ps -a
```
发现报错如下
Got permission denied while trying to connect to the Docker daemon
##### 解决方法
```shell
# 以root用户登录过
docker exec -it -u mJenkins /bin/bash
# 创建docker用户组
groupadd docker
# jenkins用户添加到用户组中
usermod -a -G docker jenkins
# 给docker.sock添加授权
chmod 777 /var/run/docker.sock
```
后续使用jenkins用户登录即可使用