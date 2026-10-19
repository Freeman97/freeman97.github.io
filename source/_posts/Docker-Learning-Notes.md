---
title: Docker学习笔记
date: 2020-12-24 15:59:41
categories: 工具
tags:
- Docker
---

<!--more-->

# 核心概念
## Why Docker?

## Docker镜像(Images)
### 基本概念
一个镜像是一个只读的模板，它包含了创建一个Docker容器的命令。要创建一个新的Docker镜像，编写Dockerfile然后build即可。
当运行一个Docker容器时，容器使用一个隔离的文件系统。这一文件系统由容器的镜像来提供。因为镜像包括了容器的文件系统，所以镜像必须包括运行一个应用所需要的所有东西，包括依赖、配置项、脚本、二进制文件等。镜像也包括容器的其它配置，比如环境变量，一个用于运行的默认命令，以及其它元信息。

### 层(layer)的概念
在Dockerfile的每条命令都会为镜像创建一个层(Layer)。当修改Dockerfile并rebuild一个镜像时，只有被修改的层需要rebuild。


## Docker容器(Container)
### 基本概念
一个容器是一个镜像的可运行实例，是一个被松散地隔离开的进程。可以通过Docker CLI或Docker API来进行创建(create)、运行(start)、停止(stop)、移动(move)或删除(delete)。默认情况下，一个容器和其它容器以及宿主机是隔离的。一个容器由用于创建它的镜像以及创建时或运行时提供的配置选项来定义。

# 常用命令及工作流程
## Docker镜像加速
对于Windows 10，在Docker Desktop里修改`registry-mirrors`即可。阿里云会提供Docker镜像加速服务（容器镜像服务）。


## 镜像管理
### Build an Image
在Dockerfile的同一目录下运行`docker build`命令
```shell
docker build [-t IMAGE_NAME] [DOCKERFILE_DIRECTORY]
```

### 查看所有的Image
```shell
docker image ls
```

## 容器管理
### Start a Container
```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARGS]

# -p: 端口映射
# -d: detached mode
docker run -dp 3000:3000 getting-started
```

### 查看Container运行情况
```shell
docker ps
```

### Stop a Container
```shell
docker stop CONTAINER_ID
```

### Remove a Container
```shell
docker rm CONTAINER_ID
# Remove a running container
docker rm -f CONTAINER_ID
```

## Docker容器卷(Container Volumes)
每个容器都持有自己的暂存空间来创建、存储、修改和删除文件，任何容器内的改变都不会影响其它容器。一般情况下容器被删除时容器内所有文件都不复存在。可以使用容器卷(Container Volumes)来让容器的特定系统路径连接回宿主机。如果一个容器内的目录被挂载，在这一目录中的修改可以在宿主机上观察到。同时如果我们在容器重启时挂载相同的目录，就能在容器中看到相同的文件。

### 具名容器卷(Named Volume)
可以认为这是一个数据桶，Docker会负责维护硬盘上的物理位置，用户只需要知道这个数据桶的名称，只要提供这个名称就能得到对应位置的正确数据。
#### 创建卷
```shell
docker volume create VOLUME_NAME
# docker volume create todo-db
```
#### 为容器挂载卷
```shell
docker run -v VOLUME_NAME:DIRECTORY IMAGE
# VOLUME_NAME: 卷名
# DIRECOTRY: 将卷挂在到容器内的指定目录
# docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```
#### 查看所有卷
```shell
docker volume ls
```
#### 查看指定卷的详细信息
```shell
docker volume inspect VOLUME_NAME
# VOLUME_NAME: 卷名，可以通过docker volume ls找到
```
其中`Mountpoint`是数据在硬盘上存储的实际位置。
```
[
    {
        "CreatedAt": "2020-12-25T15:31:19+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```

### 绑定挂载(Bind Mount)
使用绑定挂载可以控制宿主机上的具体挂载点，bind mount通常用于为容器内的应用提供额外数据。bind mount不支持容器内容写入宿主目录，也不支持volume driver。
应用场景:在本地设置开发环境。开发机器不需要安装所有的构建工具和环境，而是可以通过Docker构建一个可以分发的开发环境并分发给开发人员(包括数据库开发环境也可以进行分发，但是数据库开发环境应该使用named volume)。

## Docker网络配置
### 容器之间的通信
只要容器处于同一个Docker网络内，就可以进行通信。

### 创建网络
``` shell
docker network create [ARGS] NETWORK_NAME
# 可以在ARGS中提供各种参数
# --driver=bridge 指定网络类型
# --subnet=192.168.6.0/24 指定子网划分
# --ip-range=192.168.6.0/24 指定IP范围
# --gateway=192.168.6.1 指定网关, 可以不提供
```

### 查看当前Docker网络
* 如果要查看当前所有的网络
```shell
docker network ls
# 大概会出现四栏的表格：
# NETWORK ID    NAME    DRIVER    SCOPE
```
* 如果要查看某个网络的详细情况（子网、默认网关等），可以使用`inspect`命令。可以看到连接到某一网络的所有容器信息，包括它们各自的IP地址。
```shell
docker network inspect [NETWORK ID | NAME]
```
* 节选一段返回信息。**如果不想让Docker容器直接通过iptables进行端口映射，或者iptables的端口映射出现问题，可以通过使用nginx反向代理等手段，让请求直接转发到 容器IP:端口 上**
```
[
    {
        "Name": "bridge",
        "Id": "ca18bdeb0aea357f983a90012ce2245670597495d3bc824f4d86f2cf81f75800",
        "Created": "2020-12-24T21:53:47.388294816+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "192.168.5.0/24",
                    "Gateway": "192.168.5.1",
                    "AuxiliaryAddresses": {
                        "DefaultGatewayIPv4": "192.168.5.2"
                    }
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "a41a18d43c71e52843f07683d1f672e241afaa922510c46934da2bcd71caaf1e": {
                "Name": "vigilant_jang",
                "EndpointID": "fe6dcd7083dd9bc8484e77b96ecf7bb82b241d4d0b2069e7012a315fc09b48ef",
                "MacAddress": "02:42:c0:a8:05:03",
                "IPv4Address": "192.168.5.3/24",
                "IPv6Address": ""
            },
            "b484dc156df6f30de8f4c13fff4a8cfbe31177a475d66f69c6efb16dfa2af8f5": {
                "Name": "hardcore_goldberg",
                "EndpointID": "50a6883e77447d0e21765f42daacbcaa0f435eea558e5d670ff785bec75e97da",
                "MacAddress": "02:42:c0:a8:05:04",
                "IPv4Address": "192.168.5.4/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```
### 在启动时让容器连接到指定网络
在执行`docer run`时提供`--network`参数即可。还可以提供`--network-alias`来为该容器在该网络上指定host名称，其他容器可以使用Docker提供的DNS服务，使用这一主机名来与该容器通信。
```shell
docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:5.7
```
