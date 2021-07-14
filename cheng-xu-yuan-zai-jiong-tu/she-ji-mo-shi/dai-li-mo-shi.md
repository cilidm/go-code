# 代理模式

```go
package proxy

import "fmt"

// 代里模式和装饰器模式

type UserService struct{}

func (s *UserService) Login(name, pwd string) {
	fmt.Println("登录成功")
}

type UserProxy struct {
	svc *UserService
}

func NewUserProxy(svc *UserService) *UserProxy {
	return &UserProxy{svc: svc}
}

func (s *UserProxy) Login(decorator LogDecorator) LoginFunc {
	return decorator(s.svc.Login)
}

type LoginFunc func(name, pwd string)
type LogDecorator func(f LoginFunc) LoginFunc

func LogToRedis(f LoginFunc) LoginFunc {
	return func(name, pwd string) {
		fmt.Println("记录日志到redis")
		f(name, pwd)
	}
}

func LogToMysql(f LoginFunc) LoginFunc {
	return func(name, pwd string) {
		fmt.Println("记录日志到Mysql")
		f(name, pwd)
	}
}

```

```go
package proxy

import "testing"

func TestProxy(t *testing.T) {
	user := new(UserService)
	pro := NewUserProxy(user)
	pro.Login(LogToMysql)("abc", "123")
	pro.Login(LogToRedis)("abc", "123")
}

```

