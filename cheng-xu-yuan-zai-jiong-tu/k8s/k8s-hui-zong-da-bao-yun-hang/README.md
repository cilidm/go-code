# k8s汇总(打包 运行)

## Dockerfile

```
# FROM 基于 golang:1.16-alpine
FROM golang:1.16-alpine3.14 AS builder

# ENV 设置环境变量
ENV GOOS=linux
ENV GOPROXY=https://goproxy.cn
ENV GOPATH=/opt/repo
ENV GO111MODULE=on

# 更换 apk add build-base 仓库镜像源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

RUN apk add build-base
# ADD 源路径 目标路径
COPY . $GOPATH/src/github.com/cilidm/pear-admin-golang

# RUN 执行 go build .
RUN cd $GOPATH/src/github.com/cilidm/pear-admin-golang && GOOS=linux CGO_ENABLED=1 GOARCH=amd64 go build -ldflags="-s -w" -installsuffix cgo -o pear-admin-golang main.go

# FROM 基于 alpine:latest
FROM alpine:latest

# RUN 设置代理镜像
RUN echo -e http://mirrors.ustc.edu.cn/alpine/v3.13/main/ > /etc/apk/repositories

# RUN 设置 Asia/Shanghai 时区
RUN apk --no-cache add tzdata  && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

# COPY 源路径 目标路径 从镜像中 COPY
COPY --from=builder /opt/repo/src/github.com/cilidm/pear-admin-golang /opt

# EXPOSE 设置端口映射
EXPOSE 8009/tcp

# WORKDIR 设置工作目录
WORKDIR /opt

# CMD 设置启动命令
CMD ["./pear-admin-golang"]
```

## 镜像推送阿里云

```shell
sudo docker login --username=xxx@xxx.com registry.cn-hangzhou.aliyuncs.com
# 为本地镜像添加tag
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/[命名空间]/pear-admin-go:v1.0.3
# 推送镜像
docker push registry.cn-hangzhou.aliyuncs.com/[命名空间]/pear-admin-go:v1.0.3
# 拉取镜像
docker pull registry.cn-hangzhou.aliyuncs.com/[命名空间]/pear-admin-go:v1.0.3
```

## Kubectl问题

rancher2.0 - 集群 - kubeconfig下载 保存为config

```
echo "export KUBECONFIG=/home/ubuntu/kubeconfig/config"  >> /etc/profile
source /etc/profile
```

## 永久关闭swap

注释掉 /etc/fstab 文件中的 swap配置

## 允许master节点运行pod

```bash
# 允许master节点部署pod
kubectl taint nodes --all node-role.kubernetes.io/master-
# 如果不允许调度
kubectl taint nodes master1 node-role.kubernetes.io/master=:NoSchedule
# 污点可选参数
# NoSchedule: 一定不能被调度
# PreferNoSchedule: 尽量不要调度
# NoExecute: 不仅不会调度, 还会驱逐Node上已有的Pod

```
