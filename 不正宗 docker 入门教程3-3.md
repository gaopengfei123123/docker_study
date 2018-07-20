> 本章通过一个具体的demo来了解 docker-compose

#### docker-compose 是做什么的

在构建一个完整的服务时，我们通常启动一个容器， 一旦出现多个容器需要同时启动的时候手打是下下之策， 因为时间一长难免会忘记细节，写脚本也不是不可以，但是大家没有达成共识时脚本也很难维护...

`docker-compose` 就是来解决这个痛点， 只需要按照统一的格式书写，那么大家生成的容器也都是一致的， 在团队开发的时候扔一个配置好的   `docker-compose` 能节省很多时间和口水


####  配置 docker-compose 
这是我构建的一个开发环境的容器:[dnmp](https://github.com/gaopengfei123123/dnmp)

首先下载下来
```bash
git clone https://github.com/gaopengfei123123/dnmp.git && cd dnmp
```

我们第一件事就是先瞄一眼 `.env` 文件， 这里设置了很多常量，一会根据个人需求来调整

第二步才是打开 `docker-compose.yml` 文件， 看后缀都能猜到这是一个配置文件， 另外 `docker-compose.yml` **是根据缩进来进行分层的，注意书写格式**

```yml
# docker-compose.yml
# 语法版本( 3 和 2 区别有点大, 比如 3 取消了 volume_from 的相关语法)
version: "3"

networks:
  frontend:
    driver: ${NETWORKS_DRIVER}
  backend:
    driver: ${NETWORKS_DRIVER}


volumes:
  mysql_volume:
    driver: ${VOLUMES_DRIVER}
  redis_volume:
    driver: ${VOLUMES_DRIVER}
  rabbitmq_volume:
    driver: ${VOLUMES_DRIVER}
# 服务编排
services:
  # workspace:
  #   image: tianon/true
  #   container_name: dnmp-www
  #   volumes:
  #     - ./www:/usr/share/nginx/html

# NGINX #############################################
  nginx:
    container_name: dnmp-nginx
    build: 
      context: ./nginx
      args:
          - PHP_UPSTREAM_CONTAINER=${NGINX_PHP_UPSTREAM_CONTAINER}
          - PHP_UPSTREAM_PORT=${NGINX_PHP_UPSTREAM_PORT}
    depends_on:
      - php-fpm
    ports:
      - "${NGINX_HOST_HTTP_PORT}:80"
      - "${NGINX_HOST_HTTPS_PORT}:443"
    volumes:
      # 没必要把配置文件用卷来挂载, 不然就算配置更新了 nginx 也是要重启的

      # 挂载运行代码目录
      - ${APP_CODE_PATH_HOST}:/var/www
      # 挂载日志目录
      - ${NGINX_HOST_LOG_PATH}:/var/log/nginx
    # 使用 networks 取代 links 在同一个网络模式下的服务是互通的
    # 在service 中使用其他的 service 就直接调用 service 名就行, 不用管 ip 地址, docker 会自己维护一套
    networks:
      - frontend
      - backend


# PHP-FPM #############################################
  php-fpm:
    container_name: dnmp-php-fpm
    # 这里的args 是属于 build 下面的,用于构建./php-fpm/Dockerfile 文件中 ARG 参数指定 php 版本
    build: 
      context: ./php-fpm
      args:
        - PHP_VERSION=${PHP_VERSION}
    volumes:
      - ${APP_CODE_PATH_HOST}:/var/www
      - ./php-fpm/php${PHP_VERSION}.ini:/usr/local/etc/php/php.ini
    expose:
      - "9000"
    networks:
      - backend
    
  redis:
    container_name: dnmp-redis
    build:
      context: ./redis
      args:
        - REDIS_SET_PASSWORD=${REDIS_SET_PASSWORD}
    ports:
      - ${REDIS_HOST_PORT}:6379
    volumes:
      # 这里卷挂载的是本地文件
      # - ${DATA_PATH_HOST}/redis:/data
      # 这里创建一个 redis_volume来存放数据
      - redis_volume:/data

# Mysql #############################################
  mysql:
    container_name: dnmp-mysql
    # 镜像来源: https://github.com/docker-library/mysql/blob/fc3e856313423dc2d6a8d74cfd6b678582090fc7/5.7/Dockerfile
    image: mysql:${MYSQL_VERSION}
    volumes:
      # - ${DATA_PATH_HOST}/mysql:/var/lib/mysql
      - mysql_volume:/var/lib/mysql
    # 容器只要停止就会重启
    restart: always
    environment: 
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - ${MYSQL_HOST_PORT}:3306

```
接下来看看它的关键词都起着什么作用：
##### version  
这个规定了文件的版本， 既然有 3 就肯定不用 2 啊， 虽然两者没冲突，但是我喜欢， 2 和 3 版本之间有轻微的变动，具体区别你可以在写配置文件时产生的报错信息来体验一下
##### network  
```yml
networks:
  frontend:
    driver: ${NETWORKS_DRIVER}
  backend:
    driver: ${NETWORKS_DRIVER}

```
`${NETWORKS_DRIVER}` 是从 `.env` 文件中取的值， 下面的同理

这一块就相当于执行 `docker network create -d bridge frontend && docker network create -d bridge backend`
在本地持久化的建立一个网络配置，稍后方便容器进行连接， 当然这里也不止是一个 `driver` 参数，具体配置情况还是参考`docker network inspect dnmp_frontend` 来看一下

没有设置名字的配置当需要名字的时候会 `{当前docker-compose.yml文件名}_{key}` 这种格式

有了 `network` 配置就极大的简化了老版的 `--links` 命令， 只要属于同一个 network 就能互相访问到， 而不是每新增一个服务就要把原来的服务都 link 一遍 

##### volume
```yml
volumes:
  mysql_volume:
    driver: ${VOLUMES_DRIVER}
  redis_volume:
    driver: ${VOLUMES_DRIVER}
  rabbitmq_volume:
    driver: ${VOLUMES_DRIVER}
```
和 `network` 部分一样， 持久化的创建几个 `volume`, 相当于命令 `docker network create mysql_volume`等等

这算是 v3 的一个新特性， 在 v2 的时候， 为了共享数据大家会创建一个什么镜像都不继承的image， 所有容器的 volume 都会和它连接， 现在有了 `volume` 就没必要这么搞了
##### service
这个是本章的重点， 我们来看下面的例子中的注释， 按序号来
```yml
services:

  #1 创建一个服务叫做nginx服务
  nginx:
    #2 为了显得个性化一点，我们指定这个容器的名字叫做 dnmp-nginx
    container_name: dnmp-nginx
    #3 标明这个服务的 Dockerfile 的地址，用相对路径方便项目迁移
    build: 
      #3.1 相当于命令： 
      # docker build ./nginx -t dnmp-nginx \
      #     --build-arg PHP_UPSTREAM_CONTAINER=xxx \
      #     --build-arg PHP_UPSTREAM_PORT=zzz
      context: ./nginx
      #3.2 这里 ${NGINX_PHP_UPSTREAM_PORT} 的值是从 .env 文件中取的， args 属于构建时传入的参数
      args:
          - PHP_UPSTREAM_CONTAINER=${NGINX_PHP_UPSTREAM_CONTAINER}
          - PHP_UPSTREAM_PORT=${NGINX_PHP_UPSTREAM_PORT}
    #4  在启动这个容器之前先启动 php-fpm 这个容器 
    depends_on:
      - php-fpm
    #5 将本地端口和容器端口绑定， 本地哪个端口就看 .env 里怎么写的
    ports:
      - "${NGINX_HOST_HTTP_PORT}:80"
      - "${NGINX_HOST_HTTPS_PORT}:443"
    
    #6 设置需要挂载的卷, 这里时将本地目录和容器绑定， 也可以像 services.redis 那样和创建好的卷绑定 
    volumes:
      # 没必要把配置文件用卷来挂载, 不然就算配置更新了 nginx 也是要重启的

      # 挂载运行代码目录
      - ${APP_CODE_PATH_HOST}:/var/www
      # 挂载日志目录
      - ${NGINX_HOST_LOG_PATH}:/var/log/nginx
    # 使用 networks 取代 links 在同一个网络模式下的服务是互通的
    # 在service 中使用其他的 service 就直接调用 service 名就行, 不用管 ip 地址, docker 会自己维护一套
    
    #7 设置容器从属的网络， 同一个网络下可互相访问
    networks:
      - frontend
      - backend
```
在上文的 `#3` 步骤看其他的service也有直接使用`image`的， 这是直接从远程获取镜像的方式

配置文件写完了， 我们看下nginx的构建文件

```Dockerfile
# in file ./nginx/Dockerfile

#1 选择继承的镜像
FROM nginx:1.13.1-alpine
#2 各种标签
LABEL maintainer="GPF <5173180@qq.com>"

#3 容器中执行命令， 且把本地的配置文件添加进去
#https://yeasy.gitbooks.io/docker_practice/content/image/build.html
RUN mkdir -p /etc/nginx/cert \
    && mkdir -p /etc/nginx/conf.d \
    && mkdir -p /etc/nginx/sites

COPY ./nginx.conf /etc/ngixn/nginx.conf
COPY ./conf.d/ /etc/nginx/conf.d/
COPY ./cert/ /etc/nginx/cert/

COPY ./sites /etc/nginx/sites/


#4 这里也是设置构建参数， 不过相同 key 值会被 docker-compose 中的给覆盖掉
ARG PHP_UPSTREAM_CONTAINER=php-fpm
ARG PHP_UPSTREAM_PORT=9000
#5 ${PHP_UPSTREAM_CONTAINER} 就在构建时的参数使用方式
RUN echo "upstream php-upstream { server ${PHP_UPSTREAM_CONTAINER}:${PHP_UPSTREAM_PORT}; }" > /etc/nginx/conf.d/upstream.conf

#6 设置挂载的目录， 该目录下文件变化不会影响到容器
VOLUME ["/var/log/nginx", "/var/www"]

#7 设置目录运行时所处在容器中的目录地址
WORKDIR /usr/share/nginx/html

```

这里需要注意的时 `ARG` 和 `ENV` 的区别， 参考这篇文章： [Docker中 Arg 和 Env 的区别](http://blog.justwe.site/2018/06/28/docker-arg-env/)


#### 启动docker-compse
在配置好 `.env` 文件和 `docker-compose.yml` 配置文件后就可以启动它了， 命令也很简单，在同级目录下运行：
```bash
docker-compose up -d
```
它会自动创建`volume`，`network`，`services`， 而且相关的运行参数都是按着配置文件来的， 这样一来每个整个`docker-compose.yml`中的service就相当于时一个整体，每个服务又属于各自的容器，这样操控是不是节省了很多代码呢？

查看这些容器的运行状况也很是简单

```bash
docker-compose ps
# 或者使用更方便的一个工具： ctop ， github地址： https://github.com/bcicen/ctop
```

可操控单一容器一样， 但是它会把这一组容器都囊括了进去，操控起来只需要知道操控哪个服务，而一些参数就写在配置文件当中已经默认添加了


一些常用的命令：
```bash
# 终止整个服务集合
docker-compose stop

# 终止指定的服务 （这有个点就是启动的时候会先启动 depond_on 中的容器，关闭的时候不会影响到 depond_on 中的）
docker-compose stop nginx

# 查看容器的输出日志
docker-compose logs -f [services...]

# 构建镜像时不使用缓存（能避免很多因为缓存造成的问题）
docker-compose build --no-cache --force-rm

# 移除指定的容器
docker-compose rm nginx
```

还有个很不错的 docker-compose 项目就是 [laradock](http://laradock.io/), dnmp 就是仿照着它写的， 不过网络不好的情况下别运行 laradock， 它现在做的太臃肿了。。。。 看看它里面的镜像是怎么写的还是很有收获的


#### 导航

