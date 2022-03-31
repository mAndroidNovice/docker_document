## Dockerfile
https://blog.csdn.net/zisefeizhu/article/details/83472190



| 关键字      | 作用                     | 备注                                                         |
| ----------- | ------------------------ | ------------------------------------------------------------ |
| FROM        | 指定父镜像               | 指定dockerfile基于那个image构建                              |
| MAINTAINER  | 作者信息                 | 用来标明这个dockerfile谁写的                                 |
| LABEL       | 标签                     | 用来标明dockerfile的标签 可以使用Label代替Maintainer 最终都是在docker image基本信息中可以查看 |
| RUN         | 执行命令                 | 执行一段命令 默认是/bin/sh 格式: RUN command 或者 RUN ["command" , "param1","param2"] |
| CMD         | 容器启动命令             | 提供启动容器时候的默认命令 和ENTRYPOINT配合使用.格式 CMD command param1 param2 或者 CMD ["command" , "param1","param2"] |
| ENTRYPOINT  | 入口                     | 一般在制作一些执行就关闭的容器中会使用                       |
| COPY        | 复制文件                 | build的时候复制文件到image中                                 |
| ADD         | 添加文件                 | build的时候添加文件到image中 不仅仅局限于当前build上下文 可以来源于远程服务 |
| ENV         | 环境变量                 | 指定build时候的环境变量 可以在启动的容器的时候 通过-e覆盖 格式ENV name=value |
| ARG         | 构建参数                 | 构建参数 只在构建的时候使用的参数 如果有ENV 那么ENV的相同名字的值始终覆盖arg的参数 |
| VOLUME      | 定义外部可以挂载的数据卷 | 指定build的image那些目录可以启动的时候挂载到文件系统中 启动容器的时候使用 -v 绑定 格式 VOLUME ["目录"] |
| EXPOSE      | 暴露端口                 | 定义容器运行的时候监听的端口 启动容器的使用-p来绑定暴露端口 格式: EXPOSE 8080 或者 EXPOSE 8080/udp |
| WORKDIR     | 工作目录                 | 指定容器内部的工作目录 如果没有创建则自动创建 如果指定/ 使用的是绝对地址 如果不是/开头那么是在上一条workdir的路径的相对路径 |
| USER        | 指定执行用户             | 指定build或者启动的时候 用户 在RUN CMD ENTRYPONT执行的时候的用户 |
| HEALTHCHECK | 健康检查                 | 指定监测当前容器的健康监测的命令 基本上没用 因为很多时候 应用本身有健康监测机制 |
| ONBUILD     | 触发器                   | 当存在ONBUILD关键字的镜像作为基础镜像的时候 当执行FROM完成之后 会执行 ONBUILD的命令 但是不影响当前镜像 用处也不怎么大 |
| STOPSIGNAL  | 发送信号量到宿主机       | 该STOPSIGNAL指令设置将发送到容器的系统调用信号以退出。       |
| SHELL       | 指定执行脚本的shell      | 指定RUN CMD ENTRYPOINT 执行命令的时候 使用的shell            |



### FROM
FROM指令是最重要的一个且必须为 Dockerfile文件开篇的第一个非注释行，用于为映像文件构建过程指定基准镜像，后续的指令运行于此基准镜像所提供的运行环境 .
实践中，基准镜像可以是任何可用镜像文件，默认情况下， docker build会在 docker主机上查找指定的镜像文件，在其不存在时，则会从 Docker Hub Registry上拉取所需的镜像文件 .如果找不到指定的镜像文件， docker build会返回一个错误信息

FROM 语法
FROM <repository>[:<tag>] 或者
FROM <repository>@<digest>
<repository>:指定作为base image的名称
<tag>：base image的标签，为可选项，省略时默认为 latest；
<digest>为校验码

### LABEL
LABEL用于为镜像添加元数据，元数以键值对的形式指定：
```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```
使用LABEL指定元数据时，一条LABEL指定可以指定一或多条元数据，指定多条元数据时不同元数据之间通过空格分隔。推荐将所有的元数据通过一条LABEL指令指定，以免生成过多的中间镜像。
如，通过LABEL指定一些元数据：
```
LABEL version="1.0" description="这是一个Web服务器" by="IT笔录"
```
指定后可以通过docker inspect查看：
```
docker inspect itbilu/test
"Labels": {
    "version": "1.0",
    "description": "这是一个Web服务器",
    "by": "IT笔录"
},
```

