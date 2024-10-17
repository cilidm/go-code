# kubectl

## 普通安装

### 第一步、添加源，更新缓存索引 支持https传送

```bash
apt update && apt install -y apt-transport-https 
```

### 添加访问公钥

```bash
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
```

 

### 添加源

```bash
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
```

### 更新缓存索引

```bash
apt update
```

### 查看kubectl可用版本

```bash
apt-cache madison kubeadm kubelet kubectl
```

### 第二步、进行安装 

```bash
apt install  kubectl kubelet kubeadm -y
或者指定版本安装
apt-get install -y kubelet=1.18.6-00 kubectl=1.18.6-00 kubeadm=1.18.6-00
```

### 第三步、开机自启

```bash
systemctl enable kubelet
```

## [国内Ubuntu20.04下安装kubectl和helm](https://my.oschina.net/ykbj/blog/4315903)

### 安装kubectl

```bash
# 更新apt安装源
cp /etc/apt/sources.list /etc/apt/sources.list.bak

sudo vi /etc/apt/sources.list

# 文件最上面添加国内源
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main

# 更新系统
sudo apt-get update && sudo apt-get install -y apt-transport-https

# 加入国内kubernetes-xenial源
echo "deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

# 更新
sudo apt-get update

# 安装kubectl
sudo apt-get install -y kubectl
```

### 安装helm

```bash
# 这个需要访问gihub，可以挂代理下载，后续我会写一个WSL2如何通过win10宿主机代理访问外网的教程
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh

chmod +x get_helm.sh

./get_helm.sh
```

## did you specify the right host or port?解决 <a id="articleContentId"></a>

```bash
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
source /etc/profile
# 普通用户
echo "export KUBECONFIG=/home/ubuntu/kubectl/config" >> ~/.bash_profile
source ~/.bash_profile

# 复制仪表盘kubeconfig到~/.kube/config 然后执行
echo "export KUBECONFIG=~/.kube/config" >> ~/.bash_profile
source ~/.bash_profile
```



