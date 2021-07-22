# 算法

1.**问题：如何实现一个高效的单向链表逆序输出？**

**出题人：阿里巴巴出题专家：昀龙／阿里云弹性人工智能负责人**

**参考答案：下面是其中一种写法，也可以有不同的写法，比如递归等。供参考。**

```text
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

\*\*\*\*

**2.题目：已知 sqrt \(2\)约等于 1.414，要求不用数学库，求 sqrt \(2\)精确到小数点后 10 位。**

**出题人：——阿里巴巴出题专家：文景／阿里云 CDN 资深技术专家**

**参考答案：**

**\* 考察点**

1. 基础算法的灵活应用能力（二分法学过数据结构的同学都知道，但不一定往这个方向考虑；如果学过数值计算的同学，应该还要能想到牛顿迭代法并解释清楚）
2. 退出条件设计

**二分法**

**1. 已知 sqrt\(2\)约等于 1.414，那么就可以在\(1.4, 1.5\)区间做二分**

查找，如： a\) high=&gt;1.5 b\) low=&gt;1.4 c\) mid =&gt; \(high+low\)/2=1.45 d\) 1.45\*1.45&gt;2 ? high=&gt;1.45 : low =&gt; 1.45 e\) 循环到 c\)

**2. 退出条件**

a\) 前后两次的差值的绝对值&lt;=0.0000000001, 则可退出

```text
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

xn+1 = xn-f\(xn\)/f'\(xn\)

对于本题，需要求解的问题为：f\(x\)=x2-2 的零点

```text
EPSILON = 0.1 ** 10
def newton(x):
    if abs(x ** 2 - 2) > EPSILON:
        return newton(x - (x ** 2 - 2) / (2 * x))
    else:
        return x
```

