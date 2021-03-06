# 布隆过滤器



> **文章已收录Github精选，欢迎Star**：[https://github.com/yehongzhi](https://github.com/yehongzhi/learningSummary)

## 概念

布隆过滤器(BloomFilter)是由一个叫“布隆”的小伙子在1970年提出的，它是一个很长的二进制向量，主要**用于判断一个元素是否在一个集合中**。

## 原理

在介绍原理之前，要先讲一下**Hash函数**的概念。

我们在Java中的HashMap，HashSet其实也接触过hashcode()这个函数，哈希函数是可以将任意大小的输入数据转换成特定大小的输出数据的函数，转换后的数据称为**哈希值**。

哈希函数有以下特点：

* 如果根据同一个哈希函数得到的哈希值不同，那么这两个哈希值的原始输入值肯定不同。
* 如果根据同一个哈希函数得到的两个哈希值相等，两个哈希值的原始输入值有可能相等，有可能不相等。

布隆过滤器是由一个很长的二进制向量和一系列的哈希函数组成。那么布隆过滤器是怎么判断一个元素是否在一个集合中的呢？

假设布隆过滤器的底层存储结构是一个长度为16的位数组，初始状态时，它的所有位置都设置为0。

![](https://static.lovebilibili.com/redis\_bl\_01.png)

当有变量添加到布隆过滤器中，通过K个映射函数将变量映射到位数组的K个点，并把这K个点的值设置为1(假设有三个映射函数)。

![](https://static.lovebilibili.com/redis\_bl\_02.png)

查询某个变量是否存在的时候，我们只需要通过同样的K个映射函数，找到对应的K个点，判断K个点上的值是否全都是1，**如果全都是1则表示很可能存在**，如果**K个点上有任何一个是0则表示一定不存在**。

## 特性

第一个问题，为什么说全都是1的情况是很可能存在，而不是一定存在呢？

还记得前面说的哈希函数的特点，根据同一个哈希函数得到相同的哈希值，输入值不一定相等。类似于Java中两个对象的hashcode相等，但是不一定相等的道理。说白了，映射函数得到位数组上映射点全都是1，不一定是要查询的这个变量之前存进来时设置的，也有可能是其他变量映射的点。

所以这里引出了布隆过滤器的其中一个特点，**存在一定的误判**。

第二个问题，布隆过滤器能不能删除元素呢？

答案是不能的。因为在位数组上的同一个点有可能有多个输入值映射，如果删除了会影响布隆过滤器里其他元素的判断结果。

![](https://static.lovebilibili.com/redis\_bl\_03.png)

如上图，如果删除obj1，把4,7,15置为0，那么判断obj2是否存在时就会导致因为映射点7是0，结果判断obj2是不存在的，结果出错。

这是第二个特点，**不能删除布隆过滤器里的元素。**

## 优缺点

**优点：**

* 在空间和时间方面，都有着巨大的优势。因为不是存完整的数据，是一个二进制向量，能节省大量的内存空间，时间复杂度方面，是根据映射函数查询，假设有K个映射函数，那么时间复杂度就是O(K)。
* 因为存的不是元素本身，而是二进制向量，所以在一些对**保密性**要求严格的场景有一定优势。

**缺点：**

* \*\*存在一定的误判。\*\*存进布隆过滤器里的元素越多，误判率越高。
* \*\*不能删除布隆过滤器里的元素。\*\*随着使用的时间越来越长，因为不能删除，存进里面的元素越来越多，占用内存越来越多，误判率越来越高，最后不得不重置。

## 应用于缓存穿透

\*\*用于缓解缓存穿透。\*\*缓存穿透的问题主要是因为传进来的key在Redis中是不存在的，那么就会直接打在DB上，造成DB压力增大。

![](https://static.lovebilibili.com/redis\_hc\_2.png)

针对这种情况，可以在Redis前加上布隆过滤器，预先把数据库中的数据加入到布隆过滤器中，因为布隆过滤器的底层数据结构是一个二进制向量，所以占用的空间并不是很大。**在查询Redis之前先通过布隆过滤器判断是否存在，如果不存在就直接返回，如果存在的话，按照原来的流程还是查询Redis，Redis不存在则查询DB**。

这里主要利用的是**布隆过滤器判断结果是不存在的话就一定不存在**这一个特点，但是由于布隆过滤器有一定的误判，所以并不能说完全解决缓存穿透，但是能很大程度缓解缓存穿透的问题。

![](https://static.lovebilibili.com/redis\_bl\_04.png)

## 布隆过滤器插件

我们知道布隆过滤器的底层原理之后，理论上是可以自己

在Redis4.0后，官方提供了布隆过滤器的插件功能，布隆过滤器可以作为一个插件加载到Redis服务器直接使用。

首先安装Redis，网上有很多安装教程，这里就不多赘述。这里我用的是Redis6.0.10版本。安装完Redis之后，下载插件，使用git命令拉取：

```shell
git clone https://github.com/RedisBloom/RedisBloom.git
```

拉取下来之后会得到一个RedisBloom的项目。

![](https://static.lovebilibili.com/redis\_bl\_05.png)

然后cd到文件夹/RedisBloom，使用make命令编译。

![](https://static.lovebilibili.com/redis\_bl\_06.png)

编译完成后生成一个redisbloom.so文件。

![](https://static.lovebilibili.com/redis\_bl\_07.png)

在启动Redis时，加载布隆过滤器模块到服务器中即可。

```shell
./src/redis-server --loadmodule /usr/local/RedisBloom/redisbloom.so
```

最后使用客户端测试一下。

```
redis-6.0.10]$ ./src/redis-cli 

127.0.0.1:6379> bf.add user sam
(integer) 1
127.0.0.1:6379> bf.add user jack
(integer) 1
127.0.0.1:6379> bf.exists user jack
(integer) 1
127.0.0.1:6379> bf.exists user tom
(integer) 0
```

布隆过滤器的基本指令如下：

* bf.add 添加元素到布隆过滤器
* bf.exists 判断元素是否在布隆过滤器
* bf.madd 添加多个元素到布隆过滤器
* bf.mexists 判断多个元素是否在布隆过滤器

```shell
127.0.0.1:6379> bf.madd user mike rose
1) (integer) 1
2) (integer) 1
127.0.0.1:6379> bf.mexists user blue mike
1) (integer) 0
2) (integer) 1
```

在实际开发中，一般都是使用Java程序操作Redis，不太可能直接使用命令行判断，Java程序怎么操作呢？

首先引入依赖，我使用的是SpringBoot2.0，所以引入依赖是3.15.0。

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.15.0</version>
</dependency>
```

接着写个main方法示范一下。

```java
public static void main(String[] args) throws Exception {
    Config config = new Config();
    config.useSingleServer().setAddress("redis://192.168.0.109:6379");
    RedissonClient client = Redisson.create(config);

    RBloomFilter<String> bloomFilter = client.getBloomFilter("user");
    //尝试初始化，预计元素55000000，期望误判率0.03
    bloomFilter.tryInit(55000000L, 0.03);
    //添加元素到布隆过滤器中
    bloomFilter.add("tom");
    bloomFilter.add("mike");
    bloomFilter.add("rose");
    bloomFilter.add("blue");
    System.out.println("布隆过滤器元素总数为：" + bloomFilter.count());//布隆过滤器元素总数为：4
    System.out.println("是否包含tom：" + bloomFilter.contains("tom"));//是否包含tom：true
    System.out.println("是否包含lei：" + bloomFilter.contains("lei"));//是否包含lei：false
    client.shutdown();
}
```

## 总结

布隆过滤器有着明显的优缺点，所以在使用的时候需要充分地考虑场景，还是那句话，没有最好的技术，看菜下饭才是硬道理。除了在缓存穿透中使用之外，其实还可以使用于元素去重，web拦截器等等。

这篇文章就讲到这里，感谢大家的阅读，希望看完之后能有所收获。

**觉得有用就点个赞吧，你的点赞是我创作的最大动力**\~

**我是一个努力让大家记住的程序员。我们下期再见！！！**

![img](https://static.lovebilibili.com/dashacha.png)

> 能力有限，如果有什么错误或者不当之处，请大家批评指正，一起学习交流！
