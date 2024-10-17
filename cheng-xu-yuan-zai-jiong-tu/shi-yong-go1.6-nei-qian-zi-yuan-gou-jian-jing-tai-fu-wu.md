# 使用go1.6内嵌资源构建静态服务

```text
# content.txt内容
123
```

```text
package main

import (
	_ "embed"
	"fmt"
)

//go:embed test.txt

// 三个类型
// []byte 需import (_"embed")
// string
// embed.FS 多个文件和目录（注意是多个）

// var test string
var test []byte

func main() {
	fmt.Println(test)
}
```

```text
package main

import (
	"embed"
	_ "embed"
	"fmt"
)

// 资源目录地址： html lib

//go:embed html lib
var test embed.FS

func main() {
	list, _ := test.ReadDir("html")
	for _, item := range list {
		fmt.Println(item.Name(), item.Type(), item.IsDir())
	}
}

```

