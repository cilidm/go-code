# 先洗牌算法

```go
package algo

import (
	"errors"
	"math/rand"
)

/**
先洗牌算法：
先随机出基数，在通过这个基数生成随机数。

count:  红包总数量
amount: 红包总金额，分

使用int64，单位“分”
 */

func BeforeShuffle(count, amount int64) (int64, error) {
	if count == 1 {
		return amount, nil
	}
	// 计算最大可调度金额
	max := amount - count * MIN_MONEY
	if max == 0 {
		return 0, nil
	}
	if max < 0 {
		return 0, errors.New("红包金额与个数不匹配")
	}
	// 计算随机基数的序列
	seeds := make([]int64, 0)
	// 随机基数序列的长度：3 ~ count * 1/2
	size := count / 2
	if size < 3 {
		size = 3
	}
	for i := int64(0); i < size; i++ {
		x := max / (i + 1)
		seeds = append(seeds, x)
	}
	// 取出随机基数
	index := rand.Int63n(int64(len(seeds)))
	// 使用随机基数生成随机数
	y := rand.Int63n(seeds[index])

	return y + MIN_MONEY, nil
}
```

```go
package main

import (
	"bonus/infra/algo"
	"fmt"
	"math/rand"
	"time"
)

func main() {
	count, amount := int64(10), int64(100) * 100
	rand.Seed(time.Now().UnixNano())

	for i := int64(0); i < count; i++ {
		money, err := algo.BeforeShuffle(count - i, amount)
		if err != nil {
			panic(err.Error())
		}

		amount -= money

		fmt.Println(money)
	}
}
```

