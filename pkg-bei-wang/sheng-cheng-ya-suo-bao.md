# 生成压缩包

```go
package main

import (
	"archive/zip"
	ezip "github.com/alexmullins/zip"
	"io"
	"log"
	"os"
	"path/filepath"
	"strings"
)

func main() {
	if err := EncryptZip("./zip-test", "./example.zip", "123456"); err != nil {
		log.Fatal(err)
	}
}

func EncryptZip(src, dst, passwd string) error {
	zipfile, err := os.Create(dst)
	if err != nil {
		return err
	}
	defer zipfile.Close()
	archive := ezip.NewWriter(zipfile)
	defer archive.Close()
	filepath.Walk(src, func(path string, info os.FileInfo, err error) error {
		if err != nil {
			return err
		}
		header, err := ezip.FileInfoHeader(info)
		if err != nil {
			return err
		}
		header.Name = strings.TrimPrefix(path, filepath.Dir(src)+"/")
		if info.IsDir() {
			header.Name += "/"
		} else {
			header.Method = zip.Deflate
		}
		// 设置密码
		header.SetPassword(passwd)
		writer, err := archive.CreateHeader(header)
		if err != nil {
			return err
		}
		if !info.IsDir() {
			file, err := os.Open(path)
			if err != nil {
				return err
			}
			defer file.Close()
			_, err = io.Copy(writer, file)
		}
		return err
	})
	return err
}

```

```go
package zip

import (
	"fmt"
	"github.com/alexmullins/zip"
	"io"
	"log"
)

func unzip(name, pwd string) (map[string][]byte, error) {
	zipr, err := zip.OpenReader(name)
	if err != nil {
		log.Fatal(err)
	}
	defer zipr.Close()
	var chunkMap = make(map[string][]byte)
	for _, z := range zipr.File {
		z.SetPassword(pwd)
		rr, err := z.Open()
		if err != nil {
			return nil, err
		}
		var chunk []byte
		buf := make([]byte, 1024)
		for {
			//从file读取到buf中
			n, err := rr.Read(buf)
			if err != nil && err != io.EOF {
				fmt.Println("read buf fail", err)
				break
			}
			//说明读取结束
			if n == 0 {
				break
			}
			//读取到最终的缓冲区中
			chunk = append(chunk, buf[:n]...)
		}
		chunkMap[z.Name] = chunk
	}
	return chunkMap, nil
}

```

