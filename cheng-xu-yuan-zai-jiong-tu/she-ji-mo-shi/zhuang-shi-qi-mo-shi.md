# 装饰器模式

```go
package decorator

// 装饰器模式
type User struct {
	Id   int
	Name string
}

func GetInfo(id int) *User {
	return &User{Id: id, Name: "this is user"}
}

type UserInfoFunc func(id int) *User

func GetInfoByRole(fn UserInfoFunc) UserInfoFunc {
	return func(id int) *User {
		user := fn(id)
		user.Name = "guest-" + user.Name
		return user
	}
}

```

```go
package decorator

import (
	"fmt"
	"testing"
)

func TestGetInfo(t *testing.T) {
	user := GetInfoByRole(GetInfo)(123)
	fmt.Println(user)
}

```

