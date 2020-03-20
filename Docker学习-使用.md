---
title: Docker学习&使用
copyright: true
date: 2020-02-03 13:10:52
tags:
	- docker
	- go
categories:
	- docker
description:
---

# Docker的学习&使用

>Docker：开源应用容器引擎，基于Go， 17.03后分为社区版和企业版

<!--more-->

## Debian安装Docker

```bash
# Debian stretch ======基本通用.....（*＾-＾*）=============

# 可能需要梯子加快速度。。。ㄟ( ▔, ▔ )ㄏ proxychains

# 添加key
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

# 添加docker的库
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"

# install
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# 测试一下
sudo docker run hello-world
# 有以下信息则成功了。。。
# Unable to find image 'hello-world:latest' locally
# latest: Pulling from library/hello-world
# ............
# Status: Downloaded newer image for hello-world:latest
# Hello from Docker!
# .............
```

## 镜像加速。。。

>没梯子用这个。。。(๑•̀ㅂ•́)و✧...===================

>debian 8+使用
>
>vim /etc/docker/daemon.json 
```json
{"registry-mirrors":["https://registry.docker-cn.com"]}
```

>重启docker服务

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
# 检查一下
sudo docker info | grep https://
# 结果
# WARNING: No swap limit support
#  Registry: https://index.docker.io/v1/
#   https://registry.docker-cn.com/

# 如果没有这个结果就是失败了，，，，，
```

## 容器的使用

### 拉取镜像

```bash
# docker pull 镜像名:tag
docker pull ubuntu
# tag默认 latest
```

### 启动镜像

```bash
docker run -it ubuntu /bin/bash
# -i 交互式操作
# -t 终端
# /bin/bash 启动镜像运行的命令， 这里使用bash shell
```

### 后台运行

```bash
docker run -itd --name ubuntu1 ubuntu /bin/bash
# -d 指定容器运行模式
# --name 指定容器名字
# 这个命令会打印一个容器id
```

### 启动容器

```bash
# docker start <容器ID>
docker start 5f52c8ce6964
```

### 重启容器

```bash
# docker restart <容器ID>
docker restart 5f52c8ce6964
```



### 停止容器

```bash
# docker stop 容器id
docker stop <容器ID>
```

### 进入在后台的容器

* docker attach    ====> 使用这个命令，在退出容器后，容器会停止
* docker exec       ====> 使用这个命令，退出容器后， 容器不会停止

```bash
# docker attach <容器ID>
docker attach 5f52c8ce6964
# 5f52c8ce6964是一个我本地ubuntu的镜像

# docker exec -it <容器ID> /bin/bash
docker exec -it 5f52c8ce6964 /bin/bash
```

### 导出容器

```bash
# docker export <容器ID> > <name>.tar
docker export 5f52c8ce6964 > ubuntu.tar
```

### 导入容器

```bash
# docker import <name>.tar <name>:<tag>
docker import ubuntu.tar ubuntu1:v1
```

### 删除容器

```bash
# docker rm -f <容器ID>
docker rm -f 5f52c8ce6964
```

..........

```bash
docker --help
# docker <command> --help
docker images --help
```

## Docker容器连接

### 网络端口映射

```bash
# 启动一个webapp
docker run -d -P training/webapp python app.py
# -P 容器内部端口随机映射到主机高端口
# -p 容器内部端口映射到指定的主机端口
docker run -d -p 5000:5000 training/webapp python app.py

# 指定容器绑定的网络地址
docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py

# 如果要绑定udp端口
docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py

# 查看
docker port
```

### Docker互联

### 新建网络

```bash
docker network create -d bridge test-net
# -d 指定Docker网络类型， 有bridge， overlay
```

### 连接容器

 运行一个容器并连接到新建的网络

```bash
docker run -itd --name test1 --network test-net ubuntu /bin/bash
```

再来一个

```bash
docker run -itd --name test2 --network test-net ubuntu /bin/bash
```

测试一下是否连接

```bash
docker exec -it test1 /bin/bash # 进入容器test1
ping test2 # 可能没有ping ， 安装一下就好 apt-get update & apt-get install iputils-ping
# 有以下结果则是成功
# 64 bytes from test2.test-net (172.18.0.3): icmp_seq=1 ttl=64 time=0.196 ms
# 64 bytes from test2.test-net (172.18.0.3): icmp_seq=2 ttl=64 time=0.091 ms
```

## [DockerFile](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/)

先来个栗子。。。。

```bash
mkdir dockerfile & cd dockerfile
vim DockerFile
# 添加以下内容
```
```dockerfile
FROM nginx
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html
```
```bash
# 构建镜像
docker build -t nginx:test .

# 成功结果
# 。。。。。。
# Successfully built 20688a4eb21c
# Successfully tagged nginx:v1

docker images #查看一下是否已有镜像
```

### FROM  指定基础镜像

```dockerfile
FROM mysql
```

>特殊镜像：scratch
>
>scratch是个空白镜像， 没有任何东西，如果可执行文件不需要以系统为基础，那么使用这个镜像可以使得最后的包体积更小

### RUN 执行命令

>格式：
>
>* RUN <命令>    ======= 就像shell中直接输入命令
>* RUN ["可执行文件"， "参数1"， "参数2"] =========更像调用函数

```dockerfile
FROM debian:scratch

