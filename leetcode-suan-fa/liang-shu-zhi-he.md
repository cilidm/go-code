# 两数之和

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target 的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

示例 1：

`输入：nums = [2,7,11,15], target = 9 输出：[0,1] 解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。` 

示例 2：

`输入：nums = [3,2,4], target = 6 输出：[1,2]`

 示例 3：

`输入：nums = [3,3], target = 6 输出：[0,1]`

```go
// 暴力解法
func TwoSum(nums []int, target int) []int {
	for i, x := range nums {
		for j := i + 1; j < len(nums); j++ {
			if x+nums[j] == target {
				return []int{i, j}
			}
		}
	}
	return nil
}

// 哈希解法
func TwoSumHash(nums []int, target int) []int {
	hashTable := map[int]int{}
	for i, x := range nums {
		if p, ok := hashTable[target-x]; ok {
			return []int{p, i}
		}
		hashTable[x] = i
	}
	return nil
}


func TestTwoSum(t *testing.T) {
	//fmt.Println(TwoSum([]int{11,1,92,2,7,15},9))
	fmt.Println(TwoSumHash([]int{11,1,92,2,7,15},9))
}
```



