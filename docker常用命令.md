### docker run [image name] [cmd] 创建并启动
比如 `docker run -i -t ubuntu:16.04 /bin/bash`, 
1. 参数`-i`让容器前台执行 `-t` 给容器运行时分配一个伪终端, 这两个参数配合使用就是相当于打开了一个新终端
2. 如果需要后台运行,那么就需要参数 `-d`, 感受一下这两个的区别:
    1. `docker run ubuntu:16.04 /bin/sh -c "whild true; do echo helllo world;sleep 1; done"`
    2. `docker run -d ubuntu:16.04 /bin/sh -c "whild true; do echo helllo world;sleep 1; done"`
3. 参数`-P` 随机绑定一个宿主机的端口到容器暴露的端口上
4. 参数`-p 宿主机端口:容器端口` 绑定指定端口, 也可以是 比如 `docker run -d -p 127.0.0.1:5000:5000 image/name CMD` 这样的,指定 ip, 且可以绑定多个端口  `-p 80:80 -p 443:443`

1会将输出打印到宿主机上,2不会,只会返回一个容器 id,但是它仍然在运行, 使用 `docker container logs [container ID or NAMES]` 查看容器执行日志

### docker container stop [container-id or name] 停止容器
一般通过 `-i -t` 执行的容器退出后就会停止,有的容器执行完对应的`CMD`之后就停止, 这条命令主要是用于那些会常驻后台的容器

### docker attach 和 docker docker exec 进入后台容器
`attach` 进入再`exit`会导致容器停止, 推荐使用 `exec` 命令 eg: `docker exec -it [container-id]`, 只用 `-i` 会看不到终端,但是结果还是能输出出来


### docker container rm [container-id or name]  或者 docker rm [container-id or name]  删除容器

## docker container prune 清除所有停止的 container
这应该是个新接口吧,早先的时候都是 `docker rm (docker ps -a -q)`

### docker rmi [image-id or name] 删除 image
这个 image 和 container 是有点区别的,简单的说`image`是`container`的爸爸,要删除`image`需要先移除和它有关的`container`才行


### docker export 和 docker import  导入导出
导出:`docker export 038f424d2 > ubuntu.tar`
导入:`cat ubuntu.tar | docker import - test/ubuntu:v1.0`, 也可以导入指定的 URL, `docker import http://xxxx.xxx/example.tgz example/image`

### docker port [contain-id or name] 查看绑定的端口和地址