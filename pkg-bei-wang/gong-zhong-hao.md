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

**优化v0.1**

```go
package main

import (
	"context"
	"flag"
	"fmt"
	"github.com/PuerkitoBio/goquery"
	"github.com/axgle/mahonia"
	"github.com/chromedp/chromedp"
	"github.com/gofrs/uuid"
	"github.com/siddontang/go-log/log"
	"io/ioutil"
	"net/http"
	"os"
	"strings"
	"time"
)

var configFile string
var pathMap map[string]string
var sourceContent string
var projectPath string

func main() {
	pathMap = make(map[string]string)
	ctxt, cancel := chromedp.NewContext(context.Background())
	defer cancel()
	var res string
	context.WithTimeout(ctxt, 15*time.Second)
	site := "https://mp.weixin.qq.com/s/THleFV_uKMq4qsoODi2PdQ"
	downWechat(ctxt, site, res)
}

func downWechat(ctxt context.Context, site string, res string) {
	err := chromedp.Run(ctxt, visible(site, &res))
	if err != nil {
		log.Fatal(err)
	}
	reader := strings.NewReader(res)
	dec := mahonia.NewDecoder("utf-8")
	rd := dec.NewReader(reader)
	doc, err := goquery.NewDocumentFromReader(rd)
	//获取将要爬取的html文档信息
	if err != nil {
		log.Fatal(err)
	}
	dom := doc.Find(".rich_media_area_primary_inner")
	//dom := doc.Find("#js_content")
	htmlStr, err := dom.Html()
	if err != nil {
		log.Fatal(err)
	}
	title := strings.TrimSpace(doc.Find("#activity-name").Text())
	err = WriteToFile(title+".html", fmt.Sprintf(def, title, htmlStr), false)
	if err != nil {
		log.Fatal(err.Error())
	}
}
var def = `
<!DOCTYPE html>
<html lang="en">
<head>
    <title>%s</title>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="color-scheme" content="light dark">
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=0,viewport-fit=cover">
    <link href="index.css" rel="stylesheet">
    <link href="my.css" rel="stylesheet">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black">
    <meta name="format-detection" content="telephone=no">
</head>
<body>
    <div id="js_article" class="rich_media">
        <div id="js_top_ad_area" class="top_banner"></div>
        <div class="rich_media_inner">
            <div id="page-content" class="rich_media_area_primary">
                <div class="rich_media_area_primary_inner">
                    <div id="img-content" class="rich_media_wrp">
                        <h2 class="rich_media_title" id="activity-name">
                            字节跳动打造的轮子：Go 表单验证器
                        </h2>
                        <div id="meta_content" class="rich_media_meta_list">
                            <span class="rich_media_meta rich_media_meta_nickname" id="profileBt">
                                <a href="javascript:void(0);" id="js_name">
                                    Go语言中文网                      
                                </a>
                            </span>
                        </div>
                        %s
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
</html>`

var defHtml = `
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
	<link href="https://res.wx.qq.com/open/libs/weui/2.4.4/weui.min.css" rel="stylesheet">
	<link href="default.css" rel="stylesheet">
	<link rel="dns-prefetch" href="//res.wx.qq.com">
	<link rel="dns-prefetch" href="//mmbiz.qpic.cn">
	<link rel="dns-prefetch" href="https://wxa.wxs.qq.com">
	<link rel="shortcut icon" type="image/x-icon" href="//res.wx.qq.com/a/wx_fed/assets/res/NTI4MWU5.ico">
    <title>%s</title>
</head>
<body>
	<div id="js_article" class="rich_media">
		<div id="js_top_ad_area" class="top_banner"></div>
		<div class="rich_media_inner">
			<div id="page-content" class="rich_media_area_primary">%s</div>
		</div>
	</div>
</body>
</html>
`

func downloadWebChat(ctxt context.Context, site string, res string, path string) {
	err := chromedp.Run(ctxt, visible(site, &res))
	if err != nil {
		log.Fatal(err)
	}
	//fmt.Println("===========" + res)
	reader := strings.NewReader(res)
	dec := mahonia.NewDecoder("utf-8")
	rd := dec.NewReader(reader)
	doc, err := goquery.NewDocumentFromReader(rd)
	//获取将要爬取的html文档信息
	if err != nil {
		log.Fatal(err)
	}
	//html, err := doc.Html()
	//fmt.Println("=====" + html)
	//创建管道
	//channels := make(chan string)
	// 文章 .rich_media_area_primary_inner
	dom := doc.Find(".rich_media_area_primary_inner")
	htmlStr, err := dom.Html()
	if err != nil {
		log.Fatal(err)
	}
	title := dom.Find("#activity-name").Text()
	//auth := dom.Find("#js_name").Text()
	err = WriteToFile(title+".html", htmlStr, false)
	fmt.Println(err)
	//doc.Find("img").Each(func(i int, selection *goquery.Selection) {
	//	img_url, _ := selection.Attr("data-src")
	//	fmt.Println("imgUrl:" + img_url)
	//	if strings.Trim(img_url, " ") == "" {
	//		return
	//	}
	//	if strings.Index(img_url, "https") == -1 {
	//		return
	//	}
	//	index := strings.Index(img_url, "?")
	//	if index != -1 {
	//		rs := []rune(img_url)
	//		newUrl := string(rs[0:index])
	//		go download(newUrl, channels, path)
	//	} else {
	//		go download(img_url, channels, path)
	//	}
	//	//从管道消费
	//	fmt.Println("src = " + <-channels + "图片爬取完毕")
	//})
}

func WriteToFile(name string, content string, append bool) error {
	var fileObj *os.File
	var err error
	if append {
		fileObj, err = os.OpenFile(name, os.O_RDWR|os.O_CREATE|os.O_APPEND, 0644)
	} else {
		fileObj, err = os.OpenFile(name, os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0644)
	}
	if err != nil {
		return err
	}
	defer fileObj.Close()
	if _, err := fileObj.WriteString(content); err != nil {
		return err
	}
	return nil
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

