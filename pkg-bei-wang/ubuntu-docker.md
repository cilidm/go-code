---
description: 'https://www.cnblogs.com/walker-lin/p/11214127.html'
---

# ubuntu docker

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
echo '{"registry-mirrors":["http://hub-mirror.c.163.com/"]}' >> /etc/docker/daemon.json

service docker restart
docker info|grep Mirrors -A 1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
```

