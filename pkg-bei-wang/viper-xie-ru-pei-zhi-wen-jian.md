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

// WriteConfig - 将当前的viper配置写入预定义的路径并覆盖（如果存在的话）。如果没有预定义的路径，则报错。
// SafeWriteConfig - 将当前的viper配置写入预定义的路径。如果没有预定义的路径，则报错。如果存在，将不会覆盖当前的配置文件。
// WriteConfigAs - 将当前的viper配置写入给定的文件路径。将覆盖给定的文件(如果它存在的话)。
// SafeWriteConfigAs - 将当前的viper配置写入给定的文件路径。不会覆盖给定的文件(如果它存在的话)。
// 根据经验，标记为safe的所有方法都不会覆盖任何文件，而是直接创建（如果不存在），而默认行为是创建或截断。
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
	//v.SafeWriteConfigAs("config.yaml")

	v.ReadInConfig()
	var conf Config
	v.Unmarshal(&conf)
	fmt.Println(conf)
}

```

