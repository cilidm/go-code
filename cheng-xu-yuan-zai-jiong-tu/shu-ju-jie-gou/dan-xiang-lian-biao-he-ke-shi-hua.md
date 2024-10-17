# 单向链表和可视化

```go
package linkedlist

import "fmt"

type SinglyNode struct {
	Value string
	Next  *SinglyNode
}

func (s *SinglyNode) String() string {
	return fmt.Sprintf("value:%s", s.Value)
}

type SinglyList struct {
	Head *SinglyNode
}

func NewSinglyList() *SinglyList {
	return &SinglyList{}
}

func (s *SinglyList) IsEmpty() bool {
	return s.Head == nil
}

func (s *SinglyList) Append(v string) {
	newNode := &SinglyNode{Value: v}
	if s.IsEmpty() {
		s.Head = newNode
	} else {
		cur := s.Head
		for cur.Next != nil { // 代表当前节点一直有值
			cur = cur.Next
		}
		cur.Next = newNode
	}
}

```

```go
package linkedlist

import (
	"fmt"
	"testing"
)

func TestSinglyList_Append(t *testing.T) {
	list := NewSinglyList()
	list.Append("Node1")
	list.Append("Node2")
	fmt.Println(list.Head)

	cur := list.Head
	for cur != nil{
		fmt.Println(cur)
		cur = cur.Next
	}
}
```

