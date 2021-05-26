---
description: 'Go语言HttpRequest项目源码地址： https://github.com/kirinlabs/HttpRequest'
---

# HttpRequest

 具有快速**构建Headers**、**Cookies**、**设置超时时间**、**请求、耗时、打印请求信息**等功能



#### **安装：** <a id="%E5%AE%89%E8%A3%85%EF%BC%9A"></a>

```go
go get https://github.com/kirinlabs/HttpRequest
```

发送请求 导入HttpRequest

```go
import "github.com/kirinlabs/HttpRequest" 
```

实例化：

```go
req := HttpRequest.NewRequest() 
```

然后，尝试获取某个网页。我们来获取 Github 的公共时间线

```go
res,err := req.Get("https://api.github.com/events") 
```

返回一个res的Response对象和err的Error对象 

#### 自定义Transport <a id="%E8%87%AA%E5%AE%9A%E4%B9%89Transport"></a>

```go
var transport *http.Transport
 
func init() {   
    transport = &http.Transport{
        DialContext: (&net.Dialer{
            Timeout:   30 * time.Second,
            KeepAlive: 30 * time.Second,
            DualStack: true,
        }).DialContext,
        MaxIdleConns:          100, 
        IdleConnTimeout:       90 * time.Second,
        TLSHandshakeTimeout:   5 * time.Second,
        ExpectContinueTimeout: 1 * time.Second,
    }
}
 
func demo(){
    // Use http.DefaultTransport
    res, err := HttpRequest.Get("http://127.0.0.1:8080")
 
    // Use custom Transport
    res, err := HttpRequest.Transport(transport).Get("http://127.0.0.1:8080")
}
```

Post 请求

```go
//无参请求
res,err := req.Post("https://www.baidu.com")
 
//发送整数类型
res,err := req.Post("https://www.baidu.com",uint32(100))
 
//发送[]byte
res,err := req.Post("https://www.baidu.com",[]byte("bytes data"))
 
//发送*bytes.Reader,*strings.Reader,*bytes.Buffer
data := bytes.NewReader(buf []byte)
res,err := req.Post("https://www.baidu.com",data)
res,err := req.Post("https://www.baidu.com",strings.NewReader("string data"))
 
//请求体为文本
res,err := req.Post("https://www.baidu.com","hello")
 
 
//请求体为Json字符串
res,err := req.Post("https://www.baidu.com","{\"name\":\"github\"}")
 
 
//map传参
res.err := req.Post("https://www.baidu.com",map[string]interface{}{
    "name":"github",
    "type":1,
})
```

  也可以**不用实例化**，**直接发送请求**

```go
//快速发送Get请求
res,err := HttpRequest.Get("https://www.baidu.com")
 
res,err := HttpRequest.Get("https://www.baidu.com","title=baidu")
 
 
//快速发送Post请求
res,err := HttpRequest.Post("https://www.baidu.com")
 
//发送整数类型
res,err := req.Post("https://www.baidu.com",uint32(100))
 
//发送[]byte
res,err := req.Post("https://www.baidu.com",[]byte("bytes data"))
 
//发送*bytes.Reader,*strings.Reader,*bytes.Buffer
res,err := req.Post("https://www.baidu.com",bytes.NewReader(buf []byte))
res,err := req.Post("https://www.baidu.com",bytes.NewBuffer(buf []byte))
res,err := req.Post("https://www.baidu.com",strings.NewReader("string data"))
 
 
res,err := HttpRequest.Post("https://www.baidu.com","title=baidu&type=pdf")
 
res,err := HttpRequest.Post("https://www.baidu.com",map[string]interface{}{
    "title":"baidu",
})
 
 
//快速发送JSON请求
res,err := HttpRequest.JSON().Post("https://www.baidu.com",map[string]interface{}{
    "title":"baidu",
})
 
res,err := HttpRequest.JSON().Post("https://www.baidu.com",`{"title":"baidu","type":"pdf"}`)
```

传递URL参数 

你想为URL的查询字符串\(query string\)传递数据。如：手工构建URL,`http://www.baidu.com/index?key=value。`HttpRequest允许你使用第2个参数以字符串`"id=100&name=github"`或`map[string]interface{}{"id":10,"name":"github"}`字典的形式把数据传递给URL：

手工传参：

```go
res,err := req.Get("https://www.baidu.com/index?name=github")
```

字符串传参：

```go
res,err := req.Get("https://www.baidu.com/index?name=github","id=100&type=1")
```

map传参：

```go
res,err := req.Get("https://www.baidu.com/index?name=github",map[string]interface{}{
    "id":10,
    "type":1,
}
```



#### 响应内容 <a id="%E5%93%8D%E5%BA%94%E5%86%85%E5%AE%B9"></a>

能读取服务器响应的内容

```go
res,err := req.Post("https://api.github.com/events")
```

 获取服务器返回的内容：

```go
body,err := res.Body() 
fmt.Println(string(body))
```

获取服务器响应状态码：

```go
res.StatusCode()
```

获取服务器响应Headers:

```go
res.Headers()
```

返回一个map\[string\]string的字典

获取请求响应时间：

```text
res.Time()
```

Json响应内容 

HttpRequest内置JSON解码，来解析JSON数据：

```go
//Format the json return value
body, err := res.Json() 
 
fmt.Println(body)
```

 如果JSON解码失败，会返回一个err错误

**定制请求头** 

如果想为请求添加HTTP头部信息，只需要简单的传一个map给SetHeaders方法 

```go
req.SetHeaders(map[string]string{
    "Content-Type":"application/json",
    "Source":"api",
})
```

