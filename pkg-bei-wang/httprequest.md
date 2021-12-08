---
description: Go语言HttpRequest项目源码地址： https://github.com/kirinlabs/HttpRequest
---

# HttpRequest

&#x20;具有快速**构建Headers**、**Cookies**、**设置超时时间**、**请求、耗时、打印请求信息**等功能



#### **安装：** <a href="#e5-ae-89-e8-a3-85-ef-bc-9a" id="e5-ae-89-e8-a3-85-ef-bc-9a"></a>

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

返回一个res的Response对象和err的Error对象&#x20;

#### 自定义Transport <a href="#e8-87-aa-e5-ae-9a-e4-b9-89transport" id="e8-87-aa-e5-ae-9a-e4-b9-89transport"></a>

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

&#x20; 也可以**不用实例化**，**直接发送请求**

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

传递URL参数&#x20;

你想为URL的查询字符串(query string)传递数据。如：手工构建URL,`http://www.baidu.com/index?key=value。`HttpRequest允许你使用第2个参数以字符串`"id=100&name=github"`或`map[string]interface{}{"id":10,"name":"github"}`字典的形式把数据传递给URL：

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



#### 响应内容 <a href="#e5-93-8d-e5-ba-94-e5-86-85-e5-ae-b9" id="e5-93-8d-e5-ba-94-e5-86-85-e5-ae-b9"></a>

能读取服务器响应的内容

```go
res,err := req.Post("https://api.github.com/events")
```

&#x20;获取服务器返回的内容：

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

返回一个map\[string]string的字典

获取请求响应时间：

```
res.Time()
```

Json响应内容&#x20;

HttpRequest内置JSON解码，来解析JSON数据：

```go
//Format the json return value
body, err := res.Json() 
 
fmt.Println(body)
```

&#x20;如果JSON解码失败，会返回一个err错误

**定制请求头**&#x20;

如果想为请求添加HTTP头部信息，只需要简单的传一个map给SetHeaders方法&#x20;

```go
req.SetHeaders(map[string]string{
    "Content-Type":"application/json",
    "Source":"api",
})
```

注：所有header值必须是字符串，SetHeaders可以多次调用，如果Key重复则会覆盖前面设置的值

#### BasicAuth 认证 <a href="#basicauth-20-e8-ae-a4-e8-af-81" id="basicauth-20-e8-ae-a4-e8-af-81"></a>

如果想为请求添加HTTP头部信息，只需要简单的传一个map给SetHeaders方法

```go
req.SetBasicAuth("username","password")
```

#### CookieJar <a href="#cookiejar" id="cookiejar"></a>

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

#### Proxy代理 <a href="#proxy-e4-bb-a3-e7-90-86" id="proxy-e4-bb-a3-e7-90-86"></a>

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

#### JSON请求 <a href="#json-e8-af-b7-e6-b1-82" id="json-e8-af-b7-e6-b1-82"></a>

如果想以json方式发送请求，HttpRequest支持2种方式

设置Header头部信息

```go
req.SetHeaders(map[string]string{"Content-Type":"application/json"})
 
req.Post("https://www.baidu.com","{\"name\":\"github\"}")
```

调用req.JSON()内置方法

```go
//直接发磅Json字符串参数
res,err := req.JSON().Post("https://www.baidu.com","{\"name\":\"github\"}")
 
//自动将Map以Json方式发送参数
res,err := req.JSON().Post("https://www.baidu.com",map[string]interface{}{
    "name":"github"
})
```

#### Cookie <a href="#cookie" id="cookie"></a>

```go
req.SetCookies(map[string]string{
    "name":"jason"
})
```

#### 超时   <a href="#e8-b6-85-e6-97-b6-c2-a0-20-c2-a0" id="e8-b6-85-e6-97-b6-c2-a0-20-c2-a0"></a>

```
 req.SetTimeout(5)
```

#### 关闭证书验证 <a href="#e5-85-b3-e9-97-ad-e8-af-81-e4-b9-a6-e9-aa-8c-e8-af-81" id="e5-85-b3-e9-97-ad-e8-af-81-e4-b9-a6-e9-aa-8c-e8-af-81"></a>

当请求https协议时提示`x509: certificate signed by unknown authority`时，[可关闭证书验证](https://blog.bbzhh.com/index.php/archives/150.html)

```go
 req.SetTLSClient(&tls.Config{InsecureSkipVerify: true})
```

#### 调试模式 <a href="#e8-b0-83-e8-af-95-e6-a8-a1-e5-bc-8f" id="e8-b0-83-e8-af-95-e6-a8-a1-e5-bc-8f"></a>

```
req.Debug(true)
```

#### 连接操作 <a href="#e8-bf-9e-e6-8e-a5-e6-93-8d-e4-bd-9c" id="e8-bf-9e-e6-8e-a5-e6-93-8d-e4-bd-9c"></a>

而且还支持连接操作

```go
req := HttpRequest.NewRequest().Debug(true).SetTimeout(5).SetHeader()
```

**Respone对象**&#x20;

获取返回的Response对象

```
resp.Response() 
```

**获取返回码**

```go
resp.StatusCode()
```

**获取Body主体信息**

```
resp.Body()
```

返回\[]byte和error

**获取请求耗时**

```
resp.Time() string
```

单位是毫秒&#x20;

**获取真实Url**

```
res.Url()
```

#### 实例代码 <a href="#e5-ae-9e-e4-be-8b-e4-bb-a3-e7-a0-81" id="e5-ae-9e-e4-be-8b-e4-bb-a3-e7-a0-81"></a>

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
