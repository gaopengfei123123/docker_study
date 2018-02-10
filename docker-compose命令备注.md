### build 构建(重构)服务容器
格式为: `docker-compose build [options] [service...]`
- `--force-rm` 删除构建时的临时容器
- `--no-cache` 构建不使用 cache
- `--pull`     始终尝试 pull 最新的镜像

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