注：所有header值必须是字符串，SetHeaders可以多次调用，如果Key重复则会覆盖前面设置的值

#### BasicAuth 认证 <a id="BasicAuth%20%E8%AE%A4%E8%AF%81"></a>

如果想为请求添加HTTP头部信息，只需要简单的传一个map给SetHeaders方法

```go
req.SetBasicAuth("username","password")
```

#### CookieJar <a id="CookieJar"></a>

```go
j, _ := cookiejar.New(nil)
j.SetCookies(&url.URL{
	Scheme: "http",
	Host:   "127.0.0.1:8000",
}, []*http.Cookie{
	&http.Cookie{Name: "identity-user", Value: "83df5154d0ed31d166f5c54ddc"},
	&http.Cookie{Name: "token_id", Value: "JSb99d0e7d809610186813583b4f802a37b99d"},
})
res, err := HttpRequest.Jar(j).Get("http://127.0.0.1:8000/city/list")
defer res.Close()
if err != nil {
	log.Fatalf("Request error：%v", err.Error())
}
```

#### Proxy代理 <a id="Proxy%E4%BB%A3%E7%90%86"></a>

通过代理Ip访问

```go
proxy, err := url.Parse("http://proxyip:proxyport")
if err != nil {
	log.Println(err)
}
 
res, err := HttpRequest.Proxy(http.ProxyURL(proxy)).Get("http://127.0.0.1:8000/ip")
defer res.Close()
if err != nil {
	log.Println("Request error：%v", err.Error())
}
body, err := res.Body()
if err != nil {
	log.Println("Get body error：%v", err.Error())
}
log.Println(string(body))
```

#### JSON请求 <a id="JSON%E8%AF%B7%E6%B1%82"></a>

如果想以json方式发送请求，HttpRequest支持2种方式

设置Header头部信息

```go
req.SetHeaders(map[string]string{"Content-Type":"application/json"})
 
req.Post("https://www.baidu.com","{\"name\":\"github\"}")
```

调用req.JSON\(\)内置方法

```go
//直接发磅Json字符串参数
res,err := req.JSON().Post("https://www.baidu.com","{\"name\":\"github\"}")
 
//自动将Map以Json方式发送参数
res,err := req.JSON().Post("https://www.baidu.com",map[string]interface{}{
    "name":"github"
})
```

#### Cookie <a id="Cookie"></a>

```go
req.SetCookies(map[string]string{
    "name":"jason"
})
```

#### 超时   <a id="%E8%B6%85%E6%97%B6%C2%A0%20%C2%A0"></a>

```text
 req.SetTimeout(5)
```

#### 关闭证书验证 <a id="%E5%85%B3%E9%97%AD%E8%AF%81%E4%B9%A6%E9%AA%8C%E8%AF%81"></a>

当请求https协议时提示`x509: certificate signed by unknown authority`时，[可关闭证书验证](https://blog.bbzhh.com/index.php/archives/150.html)

```go
 req.SetTLSClient(&tls.Config{InsecureSkipVerify: true})
```

#### 调试模式 <a id="%E8%B0%83%E8%AF%95%E6%A8%A1%E5%BC%8F"></a>

```text
req.Debug(true)
```

#### 连接操作 <a id="%E8%BF%9E%E6%8E%A5%E6%93%8D%E4%BD%9C"></a>

而且还支持连接操作

```go
req := HttpRequest.NewRequest().Debug(true).SetTimeout(5).SetHeader()
```

**Respone对象** 

获取返回的Response对象

```text
resp.Response() 
```

**获取返回码**

```go
resp.StatusCode()
```

**获取Body主体信息**

```text
resp.Body()
```

返回\[\]byte和error

**获取请求耗时**

```text
resp.Time() string
```

单位是毫秒 

**获取真实Url**

```text
res.Url()
```

#### 实例代码 <a id="%E5%AE%9E%E4%BE%8B%E4%BB%A3%E7%A0%81"></a>

```go
package main
 
import (
   "github.com/kirinlabs/HttpRequest"
   "fmt"
   "log"
)
 
func main() {
 
   req := HttpRequest.NewRequest()
 
   // 设置超时时间，不设置时，默认30s
   req.SetTimeout(5)
 
   // 设置Headers
   req.SetHeaders(map[string]string{
      "Content-Type": "application/x-www-form-urlencoded", //这也是HttpRequest包的默认设置
   })
 
   // 设置Cookies
   req.SetCookies(map[string]string{
      "sessionid": "LSIE89SFLKGHHASLC9EETFBVNOPOXNM",
   })
 
   postData := map[string]interface{}{
      "id":    1,
      "title": "csdn",
   }
 
   // GET 默认调用方法
   resp, err := req.Get("http://127.0.0.1:8000?name=flyfreely")
 
   // GET 传参调用方法
   // 第2个参数默认为nil，也可以传参map[string]interface{}
   // 第2个参数不为nil时，会把传入的map以query传参的形式重新构造新url
   // 新的URL: http://127.0.0.1:8000?name=flyfreely&id=1&title=csdn
 
   //resp, err := req.Get("http://127.0.0.1:8000?name=flyfreely", postData)
 
   // POST 调用方法
 
   //resp, err := req.Post("http://127.0.0.1:8000", postData)
 
   if err != nil {
      log.Println(err)
      return
   }
 
   if resp.StatusCode() == 200 {
      body, err := resp.Body()
      
      if err != nil {
         log.Println(err)
         return
      }
      
      fmt.Println(string(body))
   }
 
   或者打印Json
   if resp.StatusCode() == 200 {
      body, err := resp.Json()
      
      if err != nil {
         log.Println(err)
         return
      }
      
      fmt.Println(body)
   }
}
```

