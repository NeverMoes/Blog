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

```shell
# 使用 -p 端口映射到宿主机上面
docker run -d --name nginx -p 80:80 -p 443:443 nginx:1.12

# ps 中 会使用 -> 来表示端口映射关系
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                      NAMES
bc79fc5d42a6        nginx:1.12          "nginx -g 'daemon of…"   4 seconds ago       Up 2 seconds        0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   nginx
```

## 数据

### 挂载方式

Docker 的数据挂载在实现上有三种方式：

- **Bind Mount**：**容器目录 <-> 宿主机目录**。指定容器和宿主机目录，在这两个目录之间形成一个挂载的映射。使得容器内外对文件的读写，都是相互可见的。

- **Volume** ：**容器目录 -> Docker 管理目录**。只需要指定容器内的目录，宿主机中的目录由 Docker 管理。这样可以无需关注容器目录到底挂载到了哪里。

- **Tmpfs Mount**：**容器目录 -> 宿主机内存**。将容器目录挂载到宿主机内存中，注意这是非持久性的。

![](https://i.loli.net/2020/05/11/mY3LWGl7F4dNEBQ.png)

### 挂载文件

```shell
# 使用 使用 -v 或 --volume 来挂载宿主目录
# 格式为 <host-path>:<container-path>
$ docker run -d --name nginx -v /webapp/html:/usr/share/nginx/html nginx:1.12
```

可以使用 `inspect` 看到挂载状态

```shell
$ docker inspect nginx
[
    {
## ......
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/webapp/html",
                "Destination": "/usr/share/nginx/html",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
## ......
    }
]
```

也可以使用内存来临时挂载一个目录，注意这不是持久性的。

```shell
$ docker run -d --name webapp --tmpfs /webapp/cache webapp:latest
```

```shell
$ docker inspect webapp
[
    {
## ......
         "Tmpfs": {
            "/webapp/cache": ""
        },
## ......
    }
]
```

### 使用数据卷

```shell
# -v <name>:<container-path> 的方式命名数据卷
$ docker run -d --name webapp -v appdata:/webapp/storage webapp:latest
```

### 数据卷操作

```shell
# 创建数据卷
$ docker volume create appdata

# 查看数据卷
$ docker volume ls

# 删除数据卷
$ docker volume rm appdata

# 指定删除与容器关联的数据卷
$ docker rm -v webapp

# 指定删除没有被容器引用的数据卷
$ docker volume prune -f
```

### 数据卷容器

数据卷容器，就是一个没有具体指定的应用，甚至不需要运行的容器，主要是为了定义一个或多个数据卷并持有它们的引用。

```shell
# 创建数据卷容器
$ docker create --name appdata -v /webapp/storage ubuntu

# 引用数据卷容器
# --volumes-from 指定数据卷容器
$ docker run -d --name webapp --volumes-from appdata webapp:latest
```

### 备份与迁移

```shell
$ docker run --rm --volumes-from appdata -v /backup:/backup ubuntu tar cvf /backup/backup.tar /webapp/storage
```

使用 `tar` 命令打包到临时的数据卷。

* `--rm` 容器停止后自动删除。



```shell
$ docker run --rm --volumes-from appdata -v /backup:/backup ubuntu tar xvf /backup/backup.tar -C /webapp/storage --strip
```

恢复内容

### 其他

可以使用 `--mount` 更方便地传递参数。

```shell
$ docker run -d --name webapp webapp:latest --mount 'type=volume,src=appdata,dst=/webapp/storage,volume-driver=local,volume-opt=type=nfs,volume-opt=device=<nfs-server>:<nfs-path>' webapp:latest
```

## 镜像管理



## Dockerfile



## Maven 构建

```xml
<plugins>
    <!--这是原有的spring boot插件-->
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
    <!--新增的docker maven插件-->
    <plugin>
        <groupId>com.spotify</groupId>
        <artifactId>docker-maven-plugin</artifactId>
        <version>0.4.12</version>
        <!--docker镜像相关的配置信息-->
        <configuration>
            <!--镜像名，这里用工程名-->
            <imageName>${project.artifactId}</imageName>
            <!--TAG,这里用工程版本号-->
            <imageTags>
                <imageTag>${project.version}</imageTag>
            </imageTags>
            <!--镜像的FROM，使用java官方镜像-->
            <baseImage>java:8u111-jdk</baseImage>
            <!--该镜像的容器启动后，直接运行spring boot工程-->
            <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
            <!--构建镜像的配置信息-->
            <resources>
                <resource>
                    <targetPath>/</targetPath>
                    <directory>${project.build.directory}</directory>
                    <include>${project.build.finalName}.jar</include>
                </resource>
            </resources>
        </configuration>
    </plugin>
</plugins>
```



[使用Maven插件为SpringBoot应用构建Docker镜像](http://www.macrozheng.com/#/reference/docker_maven?id=使用maven插件为springboot应用构建docker镜像)

## 参考

[[开发者必备的 Docker 实践指南](https://juejin.im/book/5b7ba116e51d4556f30b476c)](https://juejin.im/books)