## Docker 应用部署

### 一、部署MySQL

1. 搜索mysql镜像

```shell
docker search mysql
```

2. 拉取mysql镜像

```shell
docker pull mysql:5.7
```
```shell
docker pull 8.0.28
```

3. 创建容器，设置端口映射、目录映射
```shell
# 在/root目录下创建mysql目录用于存储mysql数据信息
mkdir ~/mysql
cd ~/mysql
```

```shell
docker run -id \
-p 3306:3306 \
--name=c_mysql \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/logs:/logs \
-v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.6 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci
```

- 参数说明：
  - **-p 3307:3306**：将容器的 3306 端口映射到宿主机的 3307 端口。
  - **-v $PWD/conf:/etc/mysql/conf.d**：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。配置目录
  - **-v $PWD/logs:/logs**：将主机当前目录下的 logs 目录挂载到容器的 /logs。日志目录
  - **-v $PWD/data:/var/lib/mysql** ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。数据目录
  - **-e MYSQL_ROOT_PASSWORD=123456：**初始化 root 用户的密码。
  - **--rm 容器停止自动清理容器内部的文件内容
  - **-u, --user string 指定容器的用户 (格式: <name|uid>[:<group|gid>]),不指定时，默认以root用户执行 --user 1000:1000
  - **-w, --workdir string 指定工作目录
  -m, --memory="" # 指定容器的内存上限
  -h, --hostname="" # 指定容器的主机名


4. 进入容器，操作mysql

```shell
docker exec –it c_mysql /bin/bash
```

5. docker hub

https://registry.hub.docker.com/_/mysql?tab=description

6. create mysql dumps
``` shell
docker exec some-mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /some/path/on/your/host/all-databases.sql
```

7. restore dumps
```shell
docker exec -i some-mysql sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < /some/path/on/your/host/all-databases.sql
```
