---
description: 'https://www.cnblogs.com/walker-lin/p/11214127.html'
---

# docker

## Ubuntu

```bash
#!/bin/bash
# 安装docker
sudo apt-get update -y

sudo apt-get install -y apt-transport-https  ca-certificates  curl  gnupg-agent software-properties-common
    
sudo curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository  "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs)  stable"
   
sudo apt-get update -y

sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# 配置国内源
echo '{
    "registry-mirrors" : [
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com",
    "https://cr.console.aliyun.com/"
  ]
}
' >> /etc/docker/daemon.json

service docker restart
docker info|grep Mirrors -A 1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
```

```bash
echo '{
    "registry-mirrors" : [
    "http://hub-mirror.c.163.com",
    "https://cr.console.aliyun.com/"
  ]
}
' >> /etc/docker/daemon.json
```

## Centos

```bash
sudo yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine
    
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  

sudo yum install -y docker-ce docker-ce-cli containerd.io 

sudo systemctl enable docker
     
sudo systemctl start docker

# 关闭防火墙及其他

systemctl stop firewalld && systemctl disable firewalld  

setenforce 0

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

swapoff -a

sudo systemctl daemon-reload

sudo systemctl restart docker
```

## 其他

```bash
echo "your-ip your-domain-name" >> /etc/hosts
```

