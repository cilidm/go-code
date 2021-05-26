# Gin模拟大文件上传 分片存储 分片显示

```go
package main

import (
	"fmt"
	"io"
	"io/ioutil"
	"net/http"
	"os"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.New()
	r.Use(func(c *gin.Context) {
		defer func() {
			if err := recover(); err != nil {
				c.AbortWithStatusJSON(400, gin.H{"err": err})
			}
		}()
		c.Next()
	})
	r.GET("/", func(c *gin.Context) {
		c.Writer.Header().Set("Content-Type", "image/png")
		c.Writer.Header().Set("Transfer-Encoding", "chunked")
		// 模拟读取分片 读取分布式服务器中文件的分片数据
		for i := 0; i < 5; i++ {
			f, _ := os.Open(fmt.Sprintf("./files/img_%d.png", i))
			b, _ := ioutil.ReadAll(f)
			c.Writer.Write(b)
			c.Writer.(http.Flusher).Flush()
		}
	})
	r.POST("/file", func(c *gin.Context) {
		file, head, _ := c.Request.FormFile("file")
		block := head.Size / 5 // 模拟分片
		index := 0
		for {
			buf := make([]byte, block)
			n, err := file.Read(buf)
			if err != nil && err != io.EOF {
				panic(err.Error())
			}
			if n == 0 {
				break
			}
			saveBlock(fmt.Sprintf("img_%d.png", index), buf)
			index++
		}
		c.JSON(200, gin.H{"message": "OK"})
	})
	r.Run(":8080")
}

func saveBlock(name string, buf []byte) {
	save, _ := os.OpenFile("./files/"+name, os.O_CREATE|os.O_RDWR, 0600)
	defer save.Close()
	save.Write(buf)
}

```

