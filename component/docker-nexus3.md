#Docker-Nexus3配置
## 1.nexus3安装与配置
### (1)nexus3 docker hub地址
```shell
https://registry.hub.docker.com/r/sonatype/nexus3
```
### (2)配置nexus3存储空间
```shell
docker volume create --name nexus-data
```
### (3)安装镜像
```shell
mkdir -p /var/jenkins_home
chmod 777 /var/jenkins_home
docker run -d --name mjenkins -p 10240:8080 -p 10241:50000 --network jenkins \
    -e JAVA_OPTS=-Duser.timezone=Asia/Shanghai \
    -v /etc/localtime:/etc/localtime \
    -v /usr/bin/docker:/usr/bin/docker \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /var/jenkins_home:/var/jenkins_home  \
    jenkins/jenkins:lts-jdk11
# -v /var/jenkins_home:/var/jenkins_home目录为容器jenkins工作目录，我们将硬盘上的一个目录挂载到这个位置，方便后续更新镜像后继续使用原来的工作目录。这里我们设置的就是上面我们创建的 /var/jenkins_home目录
# -v /etc/localtime:/etc/localtime让容器使用和服务器同样的时间设置。
# -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock   设置docker
# 不需要使用默认自带jdk11 -v /usr/bin/mv:/usr/bin/mv -v /usr/local/java/jdk1.8.0_271/bin:/usr/local/java/jdk1.8.0_271/bin 设置jdk
# -v /usr/local/maven3.6:/usr/local/maven3.6  设置maven



```
### (4)访问jenkins
```shell
http://120.53.120.254:10240
```
### (5)切换国内jenkins镜像
因为国外镜像经常出现下载失败的问题，所以切换为国内镜像。
方法一
```shell
vim /var/lib/docker/volumes/jenkins_home/_data/hudson.model.UpdateCenter.xml
# 将镜像地址改为如下
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
# 保存后重启 jenkins
docker restart mjenkins
```
方法二
系统管理 --> 插件管理 --> Advanced --> 高级选项卡 --> 更新网站
更新为上述镜像地址


