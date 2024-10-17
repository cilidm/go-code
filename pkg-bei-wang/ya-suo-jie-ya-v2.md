# 压缩解压v2

```go
package zipv2

import (
	"bytes"
	"errors"
	f "github.com/cilidm/toolbox/file"
	"github.com/mzky/zip"
	"io"
	"os"
	"path"
	"path/filepath"
	"runtime"
	"strings"
)

func IsZip(zipPath string) bool {
	f, err := os.Open(zipPath)
	if err != nil {
		return false
	}
	defer f.Close()

	buf := make([]byte, 4)
	if n, err := f.Read(buf); err != nil || n < 4 {
		return false
	}

	return bytes.Equal(buf, []byte("PK\x03\x04"))
}

// FileIsExist 判断文件夹或文件是否存在, true为存在
func FileIsExist(fp string) bool {
	_, err := os.Stat(fp)
	return err == nil || os.IsExist(err)
}

// IsFile 判断是文件还是目录
func IsFile(fp string) bool {
	fi, e := os.Stat(fp)
	if e != nil {
		return false
	}
	return !fi.IsDir()
}

// TrimValueFromArray 去除数组中指定元素
func TrimValueFromArray(strArray []string, trimValue string) []string {
	newArray := make([]string, 0)
	for _, v := range strArray {
		if strings.TrimSpace(trimValue) != strings.TrimSpace(v) {
			newArray = append(newArray, strings.TrimSpace(v))
		}
	}

	return newArray
}

// Zip password值可以为空""
func Zip(filePath, zipPath, password string) error {
	dirPath, _ := path.Split(zipPath)
	err := f.IsNotExistMkDir(dirPath)
	if err != nil {
		return err
	}
	fz, err := os.Create(zipPath)
	if err != nil {
		return err
	}
	defer fz.Close()
	zw := zip.NewWriter(fz)
	defer zw.Close()
	filepath.Walk(filePath, func(path string, info os.FileInfo, err error) error {
		if err != nil {
			return err
		}
		if filePath == path {
			return nil
		}
		if !strings.HasSuffix(filePath, "/") {
			filePath = filePath + "/"
		}
		if runtime.GOOS == "windows" {
			filePath = strings.ReplaceAll(filePath, "\\", "/")
			path = strings.ReplaceAll(path, "\\", "/")
		}
		fileName := strings.ReplaceAll(path, filePath, "")
		if info.IsDir(){
			fileName += "/"
		}
		// 写入文件的头信息
		var w io.Writer
		var errB error
		if password != "" {
			w, errB = zw.Encrypt(fileName, password, zip.AES256Encryption)
		} else {
			w, errB = zw.Create(fileName)
		}
		if errB != nil {
			return errB
		}

		if !info.IsDir() {
			fr, err := os.Open(path)
			if err != nil {
				return err
			}
			defer fr.Close()
			_, err = io.Copy(w, fr)
		}
		return nil
	})
	return zw.Flush()
}

// UnZip password值可以为空""
// 当decompressPath值为"./"时，解压到相对路径
func UnZip(zipPath, password, decompressPath string) error {
	if !FileIsExist(zipPath) {
		return errors.New("找不到压缩文件")
	}
	if !IsZip(zipPath) {
		return errors.New("压缩文件格式不正确或已损坏")
	}
	r, err := zip.OpenReader(zipPath)
	if err != nil {
		return err
	}
	defer r.Close()

	for _, fi := range r.File {
		if password != "" {
			if fi.IsEncrypted() {
				fi.SetPassword(password)
			} else {
				return errors.New("must be encrypted")
			}
		}
		fp := path.Join(decompressPath, fi.Name)
		if fi.FileInfo().IsDir(){
			err = f.IsNotExistMkDir(fp)
			if err != nil {
				return err
			}
			continue
		}
		_ = os.MkdirAll(path.Dir(fp), os.ModePerm)
		w, errA := os.Create(fp)
		if errA != nil {
			return errors.New("无法创建解压文件")
		}
		fr, errB := fi.Open()
		if errB != nil {
			return errors.New("解压密码不正确")
		}
		if _, errC := io.Copy(w, fr); errC != nil {
			return errC
		}
		fr.Close()
		w.Close()
	}
	return nil
}

```

