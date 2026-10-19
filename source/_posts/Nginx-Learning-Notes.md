---
title: Nginx-Learning-Notes
date: 2020-12-25 10:19:38
categories: 工具
tags: 
- 后端开发
- 服务器
---

<!--more-->

# Nginx学习笔记
## 是什么?
* 一个TCP/UDP的通用服务器和反向代理服务器，可以支持HTTP、HTTPS、SMTP等协议。
* 可以实现负载均衡和URL重写
* nginx有一个master进程和多个worker进程。master进程的主要目的是读取并应用配置文件，并维护worker进程。worker进程会负责实际的请求处理。

## 为什么要反向代理
1. 保护了真实的Web服务器，外网只能看到反向代理服务器，而反向代理服务器上没有真实数据。
2. 可以减少Web服务器压力。
3. 请求的统一控制，包括权限控制和过滤规则等。
4. 区分动态和静态可缓存的内容。
5. 负载均衡
6. 解决跨域问题。**反向代理不是单纯的请求转发**，而是会代理用户向代理下的节点服务器重新发起请求。
7. 作为真实服务器的缓冲。

## 配置文件的结构
nginx由多个模块组成，而这些模块由配置文件中的指令进行控制。指令被分为简单指令(单纯的一条指令，分号结尾)和块级指令(由花括号`{}`包裹)`。如果一个块级指令能够将其它指令包裹在它的花括号中，那么他被称为一个上下文(context)。

## 基本操作
### 安装
在CentOS 7上安装nginx
```shell
sudo yum install yum-utils
```
创建`/etc/yum.repos.d/nginx.repo`文件，文件内容为:
```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```
默认情况下，nginx packages使用`nginx-stable`repository。如果要切换到`mainline`，则运行以下命令:
```shell
sudo yum-config-manager --enable nginx-mainline
```
随后，安装nginx
```shell
sudo yum install -y nginx
```

### 启动
```shell
    nginx
