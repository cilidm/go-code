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

