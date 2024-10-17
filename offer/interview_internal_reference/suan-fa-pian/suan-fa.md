# 算法

1.**问题：如何实现一个高效的单向链表逆序输出？**

**出题人：阿里巴巴出题专家：昀龙／阿里云弹性人工智能负责人**

**参考答案：下面是其中一种写法，也可以有不同的写法，比如递归等。供参考。**

```
typedef struct node{
    int           data;
    struct node*  next;
    node(int d):data(d), next(NULL){}
}node;

void reverse(node* head)
{
    if(head == NULL){
        return;
    }

    node* pleft = NULL;
    node* pcurrent = head;
    node* pright = head->next;

    while(pright){
        pcurrent->next = pleft;
        node *ptemp = pright->next;
        pright->next = pcurrent;
        pleft = pcurrent;
        pcurrent = pright;
        pright = ptemp;
    }

    while(pcurrent != NULL){
        cout<< pcurrent->data << "\t";
        pcurrent = pcurrent->next;
    }
}
```

```java
class Solution<T> {

    public void reverse(ListNode<T> head) {
       if (head == null || head.next == null) {
           return ;
       }
       ListNode<T> currentNode = head;
       Stack<ListNode<T>> stack = new Stack<>();
       while (currentNode != null) {
           stack.push(currentNode);
           ListNode<T> tempNode = currentNode.next;
           currentNode.next = null; // 断开连接
           currentNode = tempNode;
       }

       head = stack.pop();
       currentNode = head;

       while (!stack.isEmpty()) {
           currentNode.next = stack.pop();
           currentNode = currentNode.next;
       }
    }
}

class ListNode<T>{
    T val;
    public ListNode(T val) {
        this.val = val;
    }
    ListNode<T> next;
}
```

****

**2.题目：已知 sqrt (2)约等于 1.414，要求不用数学库，求 sqrt (2)精确到小数点后 10 位。**

**出题人：——阿里巴巴出题专家：文景／阿里云 CDN 资深技术专家**

**参考答案：**

**\* 考察点**

1. 基础算法的灵活应用能力（二分法学过数据结构的同学都知道，但不一定往这个方向考虑；如果学过数值计算的同学，应该还要能想到牛顿迭代法并解释清楚）
2. 退出条件设计

**二分法**

**1. 已知 sqrt(2)约等于 1.414，那么就可以在(1.4, 1.5)区间做二分**

查找，如： a) high=>1.5 b) low=>1.4 c) mid => (high+low)/2=1.45 d) 1.45\*1.45>2 ? high=>1.45 : low => 1.45 e) 循环到 c)

**2. 退出条件**

a) 前后两次的差值的绝对值<=0.0000000001, 则可退出

```
const double EPSILON = 0.0000000001;

double sqrt2() {
    double low = 1.4, high = 1.5;
    double mid = (low + high) / 2;

    while (high - low > EPSILON) {
        if (mid * mid > 2) {
            high = mid;
        } else {
            low = mid;
        }
        mid = (high + low) / 2;
    }

    return mid;
}
```

**牛顿迭代法**

**1.牛顿迭代法的公式为：**

xn+1 = xn-f(xn)/f'(xn)

对于本题，需要求解的问题为：f(x)=x2-2 的零点

```
EPSILON = 0.1 ** 10
def newton(x):
    if abs(x ** 2 - 2) > EPSILON:
        return newton(x - (x ** 2 - 2) / (2 * x))
    else:
        return x
```

**3.题目：给定一个二叉搜索树(BST)，找到树中第 K 小的节点。**

**出题人：阿里巴巴出题专家：文景／阿里云 CDN 资深技术专家**

**参考答案：**

**\* 考察点**

1. 基础数据结构的理解和编码能力
2. 递归使用

**\* 示例**

```
       5
      / \
     3   6
    / \
   2   4
  /
 1
```

