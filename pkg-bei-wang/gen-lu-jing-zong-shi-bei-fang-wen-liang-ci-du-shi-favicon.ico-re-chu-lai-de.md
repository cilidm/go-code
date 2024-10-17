# 根路径总是被访问两次，都是favicon.ico惹出来的

#### 在使用golang做web开发的时候，如果在根路径上直接输出内容，你会发现，你的访问总是被执行 2 次。 根路径的 handler 是这个样子的

```go
func indexHandler(writer http.ResponseWriter, request *http.Request) {
    tms := time.Now().Format("2006-01-02 15:04:05.00000000")
    fmt.Println(tms,"Yes you in path: ",)
    fmt.Fprintln(writer, tms,"你正在访问的路径：index")
}
```

#### 可是执行效果是这样的 服务器显示

很显然，这样的服务器显示会对维护人员造成困扰。  
这个多出来的一次访问，是因为每当我们访问一个web站点的根路径时，会默认的访问寻找一下 favicon.ico 文件。对，就是我们常见的网站网址前面的那个小图标。  
如果你的web站点包含这样的小图标了。

#### 需要在模板页面中指定此图标的路径，这个问题就没有了。

```markup
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>后台管理界面</title>
    <link rel="icon" href="/userLogin/css/favicon.ico" type="image/x-icon" />
</head>
<body>
{{.}}
</body>
</html>
```

####  执行结果

#### 只显示一次

#### 如果没有ico文件，也可以通过代码回避这个问题。

```go
 //取消获取facicon.ico的访问
    if request.RequestURI == "/facicon.ico" {
        return
    }
```

#### 这段代码添加到根路径对应的handler中即可。 这样，你就不用担心，在跟路径页面上的访问记录会多一次的事情了。 

