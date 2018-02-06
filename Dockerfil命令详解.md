
- FROM xxx          指定了继承的基础镜像
- RUN               执行命令, 多个 `RUN` 将生成多层镜像,所以建议命令尽量用 `&&` 符号链接
- COPY  src tar     将指定的上下文路径当中的文件拷贝到镜像中
- ADD   src tar     功能等同 `COPY` 但是这个 `src`可以是外部链接,如果是压缩文件,格式为 `gzip`,`bzip`和`xz`的将会自动解压,因为会自动执行一些操作,不清楚后果的推荐用 `COPY`
- CMD               容器的启动命令, 可以是 `CMD echo $HOME` 也可以是 `CMD ["echo","$HOME"]`, 因为最后执行的时候会转化成`CMD ["sh","-c","echo $HOME"]`   , **只能出现一次**
- ENTRYPOINT        和`CMD`类似,不同的是可以在 `docker build`的时候追加参数,而`CMD`命令不可以, 比如 `CMD ["somethisn.sh"]` 是在`docker run`的时候启动一个脚本,如果希望执行脚本时添加唉参数比 如 `docker run image_name -u root -p `这类的命令,参数不适合写死在 Dockerfile 中的,就需要用到这个`ENTRYPOINT`这个指令了 **只能出现一次**
- ENV               设置环境变量,支持的命令有`ADD,COPY,ENV, EXPOSE,LABEL,USER,WORKDIR,VOLUME,STOPSIGNAL,ONBUILD`
- ARG               功能同 `ENV` 就是写在这里的不会在`docker run`中出现,只是在`docker build`时发生作用
- VOLUME            定义匿名卷, 比如 `VOLUME /data`就是将容器内的`/data`目录作为匿名区,当其中的文件发生修改时不会记录进容器的存储层,运行时覆盖这个挂载设置`docker run -d -v mydata:/data xxxx`
- EXPOSE            端口声明,这里指的是容器要暴露出来的端口, `docker run -p 80:8080 xxx`这一句就是将本地的80端口绑定到容器的8080端口,Dockerfile 里面就是`EXPOSE 8080`
- WORKDIR           指定工作目录,如果目录不存在则创建,像使用`RUN /app`紧接着一条就是`RUN echo "hello" > world.txt` 这可能不会找 `/APP/world.txt`这个,因为 `RUN` 每多一个就多一层容器
- USER              指定当前的用户
- HEALTHCHECK       检测容器是否正常运行,1.12版本之前容器中出现错误的时候会报错但是仍然在接受用户请求, 这个命令就是检测容器的状态,正常为`healthy` 出错时`unhealthy`, 存在三个参数:检测周期(默认30s),执行超时(默认30s),连续失败次数(默认3次)
- ONBUILD           当这个 `Dockerfile` 被其他的 `Dockerfile`中`FROM`继承的时候运行,本身自己构建的时候不会自行 `ONBUILD RUN ["echo", "$HOME"]` 比如这一条就是被继承的时候才会执行


> 在执行的时候,通常是 `docker build .` 这里的 `.`并不是指构建`Dockerfile`的文件位置,而是指的上下文(Context)的路径,是告诉 docker 服务器需要从哪取拿些`COPY,ADD`等等的源文件路径, 执行的时候默认是上下文目录下名为`Dockerfile`文件,指定要执行的`Dockerfile`文件的是 `docker build -f /path/to/Dockerfile  /path/to/Context` 这样的 