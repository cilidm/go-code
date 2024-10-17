# viper写入配置文件

```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

type Config struct {
	DB    DBConf
	Redis RedisConf
	Zap   ZapConf
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

type ZapConf struct {
	Level         string `json:"level" yaml:"level"`
	Format        string ` json:"format" yaml:"format"`
	Prefix        string ` json:"prefix" yaml:"prefix"`
	Director      string ` json:"director"  yaml:"director"`
	LinkName      string ` json:"linkName" yaml:"link-name"`
	ShowLine      bool   ` json:"showLine" yaml:"showLine"`
	EncodeLevel   string ` json:"encodeLevel" yaml:"encode-level"`
	StacktraceKey string `json:"stacktraceKey" yaml:"stacktrace-key"`
	LogInConsole  bool   `json:"logInConsole" yaml:"log-in-console"`
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

	v.Set("Zap.Level", "info")
	v.Set("Zap.Format", "console")
	v.Set("Zap.Prefix", "[base-spider]")
	v.Set("Zap.Director", "runtime/log")
	v.Set("Zap.LinkName", "latest_log")
	v.Set("Zap.ShowLine", true)
	v.Set("Zap.EncodeLevel", "LowercaseColorLevelEncoder")
	v.Set("Zap.StacktraceKey", "stacktrace")
	v.Set("Zap.LogInConsole", true)

	v.WriteConfigAs("config.yaml")
	//v.SafeWriteConfigAs("config.yaml")

	v.ReadInConfig()
	var conf Config
	v.Unmarshal(&conf)
	fmt.Println(conf)
}

```

通过反射写入配置：

```go
  v := viper.New()
	//设置读取的配置文件
	v.AddConfigPath("./")
	v.SetConfigFile("config.yaml")

	var conf Config
	conf.DB.Host = "127.0.0.1"
	conf.DB.Type = "mysql"
	conf.DB.User = "root"
	conf.DB.Pwd = "123456"
	conf.DB.Name = "base-system"
	conf.DB.Host = "127.0.0.1"
	conf.DB.Port = "3306"

	t := reflect.TypeOf(conf)
	if t.Kind() != reflect.Struct {
		fmt.Println("error not struct")
		return
	}

	val := reflect.ValueOf(conf)
	for i := 0; i < t.NumField(); i++ {
		fmt.Println(t.Field(i).Tag.Get("json"),t.Field(i).Name,val.Field(i))
		v.Set(t.Field(i).Tag.Get("json"),val.Field(i).Interface())
	}
	v.WriteConfigAs("config.yaml")
```

