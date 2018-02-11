### build 构建(重构)服务容器
格式为: `docker-compose build [options] [service...]`
- `--force-rm` 删除构建时的临时容器
- `--no-cache` 构建不使用 cache
- `--pull`     始终尝试 pull 最新的镜像

### up (这个才是最常用的)
`docker-compose up [options] [service...]`

直接输出`docker-compose up`就是将执行目录下的`docker-compose.yml`里的服务都自动构建,重新创建,并启动起来,同时将不同的容器关联起来,默认保持前台运行,同时也会输出各容器的输出日志

`docker-compose up -d` 将后台执行,在生产环境中使用

- `-d` 保持后台运行
- `--no-color` 不以颜色区分日志输出
- `--no-deps` 不启动服务关联的容器
- `--force-recreate` 强制重新创建容器
- `--no-recreate` 若容器已经存在,则不再构建, 与上者不同时使用
- `--no-build` 不自动构建缺失的镜像
- `-t , --timeout  TIMEOUT` 停止容器时候的超时,默认10s

### config 检测配置文件格式
如果正确则显示配置文件内容,如果错误则显示错误原因

### down 停止`up`启动的游戏

### exec 进入指定的容器

### images 列出 compose 中包含的镜像

### kill 发送信号停止容器
`docker-compose kill [options] [service...]`
比如 `docker kill -s web`

### logs 日志
`docker-compose logs [options] [service...]`
不同容器输出的日志颜色不同,不想有颜色就添加参数`--no-color`

### pause 暂停一个服务

### pull 拉取所需的镜像
`docker-compose pull [options] [service...]`

### version 输出 compose 的版本
一个简单的示例,具体代码参考 [这里](https://github.com/gaopengfei123123/docker_study/tree/master/docker_compose_demo)

```yml
version: '3'

services:
  web:                          # 一个 service 的名字
    build: .                    # 指出 Dockerfile 所在文件夹的路径, 而且每个 service 中必须有 build 或者 image
    ports:                      # 将容器内的接口绑定到宿主机上
      - "5000:5000"
  redis:
    image: "redis:alpine"       # 这就是使用的远程镜像
```

下面列出来几个常用的参数命令:
- `build` 通过 `Dockerfile` 来构建镜像,可以指定`Dockerfile`路径以及`context`的路径, 可以用`arg`指定构建时的参数 比如:
```yml
web:
    build:
        context: ./dir
        dockerfile: ./other-dir
        arg:
            buildno: 1 

```

- `image` 根据提供的镜像名拉取镜像

- `command` 覆盖容器启动后的默认命令比如:
```yml
command: echo "hello world"
```

- `container_name` 指定 `service` 容器的名字,如果没设置的话就是 容器_序号 这样, 比如 `web_1`,`db_1` 这样的
```yml
container_name: web-container
```

- `devices` 指定设备映射关系
```yml
devices:
    - "/dev/ttyUSB1:/dev/ttyUSB0"
```

- `depends_on` 指定容器依赖关系,会优先构建依赖的容器,比如:
```yml

version: '3'
services:
    web:
        build: .
        depends_on:
            - db
            - redis
    redis:
        image: redis
    db: 
        image: mysql
```
会先构建 `db`,`redis`再构建`web`


- `dns` 自定义 dns 服务列表
```yml
dns:
    - 8.8.8.8
    - 114.114.114.114
```

- `env_file` 从文件中提取环境变量,同变量名的会被最后的变量覆盖
```yml
env_file: .env

env_file:
    - .env1
    - .env2
```
格式必须是可以被 `#`注释的
```ini
PRO_ENV=product
APP_NAME=hahaha
```

- `environment` 设置环境变量,可以是`KEY=VALUE` 也可以只给出变量名,这将调取 `docker` 宿主机的环境变量
```yml
environment:
    ROCK_ENV: 'development'
    SESSION_SECRET:

environment:
    - ROCK_ENV=development
    - SESSION_SECRET

```

- `port` 将端口暴露到宿主机上
对于表示布尔值的变量建议添加引号,防止 yaml 解析的时候产生歧义,比如:`y|yes|TRYE|true|ON|on` 这一类的

- `volumes` 将宿主机的目录路径和容器中的路径挂载,可以设置访问模式,比如:
```yml
volumes:
    - /var/lib/mysql                # 同路径挂载
    - cache/:/tmp/cache
    - ~/configs:/etc/configs:/ro    # 支持相对路径, 支持访问模式
```

- 读取变量: compose会读取宿主机的环境变量和当前目录下的`.env`文件中的变量 比如:
```yml
version: '3'
services:
    db:
        image: "mongodb:($MONGO_VERSION)
```
这在启动的时候就会根据环境变量来调整要拉取的镜像