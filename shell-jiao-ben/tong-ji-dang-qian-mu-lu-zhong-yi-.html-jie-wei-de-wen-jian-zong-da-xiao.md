# 统计当前目录中以.html 结尾的文件总大小

```bash
# 方法 1：  
find . -name "*.html" -exec du -k {} \; |awk '{sum+=$1}END{print sum}'

# 方法 2：
for size in $(ls -l *.html |awk '{print $5}'); do
    sum=$(($sum+$size))
done
echo $sum
```
