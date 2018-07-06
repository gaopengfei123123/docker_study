
>  从命名上就知道这是一篇简单粗暴的`docker`新手入门教程， 为什么要简单粗暴？ 我认为有自学能力的人帮他入门就够了， 不能自学的一时半会儿也教不会， 不符合入门教程的初衷， 建议出门左拐去找找视频教程...

### 本章目标
1. 大概了解 docker 是个什么玩意
2. 知道常用的 docker 指令参数, 能启动一个容器(不然还想怎么样? 上天吗?)

### 安装环境

强烈推荐使用 `Docker for Mac` 或 `Docker for Windows`, 这两个工具已经将 `Kitematic` 和 `docker-compose` 集成好了， 至于这两个工具是做什么的咱们后面再说， `win10` 版本需要专业版的， 不然开启不了`Hyper-V`， `win7` 就别想了，不支持...

怎么安装在 [阿里云镜像容器服务](https://cr.console.aliyun.com/#/accelerator) 里面都说的很清楚了， 连**国内镜像源**都给你安排好了， 咱们就进入下一话题
> PS: 如果是 CentOS 6 的就需要升级一下系统内核了， [centOS6.5 安装docker](http://note.youdao.com/noteshare?id=d4515598471a46ed2fd6090e548107b3&sub=WEBd79b399d399855870db6fcaf5822a497)， 毕竟都 `8102` 年了， docker 又是个比较新的东西， 对于稍微久一点的系统的支持就不那么友好

### 运行第一个容器
安装完环境之后就启动一个镜像开开眼儿
```bash
docker run -d -p 8080:80 --name local_nginx nginx
```
然后访问 `http://localhost:8080/` 就能看到 `nginx` 的初始界面了
![run-image](./images/run-nginx.png)

中间发生了什么呢？ 

1. `docker run` 运行镜像的起手式， 详情查看 `docker run --help`
2. `-d` 启动 `docker` 守护进程
3. `-p 8080:80` 将本地的 `8080` 端口绑定到容器的 `80` 端口上
4. `--name local_nginx` 分配一个容器名， 不写的话会默认分配要给， 不过这个还是很有用的
5. `nginx` 指定运行的镜像名，如果没有指定标签则默认是 `latest`， 这里其实是启动`nginx:latest`镜像

现在可以查看一下本机都在运行着什么镜像
```bash
PS D:\docker_study> docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
6732fa239270        nginx              "nginx -g 'daemon of…"   18 minutes ago      Up 18 minutes       0.0.0.0:8080->80/tcp   local_nginx
```
`docker ps` 只能看到正在运行中的容器， 想看到全部的就是 `docker ps -a`

**进入这个容器的命令:**
```bash
docker exec -it 6732fa239270 /bin/bash
#或
docker exec -it local_nginx /bin/bash
```
解释一下:
1. `docker exec`    在容器中执行命令
2. `-i`             保持stdin打开
3. `-t`             分配一个伪终端(tty)
4. `6732fa239270 或 local_nginx`    这里你也发现了， 可以是通过 `CONTAINER ID` 也可以是 `NAMES` 这里的 `CONTAINER ID`分为128位长ID和32位短ID， 不过作用都是一样的
5. `/bin/bash`      运行容器中的 `/bin/bash` 脚本

进入容器中感觉其实和进入一个虚拟机一样， 但是容器和虚拟机有点区别， 这个我们下一小节会讲到

**关闭容器**
```bash
docker stop 6732fa239270 或 local_nginx
```

### 什么是容器？什么是镜像？
![image-container](./images/image-container.jpg)

之前我们使用 `VirtualBox` 装虚拟机的时候有装盘镜像, 但是启动后就是一个个的虚拟机了, 不过在 docker 中和虚拟机还是有点区别

就拿上图来说, `container`就是镜像的实例化, `image` 是容器的底层支撑, 其实他们的关系用代码中的类`Class`来比喻是最合适的:

* Class 就是我们实际开发中写的一个代码集合, Object 是 Class 实例化之后生成的一种资源变量
* Image 也是预先写好的逻辑, 并存在一个地方, Container 是 Image 启动之后生成的一个虚拟系统
* 实例化出来的 Object 不会影响到 Class 中的内容
* 已经启动的 Container 也不会影响到 Image 中的逻辑
* Class 可以继承别的 Class, 从而继承它的特性
* Image 也是可以继承别的 Image, 并在它的基础上构建新的镜像
* 一个 Object 对应着一个 Class, 但是 一个 Class 可以实例化无数个 Object
* 同理, 一份 Image 可以生成无数个 Container, 这就是方便集群化部署的所在

简单的说 Container 就是 Image 的儿子, 模样和 Image 预想的一样, 但是 Container 运行之后会发生一些改变, 而且这种改变是可以保存的



### 常用的运行参数和命令
咱们先不说构建镜像的事儿(那是下一章的话题), 这里先了解一下 `docker run` 命令中比较常用的参数:
#### `-it`  建立一个可在终端交互的容器 
比如:
```bash
docker run -it --name local_nginx nginx:latest /bin/bash
# 或
docker exec -it local_nginx bin/bash
```
#### `-p`    用于宿主机和容器的端口绑定
绑定多个端口就设置多个映射
```bash
docker run -d -p 8088:80 -p 4433:443 nginx:latest
# 或    不写本地端口, docker 将帮你自动分配
docker run -d -p :80 -p :443 nginx:latest
# 或    加上 ip 就绑本地指定的 ip
docker run -d -p 127.0.0.1:8088:80 -p :443 nginx:latest
# 或    照样不写本地端口就随机分配
docker run -d -p 127.0.0.1::80 -p :443 nginx:latest
```
通过 `docker ps` 可以看一下上面两行命令的执行状态
```bash
➜  test docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                                           NAMES
57f65b46bd87        nginx:latest        "nginx -g 'daemon of…"   1 second ago         Up 3 seconds        0.0.0.0:32769->80/tcp, 0.0.0.0:32768->443/tcp   happy_zhukovsky
0c035ebabe44        nginx:latest        "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:8088->80/tcp, 0.0.0.0:4433->443/tcp     ecstatic_haibt
```
#### `-v` 将宿主机的卷挂载到容器中的指定目录
```bash
docker run -d -p 8088:80 -v /Users/gpf/Documents/docker_study/docker_test/www/:/usr/share/nginx/html/ nginx:latest
```
这里本地的目录要写绝对路径, 不然会报错, 这样一来, 本地的`/Users/gpf/Documents/docker_study/docker_test/www/` 就是容器中的`/usr/share/nginx/html/`, 就可以本地更改代码, 然后容器中运行
#### `-d` 后台运行
想查看日志的话就 `docker logs [containerID]` 就行
#### `docker exec` 执行 docker 容器中的命令
通常就是用来进入容器中搞七搞八的
```bash
docker exec -it 57f65b46bd87 /bin/bash
# 或
docker exsec -it local_nginx /bin/bash
```
这里注意两点:
   * `containerID` 在很多情况下都可以用 `container Name` 来代替, 很多情况是等价的
   * 命令最后的 `/bin/bash` 不是必须这么填, 而是执行的容器中的脚本, 如果你的镜像是 `alpine`版的就是 `sh`, 因为这个版本中就没有 bash 这个命令

#### `docker ps` 容器的运行状态

#### `docker stop [containerID 或 name]` 停止容器
目前版本也增加了 `docker container stop [containerID 或 name]` 其实作用是一样的, 不过 `docker container` 命令底下还有很多别的命令, docker 给各模块的命令做了细分
#### `docker rm [containerID 或 name]` 删除指定未运行的容器, 一个或多个
```bash
docker rm 6dee0a9b5232 582f708af9d3
```
#### `docker rmi [imageID 或 tag]` 删除宿主机指定的镜像
这里要注意如果这个镜像还有容器在使用就不能删除掉, 这个时候要先把对应的容器删掉才行
删除指定镜像的容器
```bash
docker stop $(docker ps | grep '这里写imageName' | awk '{ print $1}')
docker rm $(docker ps | grep '这里写imageName' | awk '{ print $1}')
```
删除临时构建的镜像
```bash
docker rmi $(docker images | grep '<none>' | awk '{ print $3}')
```
#### prune 大杀器
这一手还是慎用,一些情况下可造成 `rm -rf /*` 的效果
```bash
#移除所有未使用的镜像
docker image prune
#移除所有未运行的容器
docker container prune
#移除所有未使用的本地卷
docker volume prune
...
```
>PS: 因为有这一手, 所以可以看出官方的态度, 他们貌似也许可能没准大概差不多不建议把容器当成虚拟机一样把所有的东西都堆在一个镜像里面, 那样搞不止构建出来的镜像臃肿, 而且维护性移植性很差, 从目前网上的 docker 镜像资源来说, 基础镜像 alpine > debian > ubuntu > centos, 优先使用最小的基础构建, 然后整个 image 只为一个服务而构建, 比如 redis 镜像里只要 redis, 没有什么 MySQL, memcache 什么的, 多个独立的 service 才组成一个 APP, 里面各个组件替换的话不用考虑其他组件的环境依赖什么的, 当然, 这个也是看业务的实际需要, 不能为了拆分而拆分, 在这之间能找到最合适自己的才是工具给我们带来的便利


#### `docker network` 容器之间的互联
如果只是在一个容器里搞来搞去就真的是虚拟机了, docker 的强大之处就是它内部维护一个网络, 处在相同网络的容器是可以互通的
```bash
# 新建一个 docker 网络, -d bridge 是指定网络模式, 当前是桥接网络
docker network create -d bridge nginx_swarm
# 启动两个 nginx 容器, 分别命名 nginx_swarm_a nginx_swarm_b , 两者都加入了 nginx_swarm 这个网络  --rm 是当容器停止后自动删除
docker run -it --rm  --name nginx_swarm_a --network nginx_swarm  nginx /bin/bash
docker run -it --rm  --name nginx_swarm_b --network nginx_swarm  nginx /bin/bash
```
注意, 我们并没把接口暴露出去, 现在随便在一个容器中 `ping` 另一个容器
```bash
# 这是在 nginx_swarm_a 中
# 没有 ping 命令的先装一个 ping
# apt-get update && apt-get install -y iputils-ping
root@73d04107780f:/# ping -c 3 nginx_swarm_b
PING nginx_swarm_b (172.18.0.3) 56(84) bytes of data.
64 bytes from nginx_swarm_b.nginx_swarm (172.18.0.3): icmp_seq=1 ttl=64 time=0.084 ms
64 bytes from nginx_swarm_b.nginx_swarm (172.18.0.3): icmp_seq=2 ttl=64 time=0.161 ms
64 bytes from nginx_swarm_b.nginx_swarm (172.18.0.3): icmp_seq=3 ttl=64 time=0.146 ms

--- nginx_swarm_b ping statistics ---
```
docker 能自动的把 server name 转换成 ip, 我们只需要标明请求的是哪个容器, 而不是还要记住它的 ip 地址(当然 ip 地址也能设置)