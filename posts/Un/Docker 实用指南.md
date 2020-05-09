# docker 实用指南

 Docker 封装好的四大对象：

- 镜像 ( Image )
- 容器 ( Container )
- 网络 ( Network )
- 数据卷 ( Volume )



## 运行环境

```shell
$ docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
```



## 镜像

命名规则分为三个部分

- **username**： 主要用于识别上传镜像的不同用户，与 GitHub 中的用户空间类似。
- **repository**：表示是哪个具体的镜像，一般会使用应用命名，比如 Mysql 等。
- **tag**：表示镜像的某个版本。

![](https://i.loli.net/2020/05/08/IgrokDUCcvFOAw8.png)

```sh
# 显示所有镜像
docker images

# 拉取镜像
docker pull ubuntu

# 使用完整镜像名
docker pull openresty/openresty:1.13.6

# 显示镜像完整信息
docker inspect ubuntu 

# 搜索镜像
docker search ubuntu

# 删除镜像
docker rmi ubuntu:14.04 

# 添加 tag
docker tag ubuntu:14.04 ub14
```

```shell
docker commit -m'' -a'' id nevermoes/res:tag #提交,将容器转化为镜像
docker push test:latest

docker save -o ubuntu:14.04 ub14.tar #存出镜像
docker load < ub14.tar #导入镜像
```



## 容器

### 生命周期

- **Created**：容器已经被创建，容器所需的相关资源已经准备就绪，但容器中的程序还未处于运行状态。
- **Running**：容器正在运行，也就是容器中的应用正在运行。
- **Paused**：容器已暂停，表示容器中的所有程序都处于暂停 ( 不是停止 ) 状态。
- **Stopped**：容器处于停止状态，占用的资源和沙盒环境都依然存在，只是容器中的应用程序均已停止。
- **Deleted**：容器已删除，相关占用的资源及存储在 Docker 中的管理信息也都已释放和移除。

创建

```shell
# 创建容器
docker create nginx:1.12

# 创建容器并指定名称
docker create --name nginx nginx:1.12

# 启动容器
docker start nginx

# 合并创建和启动容器的命令, -d 表示后台运行
docker run --name nginx -d nginx:1.12
```

管理

```shell
# 列出运行中的容器
docker ps 

# 列出所有容器
docker ps -a
```

停止或删除

```shell
# 停止容器
docker stop nginx

# 删除容器
docker rm nginx
```

进入容器

```shell
# 容器运行指定命令
docker exec nginx more /etc/hostname
docker exec -it nginx bash
```

链接容器

```shell
# 连接输入输出流到容器的主程序上
docker attach nginx
```

## 网络

容量网络模型三大核心概念：**沙盒 ( Sandbox )**、**网络 ( Network )**、**端点 ( Endpoint )**。

![](https://i.loli.net/2020/05/08/CAbvNu2j5zwri4O.png)

- **沙盒**，提供了容器的虚拟网络栈，也就是之前所提到的端口套接字、IP 路由表、防火墙等的内容。其实现隔离了容器网络与宿主机网络，形成了完全独立的容器网络环境。
- **网络**，即 Docker 内部的虚拟子网，使得容器之间可以相互通讯，并且与宿主机的网络相互隔离。
- **端点**，是容器上透出的某个接口，用于提供网络互联的能力。

### 容器互联

容器互联

```shell
# 启动一个 mysql 容器
docker run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql

# 通过 --link 指定要连接的容器
docker run -d --name webapp --link mysql webapp:latest

# web 应用可以这样连接到 Mysql 数据库
# docker 会提供这样的映射
"jdbc:mysql://mysql:3306/webapp"
```

暴露端口

```shell
# 可以通过 ps 命令看到 mysql 容器对其他容器暴露的端口
# --expose 可以指定暴露的端口
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
95507bc88082        mysql:5.7           "docker-entrypoint.s…"   17 seconds ago      Up 16 seconds       3306/tcp, 33060/tcp   mysql
```

### 管理网络

```shell
# 创建指定驱动的网络
docker network create -d bridge individual

# 查看创建的网络
docker network create -d bridge individual

# 通过 --netword 可以加入指定网络
docker run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=yes --network individual mysql:5.7
```

### 端口映射

