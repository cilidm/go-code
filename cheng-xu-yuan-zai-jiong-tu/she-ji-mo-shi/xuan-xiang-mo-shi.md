# 选项模式

```go
package options

// 选项模式
type HttpClient struct {
	Timeout     int
	MaxIdle     int
	ErrCallBack func(error)
}

type ClientOption func(client *HttpClient)
type ClientOptions []ClientOption

func (this ClientOptions) apply(c *HttpClient) {
	for _, opt := range this {
		opt(c)
	}
}
func NewHttpClient(opts ...ClientOption) *HttpClient {
	c := &HttpClient{}
	ClientOptions(opts).apply(c)
	return c
}

func (c *HttpClient) Do(url string) { println("不要纠结这里") }

func WithTimeout(timeout int) ClientOption {
	return func(client *HttpClient) {
		client.Timeout = timeout
	}
}

func WithMaxIdle(maxIdle int) ClientOption {
	return func(client *HttpClient) {
		client.MaxIdle = maxIdle
	}
}

func WithErrCallBack(callback func(error)) ClientOption {
	return func(client *HttpClient) {
		client.ErrCallBack = callback
	}
}

```

```go
package options

import (
	"fmt"
	"testing"
)

func TestHttpClient_Do(t *testing.T) {
	c := NewHttpClient(
		WithMaxIdle(10),
		WithTimeout(10),
		WithErrCallBack(func(err error) {
			panic(err.Error())
		}),
	)
	fmt.Println(c)
}

```

