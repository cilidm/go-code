# kubeadm

## #ubuntu 安装(版本有问题)  ubuntu走kubectl页面

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl

sudo curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

sudo tee /etc/apt/sources.list.d/kubernetes.list <<-'EOF'
deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main
EOF

sudo apt-get update

sudo apt-get install -y kubelet=1.18.6 kubeadm=1.18.6 kubectl=1.18.6
sudo apt-mark hold kubelet=1.12.8-00 kubeadm=1.12.8-00 kubectl=1.12.8-00

sudo systemctl enable kubelet && sudo systemctl start kubelet

# 关闭 swap
sudo swapoff -a
# 关闭防火墙
sudo ufw disable

# kubeadm初始化之后执行
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## centos 安装

```bash
# 先安装docker
# 省略

cat <<EOF > /etc/yum.repos.d/kubernetes.repo[kubernetes]name=Kubernetesbaseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/enabled=1gpgcheck=1repo_gpgcheck=1gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg        https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpgEOF

yum makecache -y

sudo yum -y install kubelet-1.18.6  kubeadm-1.18.6  kubectl-1.18.6

rpm -aq kubelet kubectl kubeadm

# 开机启动
sudo systemctl enable kubelet
```

## Kubeadm

#### Kubeadm init: 集群的快速初始化，部署Master节点的各个组件

#### kubeadm join: 节点加入到指定集群中

#### kubeadm token: 管理用于加入集群时使用的认证令牌 (如list，create)

#### kubeadm reset 重置集群，如删除 构建文件以回到初始状态

```bash
# 每台机器上都要执行：
systemctl enable docker.service

# 使用systemd作为docker的cgroup driver
sudo vi  /etc/docker/daemon.json    #（没有则创建）

{"exec-opts": ["native.cgroupdriver=systemd"]}

# 重启docker
systemctl daemon-reload  && systemctl restart docker

# 确保执行这句命令 出来的值是 systemd
docker info |grep Cgroup

# 初始化集群
# master执行
sudo kubeadm init --kubernetes-version=v1.18.6  --image-repository registry.aliyuncs.com/google_containers
 
# 其他
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# root执行
echo "export KUBECONFIG=/etc/kubernetes/admin.conf"  >> /etc/profile
source /etc/profile

#  查看token列表 
sudo kubeadm token list

# 重新生成token (默认24小时有效期)
sudo kubeadm token create  --print-join-command

# 其他节点加入（ 我们先不执行,先记下来）
sudo kubeadm join 192.168.0.53:6443 --token yd38ha.1u9xsqsleyw7gjuc \    
--discovery-token-ca-cert-hash sha256:2ef5f37cc530644ba1b6a5d99150af309e9c5d6a16933323ee18a7732811f3c1

# flannel
sudo sysctl net.bridge.bridge-nf-call-iptables=1

# 在主机上执行
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 注：如报错The connection to the server localhost:8080 was refused - did you specify the right host or port?
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
source /etc/profile

# 验证
kubectl get pods --all-namespaces

# 主节点执行
kubectl taint nodes --all node-role.kubernetes.io/master-

# 子节点执行
sudo sysctl net.bridge.bridge-nf-call-iptables=1

# 子节点加入节点  省略...





```

## kubeadm安装new

```bash
#!/bin/bash
hostnamectl set-hostname k8s-master
##1.关闭防火墙
systemctl stop firewalld
systemctl disable firewall
 
##2.关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/confi 
 
##3.关闭swap分区
sed -ri 's/.*swap.*/#&/' /etc/fstab
 
 
##5.在服务器节点添加hosts
#执行脚本
cat >> /etc/hosts << EOF
81.68.204.101 k8s-master
EOF
 
##6.将桥接得IPV4流量传递到iptables的链上
#执行脚本
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness = 0
EOF
 
#加载modprobe br_netfilter包
modprobe br_netfilter
 
#查看是否加载
lsmod | grep br_netfilter
 
#生效
sysctl --system
 
#&7.同步时间
yum install ntpdate -y
ntpdate time.windows.com
 
##8.开启ipvs
yum -y install ipset ipvsadm
 
#执行脚本
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
 
#授权、运行、检查是否加载
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
 
#检查是否加载
lsmod | grep -e ipvs -e nf_conntrack_ipv4

##1.安装Docker
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
  
sudo yum-config-manager \
    --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  

sudo yum install -y docker-ce docker-ce-cli containerd.io 
 
systemctl enable docker && systemctl start docker
 
docker version
 
##2.设置Docker镜像加速器
sudo mkdir -p /etc/docker
 
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "exec-opts": ["native.cgroupdriver=systemd"],	
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
 
sudo systemctl daemon-reload
 
sudo systemctl restart docker
 
 
##3.添加阿里云YUM软件源
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF


##4.安装kubeadm、kubelet和kubectl
yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
 
vim /etc/sysconfig/kubelet
#修改
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"
 
systemctl enable kubelet

##1.确定相关系统环境已准备好
swapoff -a
 
kubeadm reset
 
systemctl daemon-reload
 
systemctl restart kubelet
 
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X 
 
##2.拉取阿里云镜像，并init
kubeadm init --image-repository registry.aliyuncs.com/google_containers --apiserver-advertise-address=192.168.0.95 --service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16 --ignore-preflight-errors=Swap
 
#查看各个节点连接情况
kubectl get nodes
 
#（可选）默认的token有效期为24小时，当过期之后，该token就不能用了，这时可以使用如下的命令创建token：
# kubeadm token create --print-join-command
 
#（可选）生成一个永不过期的token
# kubeadm token create --ttl 0 

#下载calico插件
curl https://docs.projectcalico.org/manifests/calico.yaml -O
 
#应用
kubectl apply -f calico.yaml
 
#查看部署CNI网络插件进度
kubectl get pods -n kube-system

##查看节点状态
kubectl get nodes

#查看集群健康状态
kubectl get cs

#查看集群运行状态
kubectl cluster-info
```

## 其他问题可参考页面

{% embed url="https://blog.csdn.net/baidu_38432732/article/details/105662626" %}
