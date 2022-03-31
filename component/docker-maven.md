#Docker-Maven配置
## 1.Maven安装与配置
### (1)Maven docker hub地址
```shell
https://hub.docker.com/_/maven
```
### (2) 使用maven打包程序
在maven项目的根目录即pom文件所在目录
自动进行打包 ./target/xxxx.jar
```shell
# 使用一次 即时使用 即时销毁 每次执行都需要重新下载jar包
docker run -it --rm --name maven-project \
      -v "$(pwd)":/usr/src/mymaven \
      -w /usr/src/mymaven \
      maven:3.8.4-jdk-11 \
      mvn clean install
```
### (3) maven仓库本地挂载
```shell
# will download artifacts
docker run -it -v maven-repo:/root/.m2 maven mvn archetype:generate
```

### (4) 自定义setting.xml
```shell
# 运行镜像 自动下载pom.xml中的jar包
COPY pom.xml /tmp/pom.xml
RUN mvn -B -f /tmp/pom.xml -s /usr/share/maven/ref/settings-docker.xml dependency:resolve
# 自己设置settings的目录
COPY settings.xml /usr/share/maven/ref/
```
### (5)使用非root用户运行
使用非root用户需要对操作目录进行指定，否则会出现权限异常
-v 创建新的目录 
-e 配置maven配置目录为 -v
```shell
# 新建用户1000
docker run -it --rm -u 1000 \
      -v ~/.m2:/var/maven/.m2 \
      -e MAVEN_CONFIG=/var/maven/.m2 \
      maven \
      mvn -Duser.home=/var/maven archetype:generate
```