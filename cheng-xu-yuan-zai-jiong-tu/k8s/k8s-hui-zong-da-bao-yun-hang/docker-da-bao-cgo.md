# docker打包cgo

编写Dockerfile时候注意以下几点。

可以参考，但不要照搬。

RUN apk add build-base

CGO\_ENABLED=1

这两个命令是关键。

```shell
# 编译
FROM golang:1.15.2-alpine  as builder
#ENV CGO_ENABLED=0
ENV GOOS=linux
ENV GOPROXY=https://goproxy.cn
ENV GO111MODULE=off
ENV GOPATH="/go/release:/go/release/src/gopathlib/"
#安装编译需要的环境gcc等
RUN apk add build-base

WORKDIR /go/release
#将上层整个文件夹拷贝到/go/release
ADD . /go/release/src
WORKDIR /go/release/src
#交叉编译，需要制定CGO_ENABLED=1，默认是关闭的
RUN  GOOS=linux CGO_ENABLED=1 GOARCH=amd64 go build -ldflags="-s -w" -installsuffix cgo -o ./bin/localized main.go

#编译
FROM alpine

COPY --from=builder  /go/release/src/bin/localized /app/localized-1.0/bin/localized
COPY --from=builder  /go/release/src/conf /app/localized-1.0/conf
COPY --from=builder  /go/release/src/log /app/localized-1.0/log

WORKDIR /app/localized-1.0

CMD ["/app/localized-1.0/localized"]
EXPOSE 9088
```
