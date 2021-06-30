# 简单随机算法

```go
package algo

import (
	"errors"
	"math/rand"
)

// 1分
const MIN_MONEY = int64(1)

/**
简单随机算法：
每个红包最小为1分钱，于是先每个红包塞1分钱，剩下的再随机分配出去。

count:  红包总数量
amount: 红包总金额，分

使用int64，单位“分”
 */
func SimpleRand(count, amount int64) (int64, error) {
	if count == 1 {
		return amount, nil
	}

	// 最大可调度金额
	max := amount - count * MIN_MONEY
	if max == 0 {
		return 0, nil
	}
	if max < 0 {
		return 0, errors.New("红包金额与个数不匹配")
	}

	ret := rand.Int63n(max) + MIN_MONEY
	return ret, nil
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
		money, err := algo.SimpleRand(count - i, amount)
		if err != nil {
			panic(err.Error())
		}
		amount = amount - money
		fmt.Println(money)
	}
}

```

