# 分块下载

```go
package downloader

import (
	"fmt"
	"github.com/k0kubun/go-ansi"
	"github.com/schollz/progressbar/v3"
	"io"
	"net/http"
	"os"
	"path"
	"strings"
	"sync"
)

type Downloader struct {
	concurrency int
	resume      bool
	bar         *progressbar.ProgressBar
}

func NewDownloader(concurrency int, resume bool) *Downloader {
	return &Downloader{
		concurrency: concurrency, resume: resume,
	}
}

func (d *Downloader) Download(strUrl, fileName string) error {
	if fileName == "" {
		fileName = path.Base(strUrl)
		fmt.Println(fileName)
	}
	resp, err := http.Head(strUrl)
	if err != nil {
		return err
	}
	if resp.StatusCode == http.StatusOK && resp.Header.Get("Accept-Ranges") == "bytes" {
		return d.multiDownload(strUrl, fileName, int(resp.ContentLength))
	}
	return d.singleDownload(strUrl, fileName)
}

func (d *Downloader) setBar(length int) {
	d.bar = progressbar.NewOptions(
		length,
		progressbar.OptionSetWriter(ansi.NewAnsiStdout()),
		progressbar.OptionEnableColorCodes(true),
		progressbar.OptionShowBytes(true),
		progressbar.OptionSetWidth(50),
		progressbar.OptionSetDescription("downloading..."),
		progressbar.OptionSetTheme(progressbar.Theme{
			Saucer:        "[green]=[reset]",
			SaucerHead:    "[green]>[reset]",
			SaucerPadding: " ",
			BarStart:      "[",
			BarEnd:        "]",
		}),
	)
}

func (d *Downloader) multiDownload(strUrl, fileName string, contentLen int) error {
	d.setBar(contentLen)
	partSize := contentLen / d.concurrency
	// 创建部分文件的存放目录
	partDir := d.getPartDir(fileName)
	os.Mkdir(partDir, 0777)
	defer os.RemoveAll(partDir)
	var wg sync.WaitGroup
	wg.Add(d.concurrency)
	rangeStart := 0
	for i := 0; i < d.concurrency; i++ {
		go func(i, rangeStart int) {
			defer wg.Done()
			rangeEnd := rangeStart + partSize
			// 最后一部分，总长度不能超过 ContentLength
			if i == d.concurrency-1 {
				rangeEnd = contentLen
			}
			download := 0
			if d.resume {
				partFileName := d.getPartFileName(fileName, i)
				content, err := os.ReadFile(partFileName)
				if err == nil {
					download = len(content)
				}
				d.bar.Add(download)
			}
			d.downloadPartial(strUrl, fileName, rangeStart+download, rangeEnd, i)
		}(i, rangeStart)
		rangeStart += partSize + 1
	}
	wg.Wait()
	// 合并文件
	d.merge(fileName)
	return nil
}

func (d *Downloader) merge(fileName string) error {
	destFile, err := os.OpenFile(fileName, os.O_CREATE|os.O_WRONLY, 0666)
	if err != nil {
		return err
	}
	defer destFile.Close()
	for i := 0; i < d.concurrency; i++ {
		partFileName := d.getPartFileName(fileName, i)
		partFile, err := os.Open(partFileName)
		if err != nil {
			return err
		}
		io.Copy(destFile, partFile)
		partFile.Close()
		os.Remove(partFileName)
	}
	return nil
}

func (d *Downloader) singleDownload(strUrl, fileName string) error {
	resp, err := http.Get(strUrl)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	d.setBar(int(resp.ContentLength))

	f, err := os.OpenFile(fileName, os.O_CREATE|os.O_WRONLY, 0666)
	if err != nil {
		return err
	}
	defer f.Close()

	buf := make([]byte, 32*1024)
	_, err = io.CopyBuffer(io.MultiWriter(f, d.bar), resp.Body, buf)
	return err
}

func (d *Downloader) downloadPartial(strUrl, fileName string, rangeStart, rangeEnd, i int) error {
	if rangeStart >= rangeEnd {
		return nil
	}
	req, err := http.NewRequest("GET", strUrl, nil)
	if err != nil {
		return err
	}
	req.Header.Set("Range", fmt.Sprintf("bytes=%d-%d", rangeStart, rangeEnd))
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()
	flags := os.O_CREATE | os.O_WRONLY
	partFile, err := os.OpenFile(d.getPartFileName(fileName, i), flags, 0666)
	if err != nil {
		return err
	}
	defer partFile.Close()
	buf := make([]byte, 32*1024)
	_, err = io.CopyBuffer(io.MultiWriter(partFile,d.bar), resp.Body, buf)
	if err != nil {
		if err == io.EOF {
			return nil
		}
		return err
	}
	return nil
}

// getPartDir 部分文件存放的目录
func (d *Downloader) getPartDir(fileName string) string {
	return strings.SplitN(fileName, ".", 2)[0]
}

// getPartFilename 构造部分文件的名字
func (d *Downloader) getPartFileName(fileName string, partNum int) string {
	partDir := d.getPartDir(fileName)
	return fmt.Sprintf("%s/%s-%d", partDir, fileName, partNum)
}

```

```go
package downloader

import (
	"fmt"
	"runtime"
	"testing"
)

func TestNewDownloader(t *testing.T) {
	strUrl := "https://apache.claz.org/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz"
	fileName := ""
	concurrencyNum := runtime.NumCPU()
	err := NewDownloader(concurrencyNum, true).Download(strUrl, fileName)
	if err != nil {
		fmt.Println(err.Error())
	} else {
		fmt.Println("下载完毕")
	}
}
```

