# 解释 SQL 的 left join 和 right join



**题目：解释 SQL 的 left join 和 right join**

**出题人：阿里巴巴新零售技术质量部**

**参考答案：**

left join 和 right join 都是两个表进行 merge 的操作，left join 是将右边的表 merge 到左边，right join 是将左边的表 merge 到右边，通常我们会指定按照哪几列进行 merge

举个例子：

**left table**

| 姓名 | 学号 |
| :--- | :--- |
| 小红 | SZ1716029 |
| 小明 | SZ1716030 |
| 小王 | SZ1716031 |

**right table**

| 学号 | 排名 |
| :--- | :--- |
| SZ1716029 | 1 |
| SZ1716030 | 2 |

**left table** left join **right table** on 学号

| 学号 | 姓名 | 排名 |
| :--- | :--- | :--- |
| SZ1716029 | 小红 | 1 |
| SZ1716030 | 小明 | 2 |
| SZ1716031 | 小王 | NULL |

**left table** right join **right table** on 学号

| 学号 | 姓名 | 排名 |
| :--- | :--- | :--- |
| SZ1716029 | 小红 | 1 |
| SZ1716030 | 小明 | 2 |

