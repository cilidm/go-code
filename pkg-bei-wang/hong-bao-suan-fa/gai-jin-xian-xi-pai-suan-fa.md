# 改进先洗牌算法

```go
package algo

import (
	"errors"
	"math/rand"
)

/**
改进先洗牌算法
 */

func DoubleRandom(count, amount int64) (int64, error) {
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
	// 一次随机，计算出一个种子金额作为随机基数
	seed := rand.Int63n(count * 2) + 1 // 防止出现0
	n := max / seed + MIN_MONEY
	// 二次随机，计算出红包金额序列元素
	x := rand.Int63n(n)
	return x + MIN_MONEY, nil
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
		money, err := algo.DoubleRandom(count - i, amount)
		if err != nil {
			panic(err.Error())
		}
		amount = amount - money
		fmt.Println(money)
	}
}

```

