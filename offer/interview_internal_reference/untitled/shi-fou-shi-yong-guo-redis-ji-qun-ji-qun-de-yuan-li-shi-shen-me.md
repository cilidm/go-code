# 是否使用过Redis集群，集群的原理是什么？

**题目：是否使用过Redis集群，集群的原理是什么？**

**参考答案：**

Redis Sentinel着眼于高可用，在master宕机时会自动将slave提升为master，继续提供服务。

Redis Cluster着眼于扩展性，在单个redis内存不足时，使用Cluster进行分片存储。

