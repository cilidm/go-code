# 批量下载有序文件(pdf、图片、视频等)

```bash
#!/bin/bash
#本脚本进行有序的网络资料批量下载操作（如 01.jpg，02.jpg，03.jpg）
url="http://www.test.com"
type=jpg
Dpath=/mnt
echo "开始下载..."
for num in `seq 10`
do
    echo -n "正在下载$num.$type"
    curl -s ${url}/${num}.${type} -o $Dpath/${num}.$type #-o 选项,curl指定下载文件另存为
    if [ $? -eq 0 ]; then
        echo -e " [\e[32mOK\e[0m]"
    else
        echo -e " [\e[31mERROR\e[0m]"
    fi
    sleep 1
done
```
