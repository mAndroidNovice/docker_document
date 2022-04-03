# Docker-Nexus3配置

## 1.nexus3安装与配置

### (1)nexus3 docker hub地址

```shell
https://registry.hub.docker.com/r/sonatype/nexus3
```

nexus3 System配置要求

```shell
https://help.sonatype.com/repomanager3/product-information/system-requirements
```

### (2)安装镜像

```shell
#  configuration, logs, and storage
mkdir -p /var/nexus-data
chmod 777 /var/nexus-data
docker run -d --name mnexus3 -p 11241:8081 -p 11242:8082 -p 11243:8083 -p 11244:8084 -p 11245:8085 \
-e NEXUS_CONTEXT=nexus \
-e INSTALL4J_ADD_VM_PARAMS="-Xms2703m -Xmx2703m -XX:MaxDirectMemorySize=2703m -Djava.util.prefs.userRoot=/var/nexus-data/javaprefs" \
-v /var/nexus-data:/nexus-data \
sonatype/nexus3:3.38.1
# NEXUS_CONTEXT=nexus  Nexus Context Path 默认 /
# INSTALL4J_ADD_VM_PARAMS vm最低配置详情查看(1) userRoot-> persistent path
# /var/nexus-data  Persistent Data -> configuration, logs, and storage
# 内存不够大的学生机不要头铁 改小点 java内存使用量
docker run -d --name mnexus3 -p 11241:8081 -p 11242:8082 -p 11243:8083 -p 11244:8084 -p 11245:8085 \
-e NEXUS_CONTEXT=nexus \
-e INSTALL4J_ADD_VM_PARAMS="-Xms512m -Xmx512m -XX:MaxDirectMemorySize=512m -Djava.util.prefs.userRoot=/var/nexus-data/javaprefs" \
-v /var/nexus_data:/nexus-data \
sonatype/nexus3:3.38.1
```

### (3)访问jenkins

```shell
# nexus 即为 NEXUS_CONTEXT
curl http://localhost:11240/nexus
# 查看密码
cat /var/nexus_data/admin.password |grep -v "^$"
```

### (4)停止

因为nexus比较慢 关闭时需要加一点延迟 防止数据丢失

```shell
docker stop --time=120 <CONTAINER_NAME>
```

### (5) maven配置settings

配置仓库地址 是public group仓库地址而不是releases或snapshots仓库，public默认包含了这两个仓库

```xml

<settings>
    <profiles>
        <profile>
            <id>dev</id>
            <repositories>
                <repository>
                    <id>local-nexus</id>
                    <url>http://ip:10241/repository/maven-public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
        </profile>
    </profiles>

    <activeProfiles>
        <activeProfile>dev</activeProfile>
    </activeProfiles>
</settings>
```

配置maven settings文件的服务器用户名密码 注意：id为私服中releases和snapshots仓库名，必须一致

```xml

<servers>
    <server>
        <id>maven-releases</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
    <server>
        <id>maven-snapshots</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
</servers>
```

在项目父pom文件中配置部署环境，注意id及URL必须与nexus仓库对应

```xml
<!--私服仓库-->
<distributionManagement>
    <repository>
        <id>maven-releases</id>
        <name>Nexus Release Repository</name>
        <url>http://ip:10241/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>maven-snapshots</id>
        <name>Nexus Snapshot Repository</name>
        <url>http://ip:10241/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

重新打开项目，对需要的模块进行deploy