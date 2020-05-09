# Docker 实现简析

## 生态

* Docker Engine，容器化工具包。
  * Docker daemon，即 Docker 服务，包括了容器管理、应用编排、镜像分发等。并向外暴露 API。
  * Docker CLI

## 实现



Docker 的实现，主要归结于三大技术：

* 命名空间 ( Namespaces ) 
* 控制组 ( Control Groups ) 
* 联合文件系统 ( Union File System ) 

## Namespace

运行时隔离



## Control Groups



硬件资源隔离



## Union File System

增量式文件结构

## 生命周期

![](https://i.loli.net/2020/05/08/Ml1kVdprTcONL6g.png)