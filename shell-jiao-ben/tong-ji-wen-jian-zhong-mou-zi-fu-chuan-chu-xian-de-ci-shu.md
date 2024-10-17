# 统计文件中某字符串出现的次数

```bash
#!/bin/bash
file=/etc/passwd
str="root"
#每读取一行文件内容，就从第 1 列循环到最后 1 列，
#依次判断是否包含 root 关键词，若包含则 x++
awk -F: -v str=$str '
 BEGIN{i=0;x=0}
 {while(i<=NF)
 {if($i~str)
 {x++}
 i++
 }
 }
 END{print str"的数量是：" x}
' $file
```
