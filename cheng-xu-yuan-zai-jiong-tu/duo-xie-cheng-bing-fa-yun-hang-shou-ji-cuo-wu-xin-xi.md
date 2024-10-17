# 多协程并发运行收集错误信息

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

func job(id int) (string, error) {
	rand.Seed(time.Now().UnixNano())
	time.Sleep(time.Millisecond)
	randNum := rand.Intn(1000)
	if randNum%2 == 0 {
		return "", fmt.Errorf("发生错误,ID=%d,RandNum=%d", id, randNum)
	}
	return fmt.Sprintf("成功结果,ID=%d", id), nil
}

func printErrChan() {
	errChan := make(chan error)
	wg := sync.WaitGroup{}
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(index int) {
			defer wg.Done()
			_, err := job(index)
			if err != nil {
				errChan <- err
			}
		}(i)
	}
	go func() {
		defer close(errChan)
		wg.Wait()
	}()
	count := 0
	for item := range errChan {
		fmt.Println(item)
		count++
		if count == 2 {
			break
		}
	}
}

func printAllChan() {
	retChan := make(chan interface{})
	wg := sync.WaitGroup{}
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(index int) {
			defer wg.Done()
			ret, err := job(index)
			if err != nil {
				retChan <- err
			} else {
				retChan <- ret
			}
		}(i)
	}
	go func() {
		defer close(retChan)
		wg.Wait()
	}()
	count := 0
	for item := range retChan {
		if err,ok := item.(error);ok{
			fmt.Println("ErrorInfo:",err.Error())
			count++
		}else{
			fmt.Println("SuccessInfo:",item)
		}
		fmt.Println(item)
		if count == 2 {
			break
		}
	}
}

func main() {
	printAllChan()
}

```