```
### 使用`-s`参数控制nginx
```shell
ngingx -s SIGNAL
``` 
* `stop`: 快速关闭
* `quit`: graceful shutdown，会等待worker进程完成当前请求的处理。
* `reload`: 重新加载配置文件，不必重启就可以让配置生效
  * 当master接收到这一信号时，它会检查新配置文件的语法正确性，并应用这一配置。如果这一步成功，master会启动新的worker进程并向旧的worker进程发送消息让他们关闭。否则，master会回滚更改并继续使用旧的配置来工作。旧的进程在接受到shutdown信息后会停止接受新请求，将当前的请求处理完后才退出。
* `reopen`: 重新打开日志文件

## 提供静态内容
在配置文件中的`http`块级指令中编写`server`块级指令。在`server`中加入`location`块级指令。
```
http {
    server {
        # 定义URI匹配规则(最长匹配)
        location / {
            # 静态内容的根目录
            root /data/www;
        }
    }
}
```

## 配置代理服务器
nginx的常见应用之一就是作为代理服务器，接受请求并将其转发给被代理的服务器，从被代理的服务器获取响应，然后将响应发送给客户端。配置方法是在`location`块级指令中加入`proxy_pass`指令。`location`指定的URI匹配可以编写正则表达式来表示更灵活的匹配规则，正则表达式以`~`开头
```
server {
    location / {
        proxy_pass http://localhost:8080;
    }
    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

## sendfile设置
`sendfile on`。高效文件传输模式（？），将tcp_nopush和tcp_nodelay设置为on防止网络阻塞（？？）

## 配置负载均衡策略
* 用于从upstream模块定义的上游服务器列表中选取一台服务器接受用户的请求。
```
http {
    upstream myServer {
        server xxx.xxx.xxx.xxx:yyyy;
        server yyy.yyy.yyy.yyy:zzzz;
        server zzz.zzz.zzz.zzz:xxxx [upstream基本参数];
    }

    server {
        listen [端口];
        server_name [监听地址];
        location [正则] {
            root [根目录];
            index [默认页];
            proxy_pass myServer; # 将请求转发到myServer定义的服务器列表
            deny [拒绝的IP];
            allow [允许的IP];
        }
    }
}
```
### upstream基本参数
* 对每台服务器的一些配置
* fail_timeout, max_fails: 如果在fail_timeout时间内发往该服务器的请求失败了max_fail次，认为该服务器已经停机
* fail_time：服务器被认为停机的时间长度
* backup: 标记该服务器为备用服务器，出现停机时请求会发送到此处（hash、ip_hash和random负载均衡方法下该参数不可用）
* down: 标记服务器永久停机
* max_conns: 限制该服务器的最大同时活动连接数

### 轮询
上述的例子就是轮询负载均衡。轮询是upstream模块默认的负载均衡策略。每个请求按照时间顺序逐一分配到不同的服务器。
* Nginx默认就是使用一个加权的Round-Robin负载均衡方法。

### 权重
为每台服务器配置权重，在轮询时为权重高的后端服务器分配更多请求。
* a = 1, b = 2, c = 3
* 加权轮询的顺序可能是`{b, c, b, c, a, c}`。这个加权序列应该尽量让相同的节点尽量分散。

### ip_hash
每个请求按访问ip的hash结果分配，每个客户端固定访问一个后端服务器。可以使用该策略来实现粘性session。
如果对应的服务器不可用则会转发到其他服务器。

### hash
可以指定key进行hash，进而为请求分配指定的服务器。
```
upstream hashups {
    # ip_hash;
    hash [key] [consistent]; # 加入consistent参数使用一致性哈希
    server ...;
    server ...;
}
```

## Nginx限流
### 控制单个IP的请求处理频率
* `ngx_http_limit_req_module`
* 定义内存区用于存储IP地址访问信息，设置`rate=xxr/s`可以设置每秒只处理每个IP地址的xx个请求
* 控制突发流量: `burst=xx`
* 控制单个IP的请求处理频率。

### 控制并发连接数
* `ngx_http_limit_conn_module`

### 限流的原理
#### 漏桶算法
桶的大小固定，请求依序进入桶中，但是进入桶中的请求只会以恒定的速率被处理（漏水的速率恒定）。如果进入桶中的请求溢出桶外，则这部分请求不会被处理。
#### 令牌桶算法
桶的大小固定，按固定的速度往桶中丢令牌，桶满之后后续的令牌都无法添加。取令牌时，桶中有令牌，则处理取令牌操作并处理真正的请求。否则取令牌操作将被拒绝，请求也不会被处理。
请求在队列中进行排队取令牌，无法取得令牌时请求不会被处理，需要进行等待或直接拒绝请求。
令牌桶算法允许一定程度的突发请求。

# 原理
## Nginx为什么快？
* 多进程 + IO多路复用。单个进程处理请求遇到再次需要进行IO的情况时，会直接向多路复用器增加新的事件，然后继续处理别的请求。
* Nginx也使用反应器模式，主事件循环等待操作系统发出IO事件准备完成的信号，这样数据就可以从Socket读取到缓冲区。

## 反向代理流程
1. 用户通过域名发出访问Web服务器的请求，该域名被DNS服务器解析为反向代理服务器的IP地址。
2. 反向代理服务器接收用户的请求。
3. 反向代理服务器在本地缓存中查找请求的内容，找到后直接把内容发给用户。
4. 如果本地缓存中没有用户所请求的内容，反向代理服务器代替用户向源服务器请求同样的信息内容，并把信息内容发给用户。如果信息内容是缓存的，还会把它存到缓存中。

### Nginx请求处理
通过配置文件中配置的规则将本次请求映射到一个location block。Location中配置的各个指令启动不同的模块去完成工作。
#### 多进程模型
* 一个Master进程，多个Worker进程。
* Master进程根据配置建立需要listen的网络socket文件描述符，然后fork出多个worker进程。同时Master还负责在不停止的情况下加载配置文件。
* Worker进程会使用epoll监听同一个Socket fd。连接到来时对所有worker进程来说socket fd都可读。此时为保证只有一个进程处理该事件，worker进程需要抢占互斥锁`accept_mutex`，只有抢到互斥锁的进程才会调用accept接受链接。
