# 从官方PHP镜像构建
FROM       php

# 将index.php复制到容器内的/var/www目录下
ADD        index.php /var/www/

# 对外暴露8080端口
EXPOSE     8080

# 设置容器默认工作目录为/var/www
WORKDIR    /var/www/

# 容器运行后默认执行的指令
ENTRYPOINT ["php", "-S", "0.0.0.0:8080"]