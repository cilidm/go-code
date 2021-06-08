# 公众号

```go
package main

import (
	"context"
	"flag"
	"fmt"
	"io/ioutil"
	"net/http"
	"os"
	"strings"
	"time"

	"github.com/PuerkitoBio/goquery"
	"github.com/axgle/mahonia"
	"github.com/chromedp/chromedp"
	"github.com/gofrs/uuid"
	"github.com/siddontang/go-log/log"
)

var configFile string
var pathMap map[string]string
var sourceContent string
var projectPath string

func main() {
	//initCmd()
	//var err0 error = nil
	//if err0 = conf.LoadConf(configFile); err0 != nil {
	//  return
	//}
	// dir, _ := os.Getwd()
	// println(dir)
	// projectPath = dir
	// configFile = "./conf/download.conf"
	// content, err0 := ioutil.ReadFile(configFile)
	// byteContent, _ := ioutil.ReadFile("./conf/source.conf")
	// sourceContent = string(byteContent)
	//fmt.Println(sourceContent)
	// if err0 != nil {
	// 	return
	// }
	// path := string(content)
	path := "./"
	//创建map
	pathMap = make(map[string]string)
	ctxt, cancel := chromedp.NewContext(context.Background())
	defer cancel()

	var res string

	context.WithTimeout(ctxt, 15*time.Second)
	//https://ethfans.org/posts/wtf-is-the-blockchain
	// site := "https://ethfans.org/posts/wtf-is-the-blockchain"
	site := "https://mp.weixin.qq.com/s/Oc90iCYyZGj5uDKhj8eGWw"
	downloadWebChat(ctxt, site, res, path)
	// downloadSimpleHtml(ctxt, site, res, path)

}

func downloadSimpleHtml(ctxt context.Context, site string, res string, path string) {
	err := chromedp.Run(ctxt, visibleSimpleHtml(site, &res))
	if err != nil {
		return
	}
	fmt.Println("===============" + res)
	reader := strings.NewReader(res)
	//解析utf-8格式的串
	dec := mahonia.NewDecoder("utf-8")
	rd := dec.NewReader(reader)
	doc, err := goquery.NewDocumentFromReader(rd)
	if err != nil {
		log.Fatal(err)
	}
	html, err := doc.Html()
	fmt.Println("=========" + html)
	channels := make(chan string)
	doc.Find("img").Each(func(i int, selection *goquery.Selection) {
		img_url, _ := selection.Attr("src")
		fmt.Println("imgUrl:" + img_url)
		if strings.Trim(img_url, " ") == "" {
			return
		}
		go download(img_url, channels, path)
		fmt.Println("src = " + <-channels + "图片爬取完毕")
	})
	//全部结束后，替换文件
	for originalPath, newPath := range pathMap {
		sourceContent = strings.Replace(sourceContent, originalPath, newPath, -1)
	}
	dir, _ := os.Getwd()
	//这里路径会变化，需要注意
	println(dir)
	//将新的sourceContent输出
	fmt.Println(sourceContent)
	er := ioutil.WriteFile(projectPath+"/conf/outputs.conf", []byte(sourceContent), 0644)
	if er != nil {
		//file, _ := exec.LookPath(os.Args[0])
		//path, _ := filepath.Abs(file)
		//println(path)
		//dir2, _ := os.Executable()
		//exPath := filepath.Dir(dir2)
		//println(exPath)
		log.Error(er)
	}
}

func downloadWebChat(ctxt context.Context, site string, res string, path string) {
	err := chromedp.Run(ctxt, visible(site, &res))
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("===========" + res)
	reader := strings.NewReader(res)
	dec := mahonia.NewDecoder("utf-8")
	rd := dec.NewReader(reader)
	doc, err := goquery.NewDocumentFromReader(rd)
	//获取将要爬取的html文档信息
	if err != nil {
		log.Fatal(err)
	}
	html, err := doc.Html()
	fmt.Println("=====" + html)
	//创建管道
	channels := make(chan string)
	// 文章 .rich_media_area_primary_inner
	doc.Find("img").Each(func(i int, selection *goquery.Selection) {
		img_url, _ := selection.Attr("data-src")
		fmt.Println("imgUrl:" + img_url)
		if strings.Trim(img_url, " ") == "" {
			return
		}
		if strings.Index(img_url, "https") == -1 {
			return
		}
		index := strings.Index(img_url, "?")
		if index != -1 {
			rs := []rune(img_url)
			newUrl := string(rs[0:index])
			go download(newUrl, channels, path)
		} else {
			go download(img_url, channels, path)
		}
		//从管道消费
		fmt.Println("src = " + <-channels + "图片爬取完毕")
	})
}

func visibleSimpleHtml(host string, res *string) chromedp.Tasks {
	//sel := "body > div.site-content > article > main"
	return chromedp.Tasks{
		chromedp.Navigate(host),
		chromedp.Sleep(3 * time.Second),
		chromedp.InnerHTML("body", res, chromedp.ByQuery),
	}
}

//func LoadConf(filename string) error {
//    content, err := ioutil.ReadFile(filename)
//    if err != nil {
//        return err
//    }
//
//    conf := Conf{}
//    err = json.Unmarshal(content, &conf)
//    if err != nil {
//        return err
//    }
//    GConf = &conf
//    return nil
//}

func initCmd() {
	flag.StringVar(&configFile, "config", "./config/download.conf", "where download.conf is.")
	flag.Parse()
}

func download(img_url string, channels chan string, path string) {
	fmt.Println("准备抓取:" + img_url)
	uid, _ := uuid.NewV4()
	file_name := uid.String() + ".png"
	base_dir := path
	file_dir := base_dir + "\\"
	exists, err := PathExists(file_dir)
	if err != nil {
		fmt.Printf("get dir error![%v]\n", err)
		return
	}
	if !exists {
		os.MkdirAll(file_dir, os.ModePerm)
	}
	os.Chdir(file_dir)
	f, err := os.Create(file_name)
	if err != nil {
		log.Panic("文件创建失败")
	}
	defer f.Close()

	resp, err := http.Get(img_url)
	if err != nil {
		fmt.Println("http.get err", err)
	}
	body, err1 := ioutil.ReadAll(resp.Body)
	if err1 != nil {
		fmt.Println("读取数据失败")
	}
	defer resp.Body.Close()
	f.Write(body)
	pathMap[img_url] = "img/" + file_name
	//成功后将文件名传入管道内
	channels <- "![](img/" + file_name + ")                 " + "sourceUrl is:" + img_url
}

func PathExists(path string) (bool, error) {
	_, err := os.Stat(path)
	if err == nil {
		return true, nil
	}
	if os.IsNotExist(err) {
		return false, nil
	}
	return false, err
}

/**
  设置获取的html区域
*/
func visible(host string, res *string) chromedp.Tasks {
	sel := "#page-content"
	return chromedp.Tasks{
		chromedp.Navigate(host),
		chromedp.Sleep(3 * time.Second),
		chromedp.InnerHTML(sel, res, chromedp.ByID),
	}
}

```

