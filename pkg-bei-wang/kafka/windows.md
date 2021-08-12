# windows

```bash
创建主题
kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

查看主题
kafka-topics --list --zookeeper localhost:2181

创建生产者
kafka-console-producer --broker-list localhost:9092 --topic test

创建消费者
kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning

查看topic
kafka-topics --describe --zookeeper localhost:2181 --topic
```

