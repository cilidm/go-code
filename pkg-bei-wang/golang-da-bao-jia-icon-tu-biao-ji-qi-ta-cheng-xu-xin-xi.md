# golang打包加icon图标及其他程序信息

安装依赖

```go
go get github.com/akavel/rsrc
```

示例代码

```go
//go:generate rsrc -ico resource/icon.ico -manifest resource/goversioninfo.exe.manifest -o main.syso
package main

import (
	"os/exec"
)

func main() {
	cmd := exec.Command("cmd", "/C", "start https://www.baidu.com")
	if err := cmd.Run(); err != nil {
		panic(err)
	}
}


```

## 示例代码goversioninfo.exe.manifest

```markup
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <assemblyIdentity type="win32" name="demo" version="1.0.0.0" processorArchitecture="*"/>
</assembly>
```

## 注意事项

```go
//go:generate rsrc -ico resource/icon.ico -manifest resource/goversioninfo.exe.manifest -o main.syso
```

上述代码直接放在main.go正文头部，syso文件名必须和go文件名一致，resource目录与main.go同级



测试

```go
go generate
go build
```

注意，只有下面这2项参数发生变化时才需要执行`go generate`

 -ico resource/icon.ico 

-manifest resource/goversioninfo.exe.manifest 

运行`go generate`与main.go同级目录会生成一个main.syso，之后每次go build时都会自动加载main.syso加上icon 

