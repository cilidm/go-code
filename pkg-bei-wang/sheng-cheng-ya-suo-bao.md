# 生成压缩包

### zip

```go
package lib

import (
	"archive/zip"
	ezip "github.com/alexmullins/zip"
	"github.com/cilidm/toolbox/str"
	"io"
	"os"
	"path/filepath"
	"runtime"
	"strings"
)

func EncryptZip(src, dst, passwd string, blackFile []string) error {
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
		if runtime.GOOS == "windows" {
			src = strings.ReplaceAll(src, "\\", "/")
			path = strings.ReplaceAll(path, "\\", "/")
		}
		if str.IsContain(blackFile, info.Name()) {
			return nil
		}
		header, err := ezip.FileInfoHeader(info)
		if err != nil {
			return err
		}
		header.Name = strings.ReplaceAll(path, src, "")
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

### unzip

```go
package lib

import (
	"bytes"
	"fmt"
	"github.com/alexmullins/zip"
	"github.com/cilidm/toolbox/str"
	"golang.org/x/text/encoding/simplifiedchinese"
	"golang.org/x/text/transform"
	"io"
	"io/ioutil"
	"log"
	"os"
	"path/filepath"
)


//DeCompressZip 解压zip包
func DeCompressZip(zipFile, dest, passwd string, offset int64) error {
	zr, err := zip.OpenReader(zipFile)
	if err != nil {
		return err
	}
	defer zr.Close()

	for _, f := range zr.File {
		if passwd != "" {
			f.SetPassword(passwd)
		}
		var decodeName string
		if f.Flags == 1 {
			//如果标致位是1  则是默认的本地编码   默认为gbk
			i := bytes.NewReader([]byte(f.Name))
			decoder := transform.NewReader(i, simplifiedchinese.GB18030.NewDecoder())
			content, _ := ioutil.ReadAll(decoder)
			decodeName = string(content)
		} else {
			//如果标志为是 0 << 11也就是 2048  则是utf-8编码
			decodeName = f.Name
		}
		fpath := filepath.Join(dest, decodeName)
		if f.FileInfo().IsDir() {
			os.MkdirAll(fpath, os.ModePerm)
			continue
		}

		if err = os.MkdirAll(filepath.Dir(fpath), os.ModePerm); err != nil {
			return err
		}

		inFile, err := f.Open()
		if err != nil {
			return err
		}

		outFile, err := os.OpenFile(fpath, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, f.Mode())
		if err != nil {
			inFile.Close()
			return err
		}

		_, err = io.Copy(outFile, inFile)
		inFile.Close()
		outFile.Close()
		if err != nil {
			return err
		}
	}
	return nil
}

func DeCompressZipByte(zipFile, passwd string, offset int64) (map[string][]byte, error) {
	zr, err := zip.OpenReader(zipFile)
	if err != nil {
		return nil, err
	}
	defer zr.Close()
	var chunkMap = make(map[string][]byte)
	for _, f := range zr.File {
		if passwd != "" {
			f.SetPassword(passwd)
		}
		var decodeName string
		if f.Flags == 1 {
			//如果标致位是1  则是默认的本地编码   默认为gbk
			i := bytes.NewReader([]byte(f.Name))
			decoder := transform.NewReader(i, simplifiedchinese.GB18030.NewDecoder())
			content, _ := ioutil.ReadAll(decoder)
			decodeName = string(content)
		} else {
			//如果标志为是 0 << 11也就是 2048  则是utf-8编码
			decodeName = f.Name
		}

		inFile, err := f.Open()
		if err != nil {
			return nil, err
		}
		defer inFile.Close()

		var chunk []byte
		buf := make([]byte, 1024)
		for {
			//从file读取到buf中
			n, err := inFile.Read(buf)
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
		chunkMap[decodeName] = chunk
	}
	return chunkMap, nil
}

func DeCompressZipInfo(zipFile, passwd string, offset int64) ([]zip.File, error) {
	zr, err := zip.OpenReader(zipFile)
	if err != nil {
		return nil, err
	}
	defer zr.Close()

	blackName := []string{""}
	var fileMap []zip.File
	for _, f := range zr.File {
		if passwd != "" {
			f.SetPassword(passwd)
		}
		var decodeName string
		if f.Flags == 1 {
			//如果标致位是1  则是默认的本地编码   默认为gbk
			i := bytes.NewReader([]byte(f.Name))
			decoder := transform.NewReader(i, simplifiedchinese.GB18030.NewDecoder())
			content, _ := ioutil.ReadAll(decoder)
			decodeName = string(content)
		} else {
			//如果标志为是 0 << 11也就是 2048  则是utf-8编码
			decodeName = f.Name
		}
		fmt.Println(decodeName)
		if str.IsContain(blackName, decodeName) {
			fmt.Println(decodeName, "is in black name")
			continue
		}
		f.Name = decodeName
		fileMap = append(fileMap, *f)
	}
	return fileMap, nil
}

func GetZipFileByName(zipFile, passwd, name string) ([]byte, error) {
	zipr, err := zip.OpenReader(zipFile)
	if err != nil {
		log.Fatal(err)
	}
	defer zipr.Close()

	var chunkMap []byte
	for _, f := range zipr.File {
		if passwd != "" {
			f.SetPassword(passwd)
		}

		if f.Name != name {
			continue
		}

		inFile, err := f.Open()
		if err != nil {
			return nil, err
		}
		defer inFile.Close()

		var chunk []byte
		buf := make([]byte, 1024)
		for {
			//从file读取到buf中
			n, err := inFile.Read(buf)
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
		chunkMap = chunk
		break
	}
	return chunkMap, nil
}

```

### go mod

```text
module zip

go 1.16

require (
	github.com/alexmullins/zip v0.0.0-20180717182244-4affb64b04d0
	github.com/cilidm/toolbox v0.0.0-20210603034745-e82616c96f7b
	golang.org/x/text v0.3.7
)

```

