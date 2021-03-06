# 反转字符串

### 01、题目分析

#### 第344题：反转字符串

编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 char\[\] 的形式给出。 不要给另外的数组分配额外的空间，你必须原地修改输入数组、使用 O\(1\) 的额外空间解决这一问题。

**示例 1：**

> 输入：\["h","e","l","l","o"\] 输出：\["o","l","l","e","h"\]

示例 2：

> 输入：\["H","a","n","n","a","h"\] 输出：\["h","a","n","n","a","H"\]

### 02、题目图解

这是一道相当简单的经典题目，直接上题解：使用双指针进行反转字符串。

假设输入字符串为`["h","e","l","l","0"]`

* 定义left和right分别指向首元素和尾元素
* 当`left < right` ，进行交换。
* 交换完毕，`left++，right--`
* 直至`left == right`

具体过程如下图所示：



![](https://github.com/cilicode/interview-go/raw/master/images/1.81971505.jpg)



### 03、Go语言示例

根据以上分析，我们可以得到下面的题解：

```go
func Reverse(s []byte) {
    right := len(s) - 1
    left := 0

    for left < right {
        s[left], s[right] = s[right], s[left]
        left++
        right--
    }
}
```

### 原文

> [反转字符串\(301\)](https://www.geekxh.com/1.3.%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%B3%BB%E5%88%97/301.html#_02%E3%80%81%E9%A2%98%E7%9B%AE%E5%9B%BE%E8%A7%A3)

