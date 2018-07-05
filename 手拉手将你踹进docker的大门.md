>  从命名上就知道这是一篇简单粗暴的`docker`新手入门教程， 为什么要简单粗暴？ 我认为有自学能力的人帮他入门就够了， 不能自学的一时半会儿也教不会， 不符合入门教程的初衷， 建议出门左拐去找找视频教程...


### 安装环境

强烈推荐使用 `Docker for Mac` 或 `Docker for Windows`, 这两个工具已经将 `Kitematic` 和 `docker-compose` 集成好了， 至于这两个工具是做什么的咱们后面再说， `win10` 版本需要专业版的， 不然开启不了`Hyper-V`， `win7` 就别想了，不支持...

怎么安装在 [阿里云镜像容器服务](https://cr.console.aliyun.com/#/accelerator) 里面都说的很清楚了， 连国内镜像都给你安排好了， 咱们就进入下一话题
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
4. `--name local_nginx` 分配一个容器名， 不写的话会默认分配要给， 不过这个还是有点用的
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
就拿上图来说, `container`就是镜像的实例化, `image` 是容器的底层支撑, 其实他们的关系用代码中的类`Class`
来比喻是最合适的:

* Class 就是我们实际开发中写的一个代码集合, Object 是 Class 实例化之后生成的一种资源变量
* Image 也是预先写好的逻辑, 并存在一个地方, Container 是 Image 启动之后生成的一个虚拟系统
* 实例化出来的 Object 不会影响到 Class 中的内容
* 已经启动的 Container 也不会影响到 Image 中的逻辑
* Class 可以继承别的 Class, 从而继承它的特性
* Image 也是可以继承别的 Image, 并在它的基础上构建新的镜像
* 一个 Object 对应着一个 Class, 但是 一个 Class 可以实例化无数个 Object
* 同理, 一份 Image 可以生成无数个 Container, 这就是方便集群化部署的所在

简单的说 Container 就是 Image 的儿子, 模样和 Image 预想的一样, 但是 Container 运行之后会发生一些改变, 而且这种改变是可以保存的