### COPY
用于从 Docker主机复制文件至创建的新映像文件，不能解压且只能操作本地文件
语法
COPY <src> ... <dest>或 . COPY ["<src>",... "<dest>"]

<src>：要复制的源文件或目录，支持使用通配符
 <dest>：目标路径，即正在创建的 image的文件系统路径；建议为 <dest>使用绝对路径，<dest>绝对路径为镜像中的路径，而不是宿主机的路径。否则， COPY指定则以 WORKDIR为其起始路径

注意：在路径中有空白字符时，通常使用第二种格式 .

文件复制准则

<src>必须是build上下文中的路径，即只能放在workshop这个工作目录下，不能是其父目录中的文件
如果<src>是目录，其内部文件或者子目录会被递归复制，但<src>目录自身不会被复制
如果指定了多个<src>，或在<src>中使用了通配符，则<dest>必须是一个目录，且dest目录必须以/结尾
如果<dest>事先不存在，它将会被自动创建，这包括其父目录路径

#### 例子
创建一个目录img1，在该目录下新建index.html文件用于镜像制作的素材文件，在img1下新建Dockerfile文件，把index.html拷贝到新镜像里

copy是指在当前的img1工作目录中，准备好要添加到新镜像的文件放到这个img1下面，copy过程实际是基于dockerfile在后台启动一个容器，把工作目录当做卷挂载到后台启动的容器，然后再把这些准备好的文件（img1目录下）拷贝到后台容器，然后基于这个容器制作新镜像，所以，镜像的制作过程是基于指定的镜像来制作


```
[root@node1 ~]# mkdir img1/
[root@node1 ~]# cd img1/
[root@node1 img1]# ls
[root@node1 img1]# vim index.html
<h1>zhujingxing </h1>
[root@node1 img1]# vim Dockerfile
# Description: test image
FROM busybox:latest
MAINTAINER "zisefeizhu <zisefeizhu@zhujingxing.com>"
#LABEL maintainer="zisefeizhu <zisefeizhu@zhujingxing.com>"
COPY index.html /data/web/html/
```

### ADD
ADD指令类似于 COPY指令， ADD支持使用 TAR文件和 URL路径,可以解压
语法
. ADD <src> ... <dest>或
. ADD ["<src>",... "<dest>"]
.操作准则 .
同COPY指令的4点准则
如果<src>为URL且<dest>不以/结尾，则<src>指定的文件将被下载并直接被创建为<dest>；如果<dest>以/结尾，则文件名URL指定的文件将被直接下载，并保存为<dest>/<filename>，注意，URL不能是ftp格式的url
如果<src>是一个本地系统上的压缩格式的tar文件，它将被展开为一个目录，其行为类似于“tar -x”命令，然后，通过URL获取到的tar文件将不会自动展开
如果<src>有多个，或其间接或直接使用了通配符，则<dest>必须是一个以/结尾的目录路径；如果<dest>不以/结尾，则其被视作一个普通文件，<src>的内容将被直接写入到<dest>；

### WORKDIR
workdir为工作目录，指当前容器环境的工作目录，用于为 Dockerfile中所有的 RUN、CMD、ENTRYPOINT、COPY和 ADD指定设定工作目录
语法
 WORKDIR  <dirpath>
在Dockerfile文件中， WORKDIR指令可出现多次，其路径也可以为相对路径，不过，其是相对此前一个 WORKDIR指令指定的路径
另外， WORKDIR也可调用由 ENV指定定义的变量 .例如
WORKDIR /var/log
WORKDIR  $STATEPATH

#### 例子
指定workdir为/usr/local，相当于是容器启动后，会把工作目录切换到/usr/local这个workdir路径下，而不是默认的根目录，如下例子，则相对路径 ./src/ 的绝对路径为容器的/usr/local/src，制作镜像时，把nginx包拷贝到/usr/local/src，把tomcat包解压到/usr/local/src下面
```
[root@node1 img1]# vim Dockerfile
FROM busybox:1.27.2
MAINTAINER "zisefeizhu <zisefeizhu@zhujingxing.com>"
WORKDIR "/usr/local"
ADD http://nginx.org/download/nginx-1.14.0.tar.gz  ./src/
ADD apache-tomcat-8.0.47.tar.gz ./
```
### VOLUME
定义卷，只能是docker管理的卷，VOLUME为容器上的目录，用于在 image中创建一个挂载点目录，以挂载 Docker host上的卷或其它容器上的卷
语法
. VOLUME <mountpoint>或
. VOLUME ["<mountpoint>"]
如果挂载点目录路径下此前在文件存在， docker run命令会在卷挂载完成后将此前的所有文件复制到新挂载的卷中

