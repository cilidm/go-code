# 只出现一次的数字

给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

说明：

你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

示例 1:

输入: \[2,2,1\] 输出: 1 示例 2:

输入: \[4,1,2,1,2\] 输出: 4

作者：力扣 \(LeetCode\) 链接：[https://leetcode-cn.com/leetbook/read/top-interview-questions/xm0u83/](https://leetcode-cn.com/leetbook/read/top-interview-questions/xm0u83/) 

```go
func SingleNumber(nums []int) int {
	for i := 1; i < len(nums); i++ {
		nums[0] ^= nums[i]
	}
	return nums[0]
}

func TestSingleNumber(t *testing.T) {
	fmt.Println(SingleNumber([]int{3, 2, 3, 4, 5, 6, 4, 5, 6}))
}
```

