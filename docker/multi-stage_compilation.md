使用 Docker 来多阶段编译 Golang
装环境太难， 我的天啊， 我每次都要装环境， 我们可以用下面的方法 So easy 随时切换 Golang 版本。

如果你的 Golang 项目依赖存于 vendor 下，那么我们可以使用多阶段构建并打包成容器镜像，Dockefile 如下：
```Dockfile
FROM golang:1.12-alpine AS go-build

WORKDIR /go/src/github.com/hunterhug/fafacms

COPY core /go/src/github.com/hunterhug/fafacms/core
COPY vendor /go/src/github.com/hunterhug/fafacms/vendor
COPY main.go /go/src/github.com/hunterhug/fafacms/main.go

RUN go build -ldflags "-s -w" -o fafacms main.go

FROM alpine:3.9 AS prod

WORKDIR /root/

COPY --from=go-build /go/src/github.com/hunterhug/fafacms/fafacms /bin/fafacms
RUN chmod 777 /bin/fafacms
CMD /bin/fafacms $RUN_OPTS
```
其中 github.com/hunterhug/fafacms 是你的项目。使用 golang:1.12-alpine 来编译二进制，然后将二进制打入基础镜像：alpine:3.9，这个镜像特别小。

编译：

`sudo docker build -t hunterhug/fafacms:latest .`
我们多了一个镜像 hunterhug/fafacms:latest , 而且特别小， 才几M 。

运行：

`sudo docker run -d  --net=host  --env RUN_OPTS="-config=/root/fafacms/config.json" hunterhug/fafacms`
可是，如果我们用了 cgo， 那么请将 Dockerfile 改为：
```Dockfile
FROM golang:1.12 AS go-build

WORKDIR /go/src/github.com/hunterhug/fafacms

COPY core /go/src/github.com/hunterhug/fafacms/core
COPY vendor /go/src/github.com/hunterhug/fafacms/vendor
COPY main.go /go/src/github.com/hunterhug/fafacms/main.go

RUN go build -ldflags "-s -w" -o fafacms main.go

FROM bitnami/minideb-extras-base:stretch-r165 AS prod

WORKDIR /root/

COPY --from=go-build /go/src/github.com/hunterhug/fafacms/fafacms /bin/fafacms
RUN chmod 777 /bin/fafacms
CMD /bin/fafacms $RUN_OPTS```
```