---
description: Gin框架引入embed
---

# Gin框架引入embed

```go
package main

import (
	"embed"
	"github.com/gin-gonic/gin"
	"html/template"
	"io/fs"
	"net/http"
)

//go:embed templates
var tmpl embed.FS

//go:embed static
var static embed.FS

func main() {
	r := gin.Default()

	t, _ := template.ParseFS(tmpl, "templates/*.html")
	r.SetHTMLTemplate(t)

	fads, _ := fs.Sub(static, "static")
	r.StaticFS("/static", http.FS(fads))

	r.GET("/", func(ctx *gin.Context) {
		ctx.HTML(200, "index.html", gin.H{"title": "Golang Embed 测试"})
	})
	r.Run(":8080")
}

```



```go
package router

import (
	"embed"
	"github.com/gin-gonic/gin"
	"go-spider-dy/app/controller"
	"go-spider-dy/app/middleware"
	"go-spider-dy/app/pkg/session"
	"html/template"
	"io/fs"
	"net/http"
)

func Instance(staticFs, templateFs embed.FS) *gin.Engine {
   gin.SetMode("debug")
   r := gin.Default()

   t, _ := template.ParseFS(templateFs, "template/**/**/*.html")
   r.SetHTMLTemplate(t)

   fads, _ := fs.Sub(staticFs, "static")
   r.StaticFS("/asset", http.FS(fads))

   r.Static("/download", "download")
   r.Use(gin.Logger())
   r.Use(gin.Recovery())
   r.Use(middleware.Cors())
   r.Use(session.EnableCookieSession("go-spider-dy"))

   r.GET("/", func(c *gin.Context) {
      c.Redirect(http.StatusFound, "/index")
   })
   r.GET("/write", func(c *gin.Context) {
      c.HTML(200,"test.html",nil)
   })
   r.GET("/index", controller.Index)
   r.GET("/pear_config", controller.PearConfig)
   r.GET("/menu_config", controller.MenuConfig)
   r.GET("/spider/list", controller.SpiderList)
   r.GET("/spider/json", controller.SpiderJson)
   r.GET("/spider/add", controller.SpiderAddPage)
   r.POST("/spider/add", controller.SpiderAdd)
   return r
}


package main

import (
	"embed"
	"fmt"
	"go-spider-dy/app/core/db"
	"go-spider-dy/app/router"
	"log"
)

//go:embed template
var templateFs embed.FS

//go:embed static
var staticFs embed.FS

func main() {
	db.Instance()
	port := ":8090"
	r := router.Instance(staticFs, templateFs)

	fmt.Printf(`	欢迎使用 go-spider-dy
	当前版本:V1.0.1
	默认前端文件运行地址:http://127.0.0.1%s
`, port)
	if err := r.Run(port);err != nil{
		log.Fatal(err.Error())
	}
}

```

前端引用方式：调用`static/component/pear/css/pear.css`

```text
<link rel="stylesheet" href="/asset/component/pear/css/pear.css" />
```

