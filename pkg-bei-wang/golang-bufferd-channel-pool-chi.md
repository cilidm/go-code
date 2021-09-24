# Golang - bufferd channel - pool 池

```go
package main

// ---------------Worker---------------

type Worker struct {
    IsAvailable bool
}

// 创建
func InitWorker() *Worker {
    return &Worker{}
}

// 执行
func (w *Worker)Process(){

}

// 销毁worker
func (w *Worker)Destory(){
    // 消除之前占用的某一些资源
}

// --------------Pool---------------

type Pool struct {
    availableWorker chan *Worker
}

// 创建Pool
func NewPool(maxsize int)Pool{
    return Pool{
        availableWorker:make(chan *Worker,maxsize), // 限制 bufferd chan 最大缓存数
    }
}

// 获取worker
func (p *Pool) get()(w *Worker){
    select{
    case w = <-p.availableWorker: // 有可用的worker
    default:
        // 无可用的worker,创建（实在没船就创造船）
        w = InitWorker()
    }
    return
}

// 回收worker
func (p *Pool) returnBack(w *Worker){
    // worker 是否还可用（飞船是否还可用）
    if !w.IsAvailable {
        // 不可用，销毁丢弃
        w.Destory()
        return
    }
    select {
    case p.availableWorker<-w: // 如果有位置，则缓存（停飞船喽）
    default:
        // 没位置，销毁丢弃
        w.Destory()
    }
}

// 销毁所有worker(不要重复昭君的惨剧)
func (p *Pool) destroyAllWorker(){
    for {
        select{
        case w:= <-p.availableWorker:
            w.Destory()
        default:
            return
        }
    }
}

// -------------- Let's go -----------------

func main() {
    pool := NewPool(50) // 最大缓存50个worker（50个飞船的停飞场）
    defer pool.destroyAllWorker() // 如果不想被追到街上打的话，最后记得销毁所有的工作者

    for {
        worker := pool.get() // 获取worker
        worker.Process() // 使用worker
        pool.returnBack(worker) // 归还worker
    }

    // 友情提示：该代码接近伪码，不知直接运行
}
```

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type Pool struct {
	queue chan int
	wg    *sync.WaitGroup
}

func NewPool(size int) *Pool {
	if size <= 0 {
		size = 1
	}
	return &Pool{queue: make(chan int, size), wg: &sync.WaitGroup{}}
}

func (this *Pool) Add(size int) {
	for i := 0; i < size; i++ { // size > 0
		this.queue <- 1
	}
	for i := 0; i > size; i-- { // size < 0
		<-this.queue
	}
	this.wg.Add(size)
}

func (this *Pool) Done() {
	<-this.queue
	this.wg.Done()
}

func (this *Pool) Wait() {
	this.wg.Wait()
}

func main() {
	pool := NewPool(15)
	for i := 1; i <= 500; i++ {
		pool.Add(1)
		go func(i int) {
			defer pool.Done()
			fmt.Println(i)
			time.Sleep(time.Second)
		}(i)
	}
	pool.Wait()
}

```