RUN apt-get update
RUN apt-get install gcc
```

>dockerfile每一个指令执行后都会新建立一层，这样会产生多层镜像，造成镜像臃肿多层
>
>所以许多命令可以一起执行

```dockerfile
FROM debian:scratch

RUN apt-get update && apt-get install gcc -y
```



### COPY 复制文件

>格式
>
>* COPY [--chown=<user>:<group>] <源路径>... <目标路径>
>* COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]

```dockerfile
COPY package.json /usr/src/app
```

>源路径可以有多个，可以是通配符

```bash
COPY *.conf /usr/src/app/conf/
```

><目标路径> 可以是绝对路径，也可以是相对路径
>
>PS：相对路径查看 WROKDIR

>--chown=<user>:<group> 用来改变文件属性和用户组

```dockerfile
COPY --chown=777:mygroup *.js /usr/src/app/
```

### ADD 更高级的复制文件

>ADD和COPY的格式一致，功能更加高级
>
>* <源路径>可以是 <code>URL</code> 
>* <源路径>为tar压缩文件时，ADD会自动解压到<目标路径>

```dockerfile
FROM nginx
ADD test.tar.gz /
```

>有些时候，我们希望只是复制压缩文件，而不要解压，这是就不能使用ADD， 而应使用COPY， 所以适合使用ADD的场景就是需要自动解压的时候
>
>PS：使用ADD指令会使镜像构建缓存失效，令构建变得缓慢

### CMD容器启动命令

>格式和RUN一样
>
>CMD指定的命令会在容器启动时执行

```dockerfile
CMD echo $HOME

# 实际执行的时候会被替换为
CMD ["sh", "-c", "echo $HOME"]
```

>也就是说CMD指定的命令 第一种格式最终会被替换为第二种格式 执行

```dockerfile
CMD service nginx start
# 会被替换为
CMD ["sh", "-c", "service nginx start"]
# 此时主进程为 sh ， sh执行结束后容器就会退出， 而 nginx 也不会作为守护进程启动
```

### ENTRYPOINT 入口点

>格式和RUN一样

>ENTRYPOINT 和 CMD 一样，都是指定在容器启动时的执行命令和参数
>
>但是有了 ENTRYPOINT 后， 会将CMD的内容作为参数传递给 ENTRYPOINT

```dockerfile
# 什么意思呢
FROM ubuntu:18.04

ENTRYPOINT ["curl", "-s", "https://ip.cn"]
```

```bash
# 构建镜像
docker build -t checkip

# 启动镜像
dcoker run checkip -i
# 此时， 这个参数 -i 不会是需要执行的命令，而是作为参数传递给 ENTRYPOINT 指定的命令
# 也就是说 ENTRYPOINT 最终会执行 curl -s https://ip.cn -i
# 如果不是使用 ENTRYPOINT 而是 CMD 那么将会执行 -i ， 但是 -i 并不是一个可执行的命令
```

>而根据这个特点，可以使用 参数 指定 容器 运行的脚本

```dockerfile
# 这是redis官方的镜像
FROM alpine:3.4
...
RUN addgroup -S redis && adduser -S -G redis redis
...
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]
```

>此时我们就可以指定运行的脚本

```bash
# 栗子。。。。。
docker run -it redis id
```

### ENV 设置环境变量

>这个比较简单， 格式
>
>* ENV \<key\> \<value\>
>* ENV \<key1\>=\<value1\> \<key2\>=\<value2\> ..........
>
>使用环境变量 $<Key>

```dockerfile
ENV VERSION=1.0
run apt-get install app-v$VERSION
```

### ARG 构建参数

>格式
>
>ARG <参数名>[=<默认值>]
>
>ARG和ENV一样，都是设置环境变量， 但是ARG设置的环境变量只在 docker build的时候有用， 也就是说，容器运行时，就没有这些环境变量了

```bash
# 在shell指定 ARG的值
docker build --build-tag <参数名>=<值> # 覆盖默认值
# 如果Dockerfile中没有使用 ARG 指定这个参数，在1.13前会报错，之后的版本是警告
```

### VOLUME 定义匿名卷

>格式
>
>* VOLUME ["<路径1>", "<路径2>".........]
>* VOLUME <路径>

>挂载一个匿名卷，任何向这个卷的写入信息都不会记录进存储层，保证容器存储层的无状态化

```dockerfile
VOLUME /data
```

### WORKDIR 指定工作目录

>格式
>
>WORKDIR <路径>

>指定dockerfile执行命令时所在的路径

```dockerfile
RUN cd /app
RUN echo "hello" > world.txt # 执行这条命令时路径并不是 /app， 因为在Dockerfile中两个RUN的执行环境并不一样
```

>所以，如果希望执行的命令所在的路径都在一个， 可以使用WORKDIR指定

```dockerfile
WORKDIR /app
```

### USER 指定用户

>格式： USER <用户名>[:<用户组>]

>使用USER切换的用户必须是已经创建的

```dockerfile
RUN groupadd -r redis && useradd -r -g redis redis #这样就可以创建一个用户
USER redis # 切换用户，以后所有命令的身份都是 redis 
```

### EXPOSE 暴露端口

>格式： EXPOSE <端口1> [<端口2>.....]

>EXPOSE只是声明一个容器运行时提供的端口，并不是自动开启
>
>使用 -P 时会自动映射到EXPOSE的端口

## Dockerfile多段构建



TODO...........