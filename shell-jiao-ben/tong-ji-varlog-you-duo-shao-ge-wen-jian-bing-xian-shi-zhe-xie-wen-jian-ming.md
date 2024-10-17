# 统计/var/log 有多少个文件，并显示这些文件名

```bash
#!/bin/bash
#方法 1
sum=0 #用于统计数量
for i in `ls /var/log/`
do
if [ -f $i ];then
let sum++
echo "文件名:$i"
fi
done
echo "文件总量为：$sum"

#方法 2
echo "文件总量为："`find /var/log -maxdepth 1 -type f | wc -l`
#查找 Linux 系统中的僵尸进程
#!/bin/bash
#awk 判断 ps 命令输出的第 8 列为 z 时，显示该进程的 PID 和进程命令
ps aux | awk '{if($8 == "Z" ){print $2,$11}}'
```
