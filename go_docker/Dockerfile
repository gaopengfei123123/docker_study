# 编译环境
FROM golang:1.9-alpine
RUN apk --no-cache add git
WORKDIR /go/src/github.com/go/helloworld/
RUN go get -d -v github.com/go-sql-driver/mysql
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

# 运行环境
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=0 /go/src/github.com/go/helloworld/app .
CMD [ "./app" ]

# 多阶段构建
# 这里其实构建了两个镜像, 第一个 `FROM` 构建的是编译 go 的环境,第二个 `FROM` 是 go 的运行环境,而最终生成的就是最后一个 `FROM`构建的,而且因为使用的父容器是 `alpine:latest` 因此构建出来的容器只有 5mb 左右
# 运行`docker build -t gaofeifiy/go_docker .`, 再执行 `docker images` 会发现
# REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
# gaofeifiy/go_docker   latest              c7d10e0b700e        7 seconds ago       6.56MB
# <none>                <none>              17d84419bd41        16 seconds ago      293MB
# 第一行的镜像就只最终我们要用的镜像,第二行的是编译 go 时的镜像