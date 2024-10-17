# 建造者模式

```go
package models

type Book struct {
	Id       int    // required
	BookName string // required
	Price    float64
}

func (this *Book) Builder(id int, name string) *BookBuilder {
	return NewBookBuilder(id, name)
}

func (this *Book) GetInfo() string {
	return "book"
}

type Briefs struct {
	Id   int
	Size string
}

func (this *Briefs) GetInfo() string {
	return "内裤"
}

```

```go
package models

// 建造者模式
type BookBuilder struct {
	id       int    // required
	bookName string // required
	price    float64
}

func (this *BookBuilder) Build() *Book {
	book := &Book{
		Id:       this.id,
		BookName: this.bookName,
	}
	if this.price > 0 { // 可加入其他判断
		book.Price = this.price
	}
	return book
}

func (this *BookBuilder) SetPrice(price float64) *BookBuilder {
	this.price = price
	return this
}

func NewBookBuilder(id int, bookName string) *BookBuilder {
	return &BookBuilder{id: id, bookName: bookName}
}

```

```go
package models

import (
	"fmt"
	"testing"
)

func TestBookBuilder(t *testing.T) {
	book := new(Book).Builder(100, "Go圣经").SetPrice(20.5).Build()
	fmt.Println(book)
}

```