#### 例子
```
[root@node1 ~]# vim img1/Dockerfile
#ADD http://nginx.org/download/nginx-1.15.5.tar.gz /usr/local/src/
WORKDIR /usr/locl/
ADD nginx-1.15.5.tar.gz ./src
VOLUME /data/mysql/
```

### EXPOSE
暴露指定端口，用于为容器打开指定要监听的端口以实现与外部通信
语法
EXPOSE <port>[/<protocol>] [<port>[/<protocol>] ...] l
其中<protocol>用于指定传输层协议，可为 tcp或udp二者之一，默认为 TCP协议
EXPOSE指令可一次指定多个端口，但是不能指定暴露为宿主机的指定端口，因为指定的宿主机端口可能已经被占用，因此这里使用随机端口，例如
. EXPOSE 11211/udp 11211/tcp

### ENV
ENV用于为镜像定义所需的环境变量，并可被 Dockerfile文件中位于其后的其它指令（如 ENV、ADD、COPY等）所调用 ，即先定义后调用
调用格式为 $variable_name或${variable_name}
语法
ENV <key> <value>或 . ENV <key>=<value> ... .
第一种格式中， <key>之后的所有内容均会被视作其 <value>的组成部分，因此一次只能设置一个变量
第二种格式，可用一次设置多个变量，每个变量为一个“<key>=<value>”的键值对，如果<value>包含空格，可以以反斜线（\）进行转义，也可通过对<value>加引号进行标识；另外反斜线也可以用于续行；
.定义多个变量时，建议使用第二种方式，以便在同一层中完成所有功能

### RUN
RUN用于指定 docker build过程中运行的程序，其可以是任何命令，但是这里有个限定，一般为基础镜像可以运行的命令，如基础镜像为centos，安装软件命令为yum而不是ubuntu里的apt-get命令

RUN和CMD都可以改变容器运行的命令程序，但是运行的时间节点有区别，RUN表示在docker build运行的命令，而CMD是将镜像启动为容器运行的命令。因为一个容器正常只用来运行一个程序，因此CMD一般只有一条命令，如果CMD配置多个，则只有最后一条命令生效。而RUN可以有多个。

语法
RUN <command>或 . RUN ["<executable>", "<param1>", "<param2>"]

第一种格式中，<command>通常是一个shell命令，且以“/bin/sh -c”作为父进程来运行它，这意味着此进程在容器中的PID不为1，不能接收Unix信号，因此，当使用 docker stop <container>命令停止容器时，此进程接收不到SIGTERM信号；

第二种语法格式中的参数是一个JSON格式的数组，其中<executable>为要运行的命令，后面的<paramN>为传递给命令的选项或参数；然而，此种格式指定的命令不会以“/bin/sh -c”来发起，表示这种命令在容器中直接运行，不会作为shell的子进程，因此常见的shell操作如变量替换以及通配符（？，*等）替换将不会进行，不过，如果要运行的没能力依赖此shell特性的话，可以将其替换为类似下面的格式
注意:json数组中使用双引号
RUN ["/bin/bash","-C","<executable>","<paraml>"]

#### 例子

```
[root@node1 img1]# vim Dockerfile

# Description: test image
FROM busybox:latest
MAINTAINER "zisefeizhu <zisefeizhu@zhujingxing.com>"
#LABEL maintainer="zisefeizhu <zisefeizhu@zhujingxing.com>"
ENV  DOC_ROOt=/data/web/html/ \
     WEB_SERVER_PACKAGE="nginx-1.15.5.tar.gz"

COPY index.html ${DOC_ROOT:-/data/web/html/}
COPY yum.repos.d /etc/yum.repos.d/
ADD http://nginx.org/download/${WEB_SERVER_PACKAGE} /usr/local/src/
WORKDIR /usr/local/
#ADD ${WEB_SERVER_PACKAGE}.tar.gz ./src/
VOLUME /data/mysql/
EXPOSE 80/tcp
RUN cd /usr/local/src && \
    tar xf ${WEB_SERVER_PACKAGE}
```

