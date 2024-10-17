# 从 FTP 服务器下载文件

```bash
#!/bin/bash
if [ $# -ne 1 ]; then
echo "Usage: $0 filename"
fi
dir=$(dirname $1)
file=$(basename $1)
ftp -n -v << EOF # -n 自动登录
open 192.168.1.10 # ftp 服务器
user admin password
binary # 设置 ftp 传输模式为二进制，避免 MD5 值不同或.tar.gz 压缩包格式错误
cd $dir
get "$file"
EOF
```
