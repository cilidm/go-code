# 华为云安装docker

本教程适用于华为云Linux服务器，使用的操作系统是 centos7.6 . 文档中的 \#号 你懂的是什么，如果把这个都拷贝进去了，我就疯了

一、增加普通用户，别tmd用root操作 创建用户 shenyi 。不要使用root赤裸裸操作服务器

## useradd shenyi

## passwd shenyi  \(自行输入密码）

给shenyi赋予 sudo权限

## vi /etc/sudoers   编辑这个文件

在这一行下加入 root ALL=\(ALL\) ALL \(这一行是原来有的\) shenyi ALL=\(ALL\) ALL （这一行是我们要加入的\) 注意：保存的时候要键入 wq! \(因为这厮是只读文件\)

------接下来 请退出shell。 一律使用 shenyi 进行登录和操作，禁止随意使用root

二、修改主机名： 华为云的主机名是类似：hecs-x-xlarge-2-linux-20201101235704 ，太长了，看的恶心

## hostnamectl set-hostname  jtthink1  //当前主机名设置为  jtthink1    ,其他几台分别设置为 jtthink2 以此类推 ，不要把各个机器搞成一样

## hostnamectl set-hostname  jtthink2

## hostnamectl set-hostname  jtthink3

修改hosts文件。 sudo vi /etc/hosts .给新主机增加127.0.0.1 。不然tmd 你去ping jtthink1 显示的是局域网IP （重新登录终端 主机名就变了）

三、下载docker离线安装包 1、禁用 firewalld systemctl stop firewalld && systemctl disable firewalld 2、禁用selinux \(华为云 默认是禁用的，这步可以省略，getenforce 可以看状态。如果是开的，那么自行百度禁止掉\) 2、 [https://download.docker.com/linux/centos/7/x86\_64/stable/Packages/](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/) 目前我下载的是 19.03 版本 [https://download.docker.com/linux/centos/7/x86\_64/stable/Packages/docker-ce-19.03.3-3.el7.x86\_64.rpm](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-19.03.3-3.el7.x86_64.rpm) 下载好后 上传到 服务器上 你喜欢的位置（或者直接用wget 在服务器上下载，很快很丝滑\) 我的位置是 /home/shenyi/tools/docker-ce-19.03.3-3.el7.x86\_64.rpm 3、安装docker sudo yum install docker-ce-19.03.3-3.el7.x86\_64.rpm -y

耐心等待 不出意外 会出现2个错误： 第一个错误： Requires: containerd.io &gt;= 1.2.2-3 我们可以到这里去下载 ：[https://centos.pkgs.org/7/docker-ce-stable-x86\_64/containerd.io-1.2.13-3.1.el7.x86\_64.rpm.html](https://centos.pkgs.org/7/docker-ce-stable-x86_64/containerd.io-1.2.13-3.1.el7.x86_64.rpm.html) （版本比它高是可以的） 下载下来：wget [https://download.docker.com/linux/centos/7/x86\_64/stable/Packages/containerd.io-1.2.13-3.2.el7.x86\_64.rpm](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.13-3.2.el7.x86_64.rpm) 手工安装： sudo yum install -y containerd.io-1.2.13-3.2.el7.x86\_64.rpm 第二个错误： Requires: docker-ce-cli  
于是我们 wget [https://download.docker.com/linux/centos/7/x86\_64/stable/Packages/docker-ce-cli-19.03.3-3.el7.x86\_64.rpm](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-cli-19.03.3-3.el7.x86_64.rpm) \(注意：cli工具 要和 上面下载的docker-ce版本一致\) 接下来是安装cli:sudo yum install -y docker-ce-cli-19.03.3-3.el7.x86\_64.rpm

搞定后，继续安装docker,也就是再执行一次：sudo yum install docker-ce-19.03.3-3.el7.x86\_64.rpm -y

4、设置用户组 docker安装时默认创建了docker用户组，将普通用户加入docker用户组就可以不使用sudo来操作docker

sudo usermod -aG docker shenyi \(这里请把shenyi改成你的用户名\) 注意：光加入还不行，要么重新登录，要么执行 newgrp - docker \(改变当前用户的有效群组\)

5、由于使用的是 华为云，因此镜像加速器要使用华为的设置 （阿里云的镜像加速，之前课程讲过） 帮助文档看这里：[https://support.huaweicloud.com/usermanual-swr/swr\_01\_0045.html](https://support.huaweicloud.com/usermanual-swr/swr_01_0045.html)

## sudo mkdir -p /etc/docker     \#创建一个文件夹 叫做docker

利用tee 命令把下面的配置写入 daemon.json registry-mirrors的值请改成你们自己的地址

## sudo tee /etc/docker/daemon.json &lt;&lt;-'EOF'

{ "registry-mirrors": \["[https://0a6b87ac200025770fdec00b87313bc0.mirror.swr.myhuaweicloud.com](https://0a6b87ac200025770fdec00b87313bc0.mirror.swr.myhuaweicloud.com)"\]  
} EOF

6、启动docker systemctl start docker

强烈注意。重启docker 是两条命令：

## systemctl daemon-reload

## systemctl restart docker

7、尝试pull 一个镜像（反正后面要用，有用的一比\) docker pull alpine:3.12

