这里有一个使用 wordpress 的 `docker-compose.yml`的示例:
```yml
version: '3'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment: 
      MYSQL_ROOT_PASSWORD: gaofeifiy
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    ports:
      - 33060:3306
  
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8080:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
volumes:
  db_data: 
```

其中有一点奇怪的地方就是在`db`的数据挂载当中,是这么写的:
```yml
...
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
....

volumes:
  db_data: 
```

这是`docker-compose` version2 起开始的新语法,目的就是方便多个服务之间公用同一个卷
```yml
volumes:
  db_data: 
```
这种写法就是声明一个命名卷, 通过`docker volume ls` 可以列出来宿主机上的所有挂载的卷,而且这些卷**不会因为容器被删除而删除**,为的就是方便数据和服务分离

可以通过命令`docker inspect wordpress_db_data` 来查看卷的配置,这里`wordpress_db_data`就是我们刚才声明的那个卷的名字
```shell
➜  wordpress git:(master) ✗ docker inspect wordpress_db_data
[
    {
        "CreatedAt": "2018-02-13T04:29:29Z",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "wordpress",
            "com.docker.compose.volume": "db_data"
        },
        "Mountpoint": "/var/lib/docker/volumes/wordpress_db_data/_data",
        "Name": "wordpress_db_data",
        "Options": {},
        "Scope": "local"
    }
]

```
不过他的`mountpoint`的路径并不是宿主机上的路径,而依然是属于`docker`的路径,在`macOS`系统当中可以执行

```shell
screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty

# ls -ltrh /var/lib/docker/volumes
```
可以查看卷的列表, [参考资料1](https://forums.docker.com/t/host-path-of-volume/12277/6) [参考资料2](https://stackoverflow.com/questions/39175194/docker-compose-persistent-data-mysql)

清除本地的卷可以执行`docker volume prune`

[官方文档](https://docs.docker.com/compose/compose-file/#volumes)