### CMD
类似于 RUN指令， CMD指令也可用于运行任何命令或应用程序，不过，二者的运行时间点不同 . RUN指令运行于映像文件构建过程中，而 CMD指令运行于基于 Dockerfile构建出的新映像文件启动一个容器时 . CMD指令的首要目的在于为启动的容器指定默认要运行的程序，且其运行结束后，容器也将终止；不过， CMD指定的命令其可以被 docker run的命令行选项所覆盖 .在Dockerfile中可以存在多个 CMD指令，但仅最后一个会生效

语法
CMD <command>或
CMD ["<executable>","<param1>","<param2>"]或
CMD ["<param1>","<param2>"]

.前两种语法格式的意义同 RUN
.第三种则用于为 ENTRYPOINT指令提供默认参数

### ENTRYPOINT
类似 CMD指令的功能，用于为容器指定默认运行程序，从而使得容器像是一个单独的可执行程序

与CMD不同的是，由 ENTRYPOINT启动的程序不会被 docker run命令行指定的参数所覆盖，而且，这些命令行参数会被当作参数传递给 ENTRYPOINT指定指定的程序 .不过， docker run命令的 --entrypoint选项的参数可覆盖ENTRYPOINT指令指定的程序

语法

ENTRYPOINT <command>

ENTRYPOINT ["<excutable>","<param1>","<param2>"]

docker run 命令传入的命令参数会覆盖CMD指令的内容并且附加到ENTRYPOINT命令最后做为其参数使用 . Dockerfile文件中也可以存在多个 ENTRYPOINT指令，但仅有最后一个会生效
```
[root@node1 img3]# vim Dockerfile

FROM nginx:1.14-alpine
LABEL maintainer="zhujingxing  <zisefeizhu@zhujingxing>"
ENV NGX_DOC_ROOT='/data/web/html/'
ADD index.html ${NGX_DOC_ROOT}
ADD entrypoint.sh /bin/
CMD ["/usr/sbin/nginx","-g","daemon off;"]     //注：双引号
ENTRYPOINT ["/bin/entrypoint.sh"]
```

### HEALTHCHECK:docker容器运行健康检查

语法形式:

HEALTHCHECK [OPTIONS] CMD command (通过在容器中运行一个命令执行健康检查)
HEALTHCHECK NONE (禁用从基本镜像继承的任何健康检查)
通过HEALTHCHECK，我们可以知道如何测试一个容器查检一个它是否在工作，比如检测一个web 服务是否陷入死循环，不能处理新的连接、即使服务器进程仍在运行

当一个窗口指定了健康检查时、除了正常状态之外、还会有一个健康状态作为初始、如果检查通过、则会变成健康状态、如果经过了一定次数的连续故障、则会变成非健康状态

在CMD之前可以出现的选项如下:

--interval=DURATION (default: 30s)
--timeout=DURATION (default: 30s)
--start-period=DURATION (default: 0s)
--retries=N (default: 3)
注：

启动周期为需要时间启动的容器提供初始化时间。 在此期间的探测失败不会计入最大重试次数。 但是，如果在启动期间运行状况检查成功，则认为容器已启动，并且所有连续的故障都将计入最大重试次数
单次运行检查花费时间超过timeout指定时间、判定失败
每个Dockerfile中只能存在一个HEALTHCHECK指令，如果有多个则最后一个起作用

HEALTHCHECK CMD后面的命令既可以是一个shell命令、也可以是一个exec 的数组

命令的退出状态显示出容器的健康状态、如下：

0: success - the container is healthy and ready for use

1: unhealthy - the container is not working correctly

2: reserved(保留的) - do not use this exit code

实例:每隔五分钟检查一次网络服务器是否能够在三秒钟内为该网站的主页面提供服务

HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1
为方便故障探测调试、检测命令通过stdout或者stderr输出的文本都会被缓存在健康状态中(缓存大小为4096字节)、并可以通过docker inspect查询

