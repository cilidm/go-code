# viper写入配置文件

```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

type Config struct {
	DB DBConf
	Redis RedisConf
}

type DBConf struct {
	Type string
	User string
	Pwd  string
	Host string
	Name string
	Port string
}

type RedisConf struct {
	Addr string
	PWD  string
	DB   int
}

func main() {
	v := viper.New()
	//设置读取的配置文件
	v.AddConfigPath("./")
	v.SetConfigFile("config.yaml")
	
	//v.SetDefault()
	v.Set("DB.Type", "mysql")
	v.Set("DB.Name", "base-system")
	v.Set("DB.User", "root")
	v.Set("DB.Pwd", "123456")
	v.Set("DB.Port", "3306")
	v.Set("DB.Host", "127.0.0.1")
	
	v.Set("Redis.Addr", "127.0.0.1:6379")
	v.Set("Redis.PWD", "")
	v.Set("Redis.DB", "0")
	
	v.WriteConfigAs("config.yaml")

	v.ReadInConfig()
	var conf Config
	v.Unmarshal(&conf)
	fmt.Println(conf)
}

```

