# 工厂方法模式

```go
package factory

import "design/desp/models"

// 工厂方法模式

const (
	ProductTechBook = iota
	ProductDailyBriefs
)

type ProductType int

type IProductFactory interface {
	CreateProduct(t ProductType) IProduct
}

type IProduct interface {
	GetInfo() string
}

type TechFactory struct{}

func (*TechFactory) CreateProduct(t ProductType) IProduct {
	switch t {
	case ProductTechBook:
		return &models.Book{}
	}
	return nil
}

type DailyFactory struct{}

func (*DailyFactory) CreateProduct(t ProductType) IProduct {
	switch t {
	case ProductDailyBriefs:
		return &models.Briefs{}
	}
	return nil
}

```

```go
package models

type Book struct {
	Id       int
	BookName string
}

func (this *Book) GetInfo() string {
	return "book"
}

type Briefs struct {
	Id   int
	Size string
}

func(this * Briefs) GetInfo() string {
    return "内裤"
}
```

```go
package factory

import (
	"fmt"
	"testing"
)

func TestTechBook(t *testing.T) {
	info := new(TechFactory).CreateProduct(ProductTechBook).GetInfo()
	fmt.Println(info)
}

func TestDailyFactory(t *testing.T) {
	info := new(DailyFactory).CreateProduct(ProductDailyBriefs).GetInfo()
	fmt.Println(info)
}

```

