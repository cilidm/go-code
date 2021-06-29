# 创建者模式

将一个复杂对象的构建分离成多个简单对象的构建组合

```go
package build

type Builder interface {
	Part1()
	Part2()
	Part3()
}

type Director struct {
    builder Builder
}

func NewDirector(builder Builder) *Director {
    return &Director {builder:builder}
}

func(d * Director) Construct() {
	d.builder.Part1()
	d.builder.Part2()
	d.builder.Part3()
}

type Build1 struct {
	result string
}

func(b * Build1) Part1() {
    b.result += "1"
}

func(b * Build1) Part2() {
	b.result += "2"
}

func(b * Build1) Part3() {
	b.result += "3"
}

func(b * Build1) GetResult() string{
	return b.result
}

type Builder2 struct {
	result int
}

func (b *Builder2) Part1() {
	b.result += 1
}

func (b *Builder2) Part2() {
	b.result += 2
}

func (b *Builder2) Part3() {
	b.result += 3
}

func (b *Builder2) GetResult() int {
	return b.result
}
```

```go
package build

import (
	"fmt"
	"testing"
)

func TestBuild1(t *testing.T){
	build := &Build1{}
	director := NewDirector(build)
	director.Construct()
	res := build.GetResult()
	fmt.Println(res)
	if res != "123"{
		t.Fatalf("Builder1 fail expect 123 acture %s", res)
	}
}

func TestBuilder2(t *testing.T) {
	builder := &Builder2{}
	director := NewDirector(builder)
	director.Construct()
	res := builder.GetResult()
	fmt.Println(res)
	if res != 6 {
		t.Fatalf("Builder2 fail expect 6 acture %d", res)
	}
}
```

