# kubeadm

## docker安装

```bash
# 先安装docker
# 省略

cat <<EOF > /etc/yum.repos.d/kubernetes.repo[kubernetes]name=Kubernetesbaseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/enabled=1gpgcheck=1repo_gpgcheck=1gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg        https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpgEOF

yum makecache

sudo yum -y install kubelet-1.18.6  kubeadm-1.18.6  kubectl-1.18.6

rpm -aq kubelet kubectl kubeadm

# 开机启动
sudo systemctl enable kubelet
```

## Kubeadm

#### Kubeadm init: 集群的快速初始化，部署Master节点的各个组件

#### kubeadm join: 节点加入到指定集群中

#### kubeadm token: 管理用于加入集群时使用的认证令牌 \(如list，create\)

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

## 其他问题可参考页面

[https://blog.csdn.net/baidu\_38432732/article/details/105662626](https://blog.csdn.net/baidu_38432732/article/details/105662626)

