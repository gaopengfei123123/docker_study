
> 本章讲的大概能让你明白虚拟机和 docker 的区别...

docker 设置了两种构建镜像的方式:
### 通过 docker commit 构建镜像(不推荐)
这个命令是将先有的容器制作成镜像, 不过建议仅用在排查问题的时候使用, 平时生成容器时最好不要用这种镜像, 因为不知道里面有什么改动, 对于开发者来说完全是一个黑盒

命令格式:
```bash
docker commit [参数] <容器 ID 或 容器名> [仓库名[:标签]] [flags]
```

比如(我随便找了一个本地容器 ID):
```bash
➜  ~ docker commit -m "这是一个测试镜像" -a "GPF" 5ad06ec670eb local_nginx:v1
sha256:134e09cdce58842dea03202aa5b6516ead8268afe78d2203be8595b4f0bc5ebe
➜  ~ docker images
REPOSITORY              TAG                    IMAGE ID            CREATED             SIZE
local_nginx             v1                     134e09cdce58        2 seconds ago       109MB
```
这就把当前的容器转换成了镜像, 提交到远程就是:
```bash
docker push local_nginx:v1
```
和`git`很像对不对?   不过推送到远程时需要有个 [dockerhub](https://hub.docker.com/_/php/) 的账号

`docker history` 命令可以查看镜像的构建记录
```bash
➜  ~ docker history 134e09cdce58
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
134e09cdce58        12 seconds ago      nginx -g daemon off;                            0B                  这是一个测试镜像
649dcb69b782        4 days ago          /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>           4 days ago          /bin/sh -c #(nop)  STOPSIGNAL [SIGTERM]         0B
<missing>           4 days ago          /bin/sh -c #(nop)  EXPOSE 80/tcp                0B
<missing>           4 days ago          /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B
<missing>           4 days ago          /bin/sh -c set -x  && apt-get update  && apt…   53.7MB
<missing>           4 days ago          /bin/sh -c #(nop)  ENV NJS_VERSION=1.15.1.0.…   0B
<missing>           4 days ago          /bin/sh -c #(nop)  ENV NGINX_VERSION=1.15.1-…   0B
<missing>           11 days ago         /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
<missing>           11 days ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>           11 days ago         /bin/sh -c #(nop) ADD file:28fbc9fd012eef727…   55.3MB
```

### 通过 Dockerfile 构建镜像
一个 Dockerfile 就是一个构建镜像的脚本, 常用的几个命令也不多, 也就`FROM`, `COPY`, `RUN`, `ADD`, `ARG`, `ENV`, `VOLUME`, `EXPOSE`, `CMD`, `LABEL`, 其他的一些命令就不再这里说了, 想了解完整的 Dockerfile 的关键词看这个 [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#usage)

#### 一个简单的 demo
接下来我们就用一个完整的 Dockerfile 来示范一下:

首先创建一个Dockerfile
```bash
mkdir dockerfile_demo && cd dockerfile_demo
vi Dockerfile
```

```Dockerfile
# dockerfile 内容

# 继承的 ubuntu 镜像的版本号
FROM ubuntu:16.04
# 将本地 workdir 的文件复制到镜像内部, 另外有个 ADD 指令和这个效果类似, 不过不推荐使用
COPY ./sources.list /etc/apt/sources.list
# 在镜像内部执行命令, 这里就是更新版本和安装 nginx
RUN apt-get update && apt-get install -y nginx
# 修改 nginx 默认的 index.html 的内容
RUN echo 'Hi, I am in your container'\
    >/var/www/html/index.html
# 镜像对外暴露80端口
EXPOSE 80
```
然后为了更新速度快一点, 就要换一个国内镜像
```bash
vi sources.list
```

```bash
# sources.list 内容

deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
```
然后还是在这个目录下, 运行:
```bash
docker build ./ -t local_ubuntu_nginx:v2
```
执行完毕后就能看到本地多了一个镜像
```bash
➜  Documents docker images
REPOSITORY              TAG                    IMAGE ID            CREATED             SIZE
local_ubuntu_nginx      v2                     b0f5984c042a        14 minutes ago      214MB
```
用这个镜像构建一个容器:
```bash
➜  Documents docker run -it --rm -p 8088:80 local_ubuntu_nginx:v2 /bin/bash
root@5cb9ff723bee:/# /usr/sbin/nginx
root@5cb9ff723bee:/#
```
保持终端打开, 本地访问 `http://localhost:8088/` 就能看到欢迎界面了 
 
如果这个是关闭交互端口, 这个就访问不到了, 这里要说一个**关键点**了
> 在 docker 中运行的程序不能使用后台运行的模式, 否则 docker 会任务这个容器不活跃或出现问题而自动关闭
但是 docker 容器本身是可以后台运行的: docker run -d ......



#### 关于 docker build 多说一句
之前我们构建时执行的:
```bash
docker build ./ -t local_ubuntu_nginx:v2
```
这个 `./` 的路径指的是构建文本流(context)的路径, 而不是 `Dockerfile`的文件路径, 在 `Dockerfile` 中用的各种相对路径都是基于 `context` 的,我们完全可以是
```bash
docker build /path/to/context -f /path/to/anywhere/Dockerfile_demo -t local_ubuntu_nginx
```
你看, Dockerfile 的文件名不就变成 `Dockerfile_demo` 了, 如果有
```Dockerfile
COPY ./sources.list /etc/apt/sources.list
```
这样的操作, 那么它的完整路径应该是 `/path/to/context/sources.list`, 不过默认的情况下就是这两个路径是一起的, 不设置镜像`tag`的话就拿 Dockerfile 所在的目录名为镜像名,默认`latest`版本


### 多阶段构建
有时候我们的运行环境和编译环境是两回事, 就拿 golang 来举例, 我们只会去维护代码, 而不去管它生成的二进制包是什么, 又因为golang 打包出来的二进制文件几乎是没有依赖, 只要执行这个文件就行, 那么在运行环境中和编译相关的程序就是多余的, 可以看一下官网的示例 [Use multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/), 不能翻墙的看这个

```Dockerfile
FROM golang:1.7.3
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
# 这个是关键, --from=0 是指从第一个镜像(from)中复制内容, 程序中的1 就是0
COPY --from=0 /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]
```
```bash
$ docker build -t alexellis2/href-counter:latest .
```

这是我自己构建镜像时编译 `swoole.so` 的镜像和运行时的镜像大小
![docker-images](http://blog-image.onlyoneip.com/docker-images.png)
我们只想要一个 `swoole.so` 像 `gcc`, `make` 都是不是代码运行时需要的模块, 因此只是在编译的时候用上, 吐一个`swoole.so`出来就行, [相关代码](https://github.com/gayhuber/dnmp/tree/master/php-cli)

```Dockerfile
FROM php:5.6.36-cli-alpine3.7 as builder

# 中国特色
RUN echo "http://mirrors.ustc.edu.cn/alpine/v3.7/main/" > /etc/apk/repositories

# 添加编译 swoole 需要的前置插件
RUN apk update && \
    apk upgrade && \
    apk add alpine-sdk linux-headers && \
    apk add autoconf gcc make

RUN wget https://github.com/swoole/swoole-src/archive/1.8.12-stable.tar.gz && \
    tar zxvf 1.8.12-stable.tar.gz && \
    cd swoole-src-1.8.12-stable && \
    phpize && \
    ./configure && \
    make && make install




FROM php:5.6.36-cli-alpine3.7 as runtimer

# 这里因为给每一个阶段加了别名, 这样更方便一点
COPY --from=builder /usr/local/lib/php/extensions/no-debug-non-zts-20131226/swoole.so /usr/local/lib/php/extensions/no-debug-non-zts-20131226/swoole.so

COPY ./swoole.ini /usr/local/etc/php/conf.d/swoole.ini

# 通常 run 是构建镜像时用的, 会保存一层缓存, cmd 就是镜像启动后执行的命令
RUN ["php", "-m"]
CMD ["php", "-a"]
```



### 构建镜像时需要注意的几点
1. 尽量使用官方镜像, 同时版本选择 `alpine > debian > ubuntu > centos`, `alpine`版本的镜像是最小的
2. COPY 和 ADD 尽量使用 COPY, COPY 只是单纯的复制, ADD 则会自动执行一些东西, 有可能出现意料外的问题
3. CMD 和 ENTERPOINT 这两个可以查一下区别, 我个人习惯用 CMD
4. 容器中的程序都要前台执行的模式, 使用 daemon 模式会被退出(你想想本身一个后台运行的容器中有个后台运行的程序...)
5. 每一个 `RUN` 指令就是一层缓存, 当其中一个步骤发生更改那么这之后的步骤也都重新构建
6. 系统更新最好合成一个 RUN 比如:
    ```Dockerfile
    RUN apk update && \
        apk upgrade && \
        apk add alpine-sdk linux-headers && \
        apk add autoconf gcc make
    ```
    同样还是为了避免因为缓存时出现问题
7. 多看看官方构建的镜像, 收获会很多, 比如 [docker-nginx](https://github.com/nginxinc/docker-nginx/tree/e3e35236b2c77e02266955c875b74bdbceb79c44/stable), [docker-php](https://github.com/docker-library/php/tree/4af0a8734a48ab84ee96de513aabc45418b63dc5)



