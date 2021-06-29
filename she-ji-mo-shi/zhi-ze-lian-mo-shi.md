# 职责链模式

```go
package chain

import "fmt"

type Manager interface {
	HaveRight(money int) bool // 判断是否有权限 在此栏目执行
	HandleFeeRequest(name string, money int) bool
}

type RequestChain struct {
	Manager
	successor *RequestChain // 接班人
}

func (r *RequestChain) SetRight(m *RequestChain) {
	r.successor = m
}

func (r *RequestChain) HandleFeeRequest(name string, money int) bool {
	if r.Manager.HaveRight(money) {	// 有权限在此栏目执行
		return r.Manager.HandleFeeRequest(name, money)
	}
	if r.successor != nil {	// 无权限 先判断是否有下层链级 如有则跳至下层链级执行
		return r.successor.HandleFeeRequest(name, money)
	}
	return false
}

func (r *RequestChain) HaveRight(money int) bool {
	return true
}

type ProjectManager struct{}

func (p ProjectManager) HaveRight(money int) bool {
	return money < 500
}

func (p ProjectManager) HandleFeeRequest(name string, money int) bool {
	if name == "bob" {
		fmt.Printf("Project manager permit %s %d fee request\n", name, money)
		return true
	}
	fmt.Printf("Project manager don't permit %s %d fee request\n", name, money)
	return false
}

func NewProjectManagerChain() *RequestChain {
	return &RequestChain{
		Manager: &ProjectManager{},
	}
}

type DepManager struct{}

func (d DepManager) HaveRight(money int) bool {
	return money < 5000
}

func (d DepManager) HandleFeeRequest(name string, money int) bool {
	if name == "tom" {
		fmt.Printf("Dep manager permit %s %d fee request\n", name, money)
		return true
	}
	fmt.Printf("Dep manager don't permit %s %d fee request\n", name, money)
	return false
}

func NewDepManagerChain() *RequestChain {
	return &RequestChain{
		Manager: &DepManager{},
	}
}

type GeneralManager struct{}

func NewGeneralManagerChain() *RequestChain {
	return &RequestChain{
		Manager: &GeneralManager{},
	}
}

func (*GeneralManager) HaveRight(money int) bool {
	return true
}

func (*GeneralManager) HandleFeeRequest(name string, money int) bool {
	if name == "ada" {
		fmt.Printf("General manager permit %s %d fee request\n", name, money)
		return true
	}
	fmt.Printf("General manager don't permit %s %d fee request\n", name, money)
	return false
}

```

```go
package chain

import "testing"

func TestChain(t *testing.T){
	c1 := NewProjectManagerChain()
	c2 := NewDepManagerChain()
	c3 := NewGeneralManagerChain()
	c1.SetRight(c2)
	c2.SetRight(c3)
	var c Manager = c1
	c.HandleFeeRequest("bob",500)
	c.HandleFeeRequest("tom",1400)
	c.HandleFeeRequest("ada",10000)
	c.HandleFeeRequest("floar",400)
}

```

