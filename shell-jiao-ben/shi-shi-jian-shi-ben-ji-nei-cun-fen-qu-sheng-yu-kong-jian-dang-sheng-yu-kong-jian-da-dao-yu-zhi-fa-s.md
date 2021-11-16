# 实时监视本机内存、/分区剩余空间，当剩余空间达到阈值发送 报警邮件给 root 管理员

```bash
#!/bin/bash
disk_value=1000 #设置/分区监测空间
mem_value=500 #设置内存监测空间
disk_size=`df -m / | awk '/\//{print $4}'` #获取/的剩余硬盘空间，单位 M
mem_size=`free -m | awk '/Mem/{print $4}'` #获取剩余内存空间，单位 M
while :
do
    if [ $disk_size -le $disk_value -o $mem_size -
        le $mem_value ]; then #判断条件
        echo "Insufficient resources,资源不足" | mail - s "Warning" root #达到阈值发送邮件给 root
    fi
done
```
