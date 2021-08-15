# 使用channel模拟实现锁

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type Mutex struct {
	ch chan struct{}
}

func NewMutex() *Mutex {
	return &Mutex{ch: make(chan struct{}, 1)}
}
func (m *Mutex) Lock() {
	m.ch <- struct{}{}
}

func (m *Mutex) Unlock() {
	select {
	case <-m.ch:
	default:
		panic("unlock error")
	}
}

type Job struct {
	Count int
	lock *Mutex
}

func (j *Job) Incr() {
	j.lock.Lock()
	defer j.lock.Unlock()
	j.Count++
}

func (j *Job) Decr() {
	j.lock.Lock()
	defer j.lock.Unlock()
	j.Count--
}

func makeChannel() {
	job := &Job{lock: NewMutex()}
	wg := sync.WaitGroup{}
	wg.Add(2)
	go func() {
		defer wg.Done()
		for i := 0; i < 10000; i++ {
			job.Incr()
		}
	}()
	go func() {
		defer wg.Done()
		for i := 0; i < 10000; i++ {
			job.Decr()
		}
	}()
	wg.Wait()
	fmt.Println(job.Count)
}

func main() {
	for i := 0; i < 100; i++ {
		makeChannel()
		time.Sleep(time.Second)
	}
}

```

