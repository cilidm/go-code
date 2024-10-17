# 批量修改文件名

## 方法1

```bash
# touch article_{1..3}.html
# ls
article_1.html article_2.html article_3.html
目的：把 article 改为 bbs
方法 1：
for file in $(ls *html); do
mv $file bbs_${file#*_}
# mv $file $(echo $file |sed -r 's/.*(_.*)/bbs\1/')
# mv $file $(echo $file |echo bbs_$(cut -d_ -f2)
done
```

## 方法2

```bash
for file in $(find . -maxdepth 1 -name "*html"); do
mv $file bbs_${file#*_}
done
```

## 方法3

```bash
rename article bbs *.html
```
