# 监控目录，将新创建的文件名追加到日志中

场景：记录目录下文件操作。需先安装 inotify-tools 软件包。

```bash
#!/bin/bash
MON_DIR=/opt
inotifywait -mq --format %f -e create $MON_DIR |\
while read files; do
    echo $files >> test.log
done
```
