# 切割 Nginx 日志文件(防止单个文件过大,后期处理很困难)

```bash
#!/bin/bash
logs_path="/usr/local/nginx/logs/"
mv ${logs_path}access.log ${logs_path}access_$(date +"%Y%m%d" - d yesterday).log #以前一天时间标识
kill ‐USR1 `cat /usr/local/nginx/logs/nginx.pid` #USR1 无需重启而平滑的生成已经移走的日志文件
#需要先执行一次本脚本，
#或 crontab -e 手动写入计划任务，并注释下边命令
once=0
if [ $once -eq 0 ]; then
echo "0 1 * * * `readlink -f $0`" >> /var/spool/cron/$USER
sed -i "/^once=/s/0/1/" $0
fi
```
