# 调用浏览器打开指定网页

```go
cmd := exec.Command("cmd","/c start http://localhost:8009")
	err := cmd.Start()
	if err != nil{
		fmt.Println("无法打开浏览器")
	}
```

