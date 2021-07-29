# 解析json到MAP

```go
package main

import (
	"encoding/json"
	"errors"
	"fmt"
)

var dataMap map[string]interface{}

func initConfig() error {
	data := `{
	  "id": 1,
	  "name": "Alejandra Mcdaniel",
	  "age": 12
	}`
	dataMap = make(map[string]interface{})
	err := json.Unmarshal([]byte(data), &dataMap)
	if err != nil {
		return err
	}
	return nil
}

func getConfig(key string) (interface{}, error) {
	val, ok := dataMap[key]
	if !ok {
		return nil, errors.New("not found key")
	}
	return val, nil
}

func main() {
	initConfig()
	val,err := getConfig("age")
	if err != nil{
		return
	}
	switch val.(type) {
	case float64:
		fmt.Println("float64",val.(float64))
	case string:
		fmt.Println("string",val)
	}
}

```

