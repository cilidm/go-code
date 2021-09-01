# golang中 关于json数据的处理 动态key 动态字段

一般的用法可以参考[https://www.cnblogs.com/yorkyang/p/8990570.html](https://www.cnblogs.com/yorkyang/p/8990570.html)

本文主要介绍json中动态字段 动态key的处理方法

例子一

json字段

```bash
{
    "friends": [
        {
            "id": 0,
            "name": "Robinson Woods"
        }
    ],
    "parent": [
        {
            "id": 1,
            "name": "Alejandra Mcdaniel"
        }
    ]
}
```

处理的方法，当成map来解析

```go
package main
 
import (
	"encoding/json"
	"log"
)
 
func main() {
	data := `{
		"friends": [
			{
				"id": 0,
				"name": "Robinson Woods"
			}
		],
		"parent": [
			{
				"id": 1,
				"name": "Alejandra Mcdaniel"
			}
		]
	}`
	type User struct {
		ID   int    `json:"id"`
		Name string `json:"name"`
	}
	dataMap := make(map[string][]User)
	json.Unmarshal([]byte(data), &dataMap)
	for k, v := range dataMap {
		log.Printf(`%v,%v`, k, v)
	}
}
```

例子二

```bash
{
    "ret":200,
    "data":{
        "32f49e76d7d16b76f0d49f15710b447236acfc90":{
            "torrent":[
                {
                    "sid":2,
                    "torrent_id":55171,
                    "info_hash":"32f49e76d7d16b76f0d49f15710b447236acfc90"
                }
            ]
        },
        "cf7d88fd656d10fe5130d13567aec27068b96676":{
            "torrent":[
                {
                    "sid":10,
                    "torrent_id":36,
                    "info_hash":"8bf47a8baa8bf7927ec61a850959bca8405482f5"
                },
                {
                    "sid":1,
                    "torrent_id":7233,
                    "info_hash":"07bb8defd21288a30a54ad793fe615f81afdbb2b"
                },
                {
                    "sid":8,
                    "torrent_id":5821,
                    "info_hash":"a69b89a8df716982ecceff4b98c532fc538501ae"
                }
            ]
        }
    },
    "msg":"",
    "version":"1.5.0"
}
```

处理的方法和例一类似

```go
package main
 
import (
	"encoding/json"
	"fmt"
)
 
type ApiSitesInfoHash struct {
	Ret  int `json:"ret"`
	Data Data `json:"data"`
	Msg     string `json:"msg"`
	Version string `json:"version"`
}
type Data map[string]Torrents   //自定义类型
type Torrents struct{ //结构体
	Torrent []Torrent `json:"torrent"`
}
type Torrent struct{
	Sid int `json:"sid"`
	Torrent_id int `json:"torrent_id"`
	Info_hash string `json:"info_hash"`
}
 
func main(){
	r:=`{
    "ret":200,
    "data":{
        "32f49e76d7d16b76f0d49f15710b447236acfc90":{
            "torrent":[
                {
                    "sid":2,
                    "torrent_id":55171,
                    "info_hash":"32f49e76d7d16b76f0d49f15710b447236acfc90"
                }
            ]
        },
        "cf7d88fd656d10fe5130d13567aec27068b96676":{
            "torrent":[
                {
                    "sid":10,
                    "torrent_id":36,
                    "info_hash":"8bf47a8baa8bf7927ec61a850959bca8405482f5"
                },
                {
                    "sid":1,
                    "torrent_id":7233,
                    "info_hash":"07bb8defd21288a30a54ad793fe615f81afdbb2b"
                },
                {
                    "sid":8,
                    "torrent_id":5821,
                    "info_hash":"a69b89a8df716982ecceff4b98c532fc538501ae"
                }
            ]
        }
    },
    "msg":"",
    "version":"1.5.0"
}`
 
	var dataMap ApiSitesInfoHash
 
	json.Unmarshal([]byte(r), &dataMap)
	for k, v := range dataMap.Data {
		fmt.Println(k)
		//fmt.Printf("%v",v.Torrent)
		for _,v1:=range v.Torrent{
			fmt.Println(v1.Sid,v1.Torrent_id,v1.Info_hash)
			//fmt.Printf("%v",v1.Sid)
		}
 
	}
}
```

再补充一个例子

#### 解析具有动态Key的对象

下面再做一下复杂的变化，如果把对象数组变为以`Fruit`的`Id`作为属性名的复合对象（object of object）比如：

```bash
"Fruit" : {
    "1": {
        "Name": "Apple",
        "PriceTag": "$1"
    },
    "2": {
        "Name": "Pear",
        "PriceTag": "$1.5"
    }
}
```



每个`Key`的名字在声明类型的时候是不知道值的，这样该怎么声明呢，答案是把`Fruit`字段的类型声明为一个`Key`为`string`类型值为`Fruit`类型的`map`

```go
type Fruit struct {
    Name string `json:"name"`
    PriceTag string `json:"priceTag"`
}
 
type FruitBasket struct {
    Name    string             `json:"name"`
    Fruit   map[string]Fruit   `json:"fruit"`
    Id      int64              `json:"id"`
    Created time.Time          `json:"created"`
}
```

可以运行下面完整的代码段试一下。

```go
package main
 
import (
    "fmt"
    "encoding/json"
    "time"
 
)
 
func main() {
    type Fruit struct {
        Name string `json:"name"`
        PriceTag string `json:"priceTag"`
    }
 
    type FruitBasket struct {
        Name    string             `json:"name"`
        Fruit   map[string]Fruit   `json:"fruit"`
        Id      int64              `json:"id"`
        Created time.Time          `json:"created"`
    }  
    jsonData := []byte(`
    {
        "Name": "Standard",
        "Fruit" : {
              "1": {
                    "name": "Apple",
                    "priceTag": "$1"
              },
              "2": {
                    "name": "Pear",
                    "priceTag": "$1.5"
              }
        },
        "id": 999,
        "created": "2018-04-09T23:00:00Z"
    }`)
 
    var basket FruitBasket
    err := json.Unmarshal(jsonData, &basket)
    if err != nil {
         fmt.Println(err)
    }
    for _, item := range basket.Fruit {
    fmt.Println(item.Name, item.PriceTag)
    }
}
```

