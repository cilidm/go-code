# NFS

## Ubuntu

服务器端：

```bash
sudo apt install -y nfs-kernel-server
```

客户端：

```bash
sudo apt install -y nfs-common
```

修改配置文件

```bash
sudo vim /etc/exports

/home *(rw,sync,no_root_squash)
```

各段表达的意思如下，根据实际进行修改

```bash
/home  ：共享的目录
*    ：指定哪些用户可以访问
      * 所有可以ping同该主机的用户
      192.168.1.* 指定网段，在该网段中的用户可以挂载
      192.168.1.12 只有该用户能挂载
(ro,sync,no_root_squash)： 权限
    ro : 只读
    rw : 读写
    sync : 同步
    no_root_squash: 不降低root用户的权限
```

重启nfs服务

```bash
sudo /etc/init.d/nfs-kernel-server restart
```

查看

```bash
showmount -e localhost
```

到此，nfs的服务就搭建好了。

下面介绍客户端如何访问服务器

1、检查客户端和服务端的网络是否连通（ping命令）

ping + 主机IP

2、查看服务端的共享目录

```text
showmount -e + 主机IP
showmount -e 192.168.1.93
Export list for 192.168.1.93:
/home *
```

3、将该目录挂载到本地

```bash
sudo mount -t nfs -o nolock localhost:/home /mnt
                      # nfs-server-ip:/share-dir /local-dir
```

## Centos

```bash
sudo yum -y install nfs-utils
sudo vi /etc/sysconfig/nfs
sudo systemctl restart rpcbind.service 
sudo systemctl restart nfs-server.service
sudo  systemctl enable rpcbind.service
sudo systemctl enable nfs-server.service
sudo vi /etc/exports
/opt/share  172.xx.xx.xx(ro,async) 172.xx.xx.xx(ro,async) 
sudo systemctl restart nfs-server.service
showmount -e localhost

# 挂载
mkdir -p  /opt/share
mount -t nfs 172.xx.xx.xx:/opt/share  /opt/share

ps aux|grep admin
```

