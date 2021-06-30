# 后洗牌算法

```go
package algo

import "math/rand"

/**
后洗牌算法:
先随机生成红包序列，再洗牌打乱。

count:  红包总数量
amount: 红包总金额，分

使用int64，单位“分”
 */

func AfterShuffle(count, amount int64) ([]int64, error) {
	moneySlice := make([]int64, 0)

	// 计算最大可调度金额
	max := amount - count * MIN_MONEY

	// 随机生成初级红包序列
	for i := int64(0); i < count; i++ {
		money, err := SimpleRand(count - i, max)
		if err != nil {
			panic(err.Error())
		}
		max = max - money
		moneySlice = append(moneySlice, money + MIN_MONEY)
	}
	// 洗牌打乱
	rand.Shuffle(len(moneySlice), func(i, j int) {
		moneySlice[i], moneySlice[j] = moneySlice[j], moneySlice[i]
	})

	return moneySlice, nil
}
```

```go
func main() {
	rand.Seed(time.Now().UnixNano())
	fmt.Println(algo.AfterShuffle(int64(10), int64(100) * 100))
}
```