当容器的运行状况发生变化时，新的状态会生成一个health_status事件
#### 成功的例子
```
[root@node1 img3]# cat Dockerfile
FROM nginx:1.14-alpine
LABEL maintainer="zhujingxing  <zisefeizhu@zhujingxing>"
ENV NGX_DOC_ROOT='/data/web/html/'
ADD index.html ${NGX_DOC_ROOT}
ADD entrypoint.sh /bin/
EXPOSE 80/TCP
HEALTHCHECK --start-period=3s CMD wget -O - -q http://${IP:-0.0.0.0}:${PORT:80}/
CMD ["/usr/sbin/nginx","-g","daemon off;"]
ENTRYPOINT ["/bin/entrypoint.sh"]
[root@node1 img3]# docker build -t myweb:v0.3-7 ./
[root@node1 img3]# docker run --name myweb1 --rm -P -e "PORT=8080" myweb:v0.3-7

[root@node1 ~]# docker exec -it myweb1 /bin/sh

/ # netstat -tnl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
/ # wget -O

[root@node1 img3]# docker run --name myweb1 --rm -P -e "PORT=8080" myweb:v0.3-7
127.0.0.1 - - [28/Oct/2018:13:15:35 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
127.0.0.1 - - [28/Oct/2018:13:15:43 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
```
#### 失败的例子
```
[root@node1 img3]# vim Dockerfile

FROM nginx:1.14-alpine
LABEL maintainer="zhujingxing  <zisefeizhu@zhujingxing>"
ENV NGX_DOC_ROOT='/data/web/html/'
ADD index.html ${NGX_DOC_ROOT}
ADD entrypoint.sh /bin/
EXPOSE 80/TCP
HEALTHCHECK --start-period=3s CMD wget -O - -q http://${IP:-0.0.0.0}:10080/
CMD ["/usr/sbin/nginx","-g","daemon off;"]
ENTRYPOINT ["/bin/entrypoint.sh"]
[root@node1 img3]# docker build -t myweb:v0.3-8 ./

[root@node1 img3]# docker run --name myweb1 --rm -P -e "PORT=8080" myweb:v0.3-8 //三个周期默认1.5分钟后报错
[root@node1 ~]# docker exec -it myweb1 /bin/sh
/ # netstat -tnl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
```

### ARG
ARG用于指定传递给构建运行时的变量：

ARG <name>[=<default value>]
如，通过ARG指定两个变量：

ARG site
ARG build_user=IT笔录
以上我们指定了 site 和 build_user 两个变量，其中 build_user 指定了默认值。在使用 docker build 构建镜像时，可以通过 --build-arg <varname>=<value> 参数来指定或重设置这些变量的值。

docker build --build-arg site=itiblu.com -t itbilu/test .
这样我们构建了 itbilu/test 镜像，其中site会被设置为 itbilu.com，由于没有指定 build_user，其值将是默认值 IT 笔录。

### STOPSIGNAL
STOPSIGNAL用于设置停止容器所要发送的系统调用信号：

STOPSIGNAL signal
所使用的信号必须是内核系统调用表中的合法的值，如：SIGKILL。

### SHELL
SHELL用于设置执行命令（shell式）所使用的的默认 shell 类型：

SHELL ["executable", "parameters"]

SHELL在Windows环境下比较有用，Windows 下通常会有 cmd 和 powershell 两种 shell，可能还会有 sh。这时就可以通过 SHELL 来指定所使用的

shell 类型：
```
FROM microsoft/windowsservercore

# Executed as cmd /S /C echo default
RUN echo default

# Executed as cmd /S /C powershell -command Write-Host default
RUN powershell -command Write-Host default

# Executed as powershell -command Write-Host hello
SHELL ["powershell", "-command"]
RUN Write-Host hello

# Executed as cmd /S /C echo hello
SHELL ["cmd", "/S", "/C"]
RUN echo hello
```

### USER
USER用于指定运行 image时的或运行 Dockerfile中任何 RUN、CMD或 ENTRYPOINT指令指定的程序时的用户名或 UID ，即改变容器中运行程序的身份

.默认情况下， container的运行身份为 root用户

语法
USER  <UID>|<UserName>
需要注意的是， <UID>可以为任意数字，但实践中其必须为 /etc/passwd中某用户的有效 UID，否则， docker run命令将运行失败

使用USER指定用户时，可以使用用户名、UID 或 GID，或是两者的组合。以下都是合法的指定试：
```
USER user
USER user:group
USER uid
USER uid:gid
USER user:gid
USER uid:group
```
用USER指定用户后，Dockerfile 中其后的命令 RUN、CMD、ENTRYPOINT 都将使用该用户。镜像构建完成后，通过 docker run 运行容器时，可以通过 -u 参数来覆盖所指定的用户
