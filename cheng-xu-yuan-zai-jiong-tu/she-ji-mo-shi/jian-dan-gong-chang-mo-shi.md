# 简单工厂模式

```go
package factory

type UserCreateFunc func(id int, name string) interface{}

type User struct {
	ID   int
	Name string
}

func NewUser() UserCreateFunc {
	return func(id int, name string) interface{} {
		return &User{ID: id, Name: name}
	}
}

type AdminUser struct {
	ID   int
	Name string
	Role string
}

func NewAdminUser() UserCreateFunc {
	return func(id int, name string) interface{} {
		return &AdminUser{ID: id, Name: name,Role: "admin"}
	}
}

```

```go
package factory

import (
	"fmt"
	"testing"
)

const (
	FrontUserType = iota
	AdminUserType
)
type UserType int

func CreateUser(t UserType) UserCreateFunc{
	switch t {
	case FrontUserType:
		return NewUser()
	case AdminUserType:
		return NewAdminUser()
	default:
		return NewUser()
	}
}

func TestFactoryNew(t *testing.T){
	user := CreateUser(FrontUserType)(123,"abc").(*User)
	fmt.Println(user)
	admin := CreateUser(AdminUserType)(124,"admin").(*AdminUser)
	fmt.Println(admin)
}

```

