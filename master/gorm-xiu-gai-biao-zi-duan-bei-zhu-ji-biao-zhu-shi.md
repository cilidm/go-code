# gorm修改表字段备注,及表注释

**模型表设置**

```go
package models
 
import (
	"github.com/jinzhu/gorm"
)
 
type Book struct {
	gorm.Model
	Name   string `json:"name" gorm:"type:varchar(20) not null comment '书名';"`
	Count  string `json:"count" gorm:"type:varchar(10) not null comment '价格';"`
	Author string `json:"author" gorm:"type:varchar(20) not null comment '作者';"`
	Type   string `json:"type" gorm:"type:varchar(20) not null default 1 comment '类型';"`
}
```

**表迁移初始化设置**

```go
package datasource
 
import (
	"demo/conf"
	"demo/models"
	_ "github.com/go-sql-driver/mysql"
	"github.com/jinzhu/gorm"
	"strings"
	"time"
)
 
var db *gorm.DB
 
func GetDB() *gorm.DB {
	return db
}
 
func init() {
	path := strings.Join([]string{conf.Sysconfig.DBUserName, ":", conf.Sysconfig.DBPassword, "@(", conf.Sysconfig.DBIp, ":", conf.Sysconfig.DBPort, ")/", conf.Sysconfig.DBName, "?charset=utf8&parseTime=true"}, "")
	var err error
	db, err = gorm.Open("mysql", path)
	if err != nil {
		panic(err)
	}
	db.SingularTable(true)
	db.DB().SetConnMaxLifetime(1 * time.Second)
	db.DB().SetMaxIdleConns(20)   //最大打开的连接数
	db.DB().SetMaxOpenConns(2000) //设置最大闲置个数
	db.SingularTable(true)        //表生成结尾不带s
	// 启用Logger，显示详细日志
	db.LogMode(true)
	if !db.HasTable(&models.Book{}) { //db.Set 设置一些额外的表属性                              //db.CreateTable创建表
		if err := db.Set("gorm:table_options", "ENGINE=InnoDB DEFAULT CHARSET=utf8 comment '图书表'").CreateTable(&models.Book{}).Error; err != nil {
			panic(err)
		}
	}
}
```