说明：保证输入的 K 满足 1<=K<=(节点数目）

解法1：树相关的题目，第一眼就想到递归求解，左右子树分别遍历。联想到二叉搜索树的性质，root 大于左子树，小于右子树，如果左子树的节点数目等于 K-1，那么 root 就是结果，否则如果左子树节点数目小于 K-1，那么结果必然在右子树，否则就在左子树。因此在搜索的时候同时返回节点数目，跟 K 做对比，就能得出结果了。

```
/**
 * Definition for a binary tree node.
 **/

public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}

class Solution {
    private class ResultType {

        boolean found;  // 是否找到

        int val;  // 节点数目
        ResultType(boolean found, int val) {
            this.found = found;
            this.val = val;
        }
    }

    public int kthSmallest(TreeNode root, int k) {
        return kthSmallestHelper(root, k).val;
    }

    private ResultType kthSmallestHelper(TreeNode root, int k) {
        if (root == null) {
            return new ResultType(false, 0);
        }

        ResultType left = kthSmallestHelper(root.left, k);

        // 左子树找到，直接返回
        if (left.found) {
            return new ResultType(true, left.val);
        }

        // 左子树的节点数目 = K-1，结果为 root 的值
        if (k - left.val == 1) {
            return new ResultType(true, root.val);
        }

        // 右子树寻找
        ResultType right = kthSmallestHelper(root.right, k - left.val - 1);
        if (right.found) {
            return new ResultType(true, right.val);
        }

        // 没找到，返回节点总数
        return new ResultType(false, left.val + 1 + right.val);
    }
}
```

解法2：基于二叉搜索树的特性，在中序遍历的结果中，第k个元素就是本题的解。 最差的情况是k节点是bst的最右叶子节点，不过`每个节点的遍历次数最多是1次`。 遍历并不是需要全部做完，使用计数的方式，找到第k个元素就可以退出。 下面是go的一个简单实现。

```
// BST is binary search tree
type BST struct {
    key, value  int
    left, right *BST
}

func (bst *BST) setLeft(b *BST) {
    bst.left = b
}

func (bst *BST) setRight(b *BST) {
    bst.right = b
}

// count 查找bst第k个节点的值，未找到就返回0
func count(bst *BST, k int) int {
    if k < 1 {
        return 0
    }

    c := 0
    ok, value := countRecursive(bst, &c, k)

    if ok {
        return value
    }

    return 0
}

// countRecurisive 对bst使用中序遍历
// 用计数方式控制退出遍历，参数c就是已遍历节点数
func countRecursive(bst *BST, c *int, k int) (bool, int) {
    if bst.left != nil {
        ok, value := countRecursive(bst.left, c, k)
        if ok {
            return ok, value
        }
    }

    if *c == k-1 {
        return true, bst.value
    }

    *c++

    if bst.right != nil {
        ok, value := countRecursive(bst.right, c, k)
        if ok {
            return ok, value
        }
    }

    return false, 0
}

// 下面是测试代码，覆盖了退化的情况和普通bst

func createBST1() *BST {
    b1 := &BST{key: 1, value: 10}
    b2 := &BST{key: 2, value: 20}
    b3 := &BST{key: 3, value: 30}
    b4 := &BST{key: 4, value: 40}
    b5 := &BST{key: 5, value: 50}
    b6 := &BST{key: 6, value: 60}
    b7 := &BST{key: 7, value: 70}
    b8 := &BST{key: 8, value: 80}
    b9 := &BST{key: 9, value: 90}

    b9.setLeft(b8)
    b8.setLeft(b7)
    b7.setLeft(b6)
    b6.setLeft(b5)
    b5.setLeft(b4)
    b4.setLeft(b3)
    b3.setLeft(b2)
    b2.setLeft(b1)

    return b9
}

func createBST2() *BST {
    b1 := &BST{key: 1, value: 10}
    b2 := &BST{key: 2, value: 20}
    b3 := &BST{key: 3, value: 30}
    b4 := &BST{key: 4, value: 40}
    b5 := &BST{key: 5, value: 50}
    b6 := &BST{key: 6, value: 60}
    b7 := &BST{key: 7, value: 70}
    b8 := &BST{key: 8, value: 80}
    b9 := &BST{key: 9, value: 90}

    b1.setRight(b2)
    b2.setRight(b3)
    b3.setRight(b4)
    b4.setRight(b5)
    b5.setRight(b6)
    b6.setRight(b7)
    b7.setRight(b8)
    b8.setRight(b9)

    return b1
}

func createBST3() *BST {
    b1 := &BST{key: 1, value: 10}
    b2 := &BST{key: 2, value: 20}
    b3 := &BST{key: 3, value: 30}
    b4 := &BST{key: 4, value: 40}
    b5 := &BST{key: 5, value: 50}
    b6 := &BST{key: 6, value: 60}
    b7 := &BST{key: 7, value: 70}
    b8 := &BST{key: 8, value: 80}
    b9 := &BST{key: 9, value: 90}

    b5.setLeft(b3)
    b5.setRight(b7)
    b3.setLeft(b2)
    b3.setRight(b4)
    b2.setLeft(b1)
    b7.setLeft(b6)
    b7.setRight(b8)
    b8.setRight(b9)

    return b5
}

func createBST4() *BST {
    b := &BST{key: 1, value: 10}
    last := b

    for i := 2; i < 100000; i++ {
        n := &BST{key: i, value: i * 10}
        last.setRight(n)

        last = n
    }

    return b
}

func createBST5() *BST {
    b := &BST{key: 99999, value: 999990}
    last := b

    for i := 99998; i > 0; i-- {
        n := &BST{key: i, value: i * 10}
        last.setLeft(n)

        last = n
    }

    return b
}

func createBST6() *BST {
    b := &BST{key: 50000, value: 500000}
    last := b

    for i := 49999; i > 0; i-- {
        n := &BST{key: i, value: i * 10}
        last.setLeft(n)

        last = n
    }

    last = b

    for i := 50001; i < 100000; i++ {
        n := &BST{key: i, value: i * 10}
        last.setRight(n)

        last = n
    }

    return b
}

func TestK(t *testing.T) {
    bst1 := createBST1()
    bst2 := createBST2()
    bst3 := createBST3()
    bst4 := createBST4()

    check(t, bst1, 1, 10)
    check(t, bst1, 2, 20)
    check(t, bst1, 3, 30)
    check(t, bst1, 4, 40)
    check(t, bst1, 5, 50)
    check(t, bst1, 6, 60)
    check(t, bst1, 7, 70)
    check(t, bst1, 8, 80)
    check(t, bst1, 9, 90)

    check(t, bst2, 1, 10)
    check(t, bst2, 2, 20)
    check(t, bst2, 3, 30)
    check(t, bst2, 4, 40)
    check(t, bst2, 5, 50)
    check(t, bst2, 6, 60)
    check(t, bst2, 7, 70)
    check(t, bst2, 8, 80)
    check(t, bst2, 9, 90)

    check(t, bst3, 1, 10)
    check(t, bst3, 2, 20)
    check(t, bst3, 3, 30)
    check(t, bst3, 4, 40)
    check(t, bst3, 5, 50)
    check(t, bst3, 6, 60)
    check(t, bst3, 7, 70)
    check(t, bst3, 8, 80)
    check(t, bst3, 9, 90)

    check(t, bst4, 1, 10)
    check(t, bst4, 2, 20)
    check(t, bst4, 3, 30)
    check(t, bst4, 4, 40)
    check(t, bst4, 5, 50)
    check(t, bst4, 6, 60)
    check(t, bst4, 7, 70)
    check(t, bst4, 8, 80)
    check(t, bst4, 9, 90)

    check(t, bst4, 99991, 999910)
    check(t, bst4, 99992, 999920)
    check(t, bst4, 99993, 999930)
    check(t, bst4, 99994, 999940)
    check(t, bst4, 99995, 999950)
    check(t, bst4, 99996, 999960)
    check(t, bst4, 99997, 999970)
    check(t, bst4, 99998, 999980)
    check(t, bst4, 99999, 999990)
}

func check(t *testing.T, b *BST, k, value int) {
    t.Helper()

    checkCall(t, b, k, value, count)
    // 此处可添加其他解法的实现
}

func checkCall(t *testing.T, b *BST, k, value int, find func(bst *BST, kth int) int) {
    t.Helper()

    got := find(b, k)
    if got != value {
        t.Fatalf("want:%d, got:%d", value, got)
    }
}
```



**4.题目**：LRU 缓存机制 设计和实现一个 LRU（最近最少使用）缓存数据结构，使它应该支持一下操作：get 和 put。 get(key) - 如果 key 存在于缓存中，则获取 key 的 value（总是正数），否则返回 -1。 put(key,value) - 如果 key 不存在，请设置或插入 value。当缓存达到其容量时，它应该在插入新项目之前使最近最少使用的项目作废。

**出题人**：文景／阿里云 CDN 资深技术专家

**参考答案**：

python版本的：

```python
class LRUCache(object):
    def __init__(self, capacity):
    """
    :type capacity: int
    """
    self.cache = {}
    self.keys = []
    self.capacity = capacity

    def visit_key(self, key):
        if key in self.keys:
            self.keys.remove(key)
        self.keys.append(key)

    def elim_key(self):
        key = self.keys[0]
        self.keys = self.keys[1:]
        del self.cache[key]

    def get(self, key):
        """
        :type key: int
        :rtype: int
        """
        if not key in self.cache:
            return -1
        self.visit_key(key)
        return self.cache[key]

    def put(self, key, value):
        """
        :type key: int
        :type value: int
        :rtype: void
        """
        if not key in self.cache:
        if len(self.keys) == self.capacity:
        self.elim_key()
        self.cache[key] = value
        self.visit_key(key)

def main():
    s =
    [["put","put","get","put","get","put","get","get","get"],[[1,1],[2,2],[1],[3,3],[2],[
    4,4],[1],[3],[4]]]
    obj = LRUCache(2)
    l=[]
    for i,c in enumerate(s[0]):
        if(c == "get"):
            l.append(obj.get(s[1][i][0]))
        else:
            obj.put(s[1][i][0], s[1][i][1])
    print(l)

if __name__ == "__main__":
    main()
```

c++版本的：

```cpp
class LRUCache{
    public:
        LRUCache(int capacity) {
            cap = capacity;
        }

        int get(int key) {
            auto it = m.find(key);
            if (it == m.end()) return -1;
            l.splice(l.begin(), l, it->second);
            return it->second->second;
        }

        void set(int key, int value) {
            auto it = m.find(key);
            if (it != m.end()) l.erase(it->second);
            l.push_front(make_pair(key, value));
            m[key] = l.begin();
            if (m.size() > cap) {
                int k = l.rbegin()->first;
                l.pop_back();
                m.erase(k);
            }
        }
}
```



**5.题目：给定一个链表，删除链表的倒数第 N 个节点，并且返回链表的头结点。**

◼ 示例： 给定一个链表: 1->2->3->4->5, 和 n = 2. 当删除了倒数第二个节点后，链表变为 1->2->3->5. 说明： 给定的 n 保证是有效的。 要求： 只允许对链表进行一次遍历。

**出题人：阿里巴巴出题专家：屹平／阿里云视频云边缘计算高级技术专家**

**参考答案：**

我们可以使用两个指针而不是一个指针。第一个指针从列表的开头向前移动 n+1 步，而第二个指针将从列表的开头出发。现在，这两个指针被 n 个结点分开。我们通过同时移动两个指针向前来保持这个恒定的间隔，直到第一个指针到达最后一个结点。此时第二个指针将指向从最后一个结点数起的第 n 个结点。我们重新链接第二个指针所引用的结点的 next 指针指向该结点的下下个结点。

**参考代码**：

```java
public ListNode removeNthFromEnd(ListNode head, int n)
{
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode first = dummy;
    ListNode second = dummy;
    // Advances first pointer so that the gap between first
    and second is n nodes apart
    for (int i = 1; i <= n + 1; i++) {
        first = first.next;
    }
    // Move first to the end, maintaining the gap
    while (first != null) {
        first = first.next;
        second = second.next;
    }
    second.next = second.next.next;
    return dummy.next;
}
```

**复杂度分析：**

* 时间复杂度：O(L)，该算法对含有 L 个结点的列表进行了一次遍历。因此时间复杂度为 O(L)。
* 空间复杂度：O(1)，我们只用了常量级的额外空间。

**6.题目：给定一个整数数组和一个整数，返回两个数组的索引，这两个索引指向的数字的加和等于指定的整数。需要最优的算法，分析算法的空间和时间复杂度**

参考答案：

```java
public int[] twoSum(int[] nums, int target) {
    if(nums==null || nums.length<2)
        return new int[]{0,0};

    HashMap<Integer, Integer> map = new HashMap<Integer, Integer>();
    for(int i=0; i<nums.length; i++){
        if(map.containsKey(nums[i])){
            return new int[]{map.get(nums[i]), i};
        }else{
            map.put(target-nums[i], i);
        }
    }

    return new int[]{0,0};
}
```

分析：空间复杂度和时间复杂度均为 O(n)
