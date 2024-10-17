# 监测 Nginx 访问日志 502 情况，并做相应动作

假设服务器环境为 lnmp，近期访问经常出现 502 现象，且 502 错误在 重启 php-fpm 服务后消失，因此需要编写监控脚本，一旦出现 502，则 自动重启 php-fpm 服务。

```bash
#场景：
#1.访问日志文件的路径：/data/log/access.log
#2.脚本死循环，每 10 秒检测一次，10 秒的日志条数为 300 条，出现 502 的比例不低于 10%
（30 条）则需要重启 php-fpm 服务
#3.重启命令为：/etc/init.d/php-fpm restart
#!/bin/bash
###########################################################
#监测 Nginx 访问日志 502 情况，并做相应动作
###########################################################
log=/data/log/access.log
N=30 #设定阈值
while :
do
#查看访问日志的最新 300 条，并统计 502 的次数
err=`tail -n 300 $log |grep -c '502" '`
if [ $err -ge $N ]
then
/etc/init.d/php-fpm restart 2> /dev/null
#设定 60s 延迟防止脚本 bug 导致无限重启 php-fpm 服务
sleep 60
fi
sleep 10
done
```
