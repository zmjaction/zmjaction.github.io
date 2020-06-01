---
title: Kafka -- 常用脚本
mathjax: false
date: 2019-09-27 09:49:25
categories:
    - Middleware
    - MQ
    - Kafka
tags:
    - Middleware
    - MQ
    - Kafka
    - Stream
---

## 脚本列表
```
connect-distributed              kafka-consumer-perf-test         kafka-reassign-partitions        kafka-verifiable-producer
connect-standalone               kafka-delegation-tokens          kafka-replica-verification       trogdor
kafka-acls                       kafka-delete-records             kafka-run-class                  zookeeper-security-migration
kafka-broker-api-versions        kafka-dump-log                   kafka-server-start               zookeeper-server-start
kafka-configs                    kafka-log-dirs                   kafka-server-stop                zookeeper-server-stop
kafka-console-consumer           kafka-mirror-maker               kafka-streams-application-reset  zookeeper-shell
kafka-console-producer           kafka-preferred-replica-election kafka-topics
kafka-consumer-groups            kafka-producer-perf-test         kafka-verifiable-consumer
```

<!-- more -->

#### 脚本注解

* `connect-standalone 和 connect-distributed`: 这两个脚本是 Kafka Connect 组件的启动脚本

* `kafka-acls`: 它是用于设置 Kafka 权限的，比如设置哪些用户可以访问 Kafka 的哪些主题之类的权限

* `kafka-broker-api-versions`: 这个脚本的主要目的是验证不同 Kafka 版本之间服务器和客户端的适配性

* ` kafka-configs`: 主要参数配置

* `kafka-console-consumer`：消费者脚本

* `kafka-console-producer`： 生产者脚本

* `kafka-producer-perf-test、 kafka-consumer-perf-test` ：生产者和消费者的性能测试工具

* `kafka-consumer-groups`:消费者组位移

* `kafka-delegation-tokens`：它是管理 Delegation Token 的。基于 Delegation Token 的认证是一种轻量级的认证机制，补充了现有的 SASL 认证机制

* `kafka-delete-records`：用于删除kafka分区消息，kafka本身有自己的自动消息删除策略，所以使用的不多

* `kafka-dump-log` ：非常实用，查看kafka消息文件的内容，消息的元数据信息，消息体本身

* `kafka-log-dirs`：查询各个Broker上的各个日志路径的磁盘占用情况

* `kafka-mirror-maker`：实现kafka集群间的消息同步的

* `kafka-preferred-replica-election`：执行 Preferred Leader 选举的。它可以为指定的主题执行“换 Leader”的操作

* `kafka-reassign-partitions`：用于执行分区副本迁移以及副本文件路径迁移

* `kafka-topics`：所有主题管理操作

* `kafka-run-class`： 执行任何带 main 方法的 Kafka 类

* `kafka-server-start、kafka-server-stop`：用于启动和停止 Kafka Broker 进程的

* `kafka-streams-application-reset`: 脚本用来给 Kafka Streams 应用程序重设位移，以便重新消费数据

* `kafka-verifiable-producer 、 kafka-verifiable-consumer`: 脚本是用来测试生产者和消费者功能的，过时，用不到

  ​

## kafka-broker-api-versions

```
kafka-broker-api-versions --bootstrap-server localhost:9092
localhost:9092 (id: 0 rack: null) -> (
	Produce(0): 0 to 7 [usable: 7],
	Fetch(1): 0 to 11 [usable: 11],
	ListOffsets(2): 0 to 5 [usable: 5],
	Metadata(3): 0 to 8 [usable: 8],
	LeaderAndIsr(4): 0 to 2 [usable: 2],
    ...
)
```
1. kafka-broker-api-versions脚本用于验证**不同Kafka版本**之间**服务器**和**客户端**的适配性
2. `Produce(0): 0 to 7 [usable: 7]`
    - Produce请求，序号为0，表示Kafka所有请求类型中的**第一号**请求
    - `0 to 7`表示Produce请求在Kafka 2.3中总共有**8个版本**
    - `usable: 7`表示当前连入这个Broker的客户端API能够使用的版本号是7，即**最新版本**
3. 在**0.10.2.0**之前，Kafka是**单向兼容**的，即高版本的Broker能够处理低版本Client发送的请求，反正则不行
4. 从**0.10.2.0**开始，Kafka正式支持**双向兼容**，即_**低版本的Broker也能处理高版本Client的请求**_

##kafka-topics

```bash
-- 创建主题 
    bin/kafka-topics.sh --create --zookeeper zk1:2181 --replication-factor 3 --partitions 1 --topic action
	bin/kafka-topics.sh --zookeeper zk1:2181 --create --topic action --partitions 1 --replication-factor 1 --config retention.ms=15552000000 --config max.message.bytes=5242880
	-- topic 定义topic名
    -- partitions 定义分区数
    -- replication-factor 定义副本数
-- 修改主题
	bin/kafka-configs.sh --zookeeper zk1:2181 --entity-type topics --entity-name action --alter --add-config max.message.bytes=10485760
-- 查看主题
	bin/kafka-topics.sh --describe --zookeeper zk1:2181 --topic action
-- 查看所有主题
	bin/kafka-topics.sh --zookeeper zk1:2181 --list
-- 删除主题
	bin/kafka-topics.sh --zookeeper  zk1:2181 --delete --topic first
```



## kafka-console-producer

```
$ ./kafka-console-producer.sh --broker-list 192.168.1.67:9092 --topic second --request-required-acks -1 --producer-property compression.type=lz4
>hello
>
```

## kafka-console-consumer
如果没有指定`group`，每次运行Console Consumer，都会**自动生成一个新的消费者组**（console-consumer开头）来消费
`--from-beginning`等同于将Consumer端参数`auto.offset.reset`设置为**`Earliest`**（默认值为`Latest`）
```
$ ./kafka-console-consumer.sh --bootstrap-server 192.168.1.67:9092 --topic second --group second --from-beginning --consumer-property enable.auto.commit=false
hello
```

## kafka-producer-perf-test
向指定专题发送1千万条消息，每条消息大小为**1KB**，一般关注**99th**分位即可，可以作为该生产者对外承诺的**SLA**
```
$ ./kafka-producer-perf-test.sh --topic second --num-records 10000000 --throughput -1 --record-size 1024 --producer-props bootstrap.servers=192.168.1.67:9092 acks=-1 linger.ms=2000 compression.type=lz4
24041 records sent, 4802.4 records/sec (4.69 MB/sec), 2504.6 ms avg latency, 3686.0 ms max latency.
374745 records sent, 74829.3 records/sec (73.08 MB/sec), 3955.9 ms avg latency, 4258.0 ms max latency.
530462 records sent, 106092.4 records/sec (103.61 MB/sec), 5043.2 ms avg latency, 5979.0 ms max latency.
973178 records sent, 194402.3 records/sec (189.85 MB/sec), 4616.0 ms avg latency, 6092.0 ms max latency.
1114737 records sent, 222858.3 records/sec (217.64 MB/sec), 3226.9 ms avg latency, 3501.0 ms max latency.
1274003 records sent, 254647.8 records/sec (248.68 MB/sec), 2967.1 ms avg latency, 3258.0 ms max latency.
1315568 records sent, 262798.2 records/sec (256.64 MB/sec), 2655.6 ms avg latency, 2790.0 ms max latency.
1384239 records sent, 276847.8 records/sec (270.36 MB/sec), 2665.2 ms avg latency, 2799.0 ms max latency.
1419418 records sent, 283883.6 records/sec (277.23 MB/sec), 2509.9 ms avg latency, 2612.0 ms max latency.
10000000 records sent, 201751.200420 records/sec (197.02 MB/sec), 3041.05 ms avg latency, 6092.00 ms max latency, 2684 ms 50th, 5500 ms 95th, 5999 ms 99th, 6085 ms 99.9th.
```

## kafka-consumer-perf-test
```
$ ./kafka-consumer-perf-test.sh --broker-list 192.168.1.67:9092 --messages 10000000 --topic second
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec, rebalance.time.ms, fetch.time.ms, fetch.MB.sec, fetch.nMsg.sec
2019-09-27 19:46:34:219, 2019-10-30 19:46:52:092, 9765.6250, 546.3898, 10000002, 559503.2731, 50, 17823, 547.9226, 561072.8834
```

## 查看主题消息总数
```

$ ./kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 192.168.1.67:9092 --time -2 --topic second
second:0:0

$ ./kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 192.168.1.67:9092 --time -1 --topic second
second:0:10000014
```

## 查看消息文件数据
`--files`显式的是消息批次或消息集合的**元数据**信息
```
$ ./kafka-dump-log.sh --files /tmp/kafka-logs/transaction-0/00000000000000635791.log | head -n 10
Dumping /tmp/kafka-logs/transaction-0/00000000000000635791.log
Starting offset: 635791
baseOffset: 635791 lastOffset: 635791 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 0 CreateTime: 1590738831653 isvalid: true size: 51272 magic: 2 compresscodec: NONE crc: 2926696969
baseOffset: 635792 lastOffset: 635792 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 51272 CreateTime: 1590738831657 isvalid: true size: 51272 magic: 2 compresscodec: NONE crc: 3982704702
baseOffset: 635793 lastOffset: 635793 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 102544 CreateTime: 1590738831657 isvalid: true size: 51272 magic: 2 compresscodec: NONE crc: 3982704702
baseOffset: 635794 lastOffset: 635794 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 153816 CreateTime: 1590738831657 isvalid: true size: 51272 magic: 2 compresscodec: NONE crc: 3982704702
baseOffset: 635795 lastOffset: 635795 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 205088 CreateTime: 1590738831657 isvalid: true size: 51272 magic: 2 compresscodec: NONE crc: 3982704702
baseOffset: 635796 lastOffset: 635796 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 256360 CreateTime: 1590738831657 isvalid: true size: 51272 magic: 2 compresscodec: NONE crc: 3982704702
baseOffset: 635797 lastOffset: 635797 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 307632 CreateTime: 1590738831657 isvalid: true size: 51272 magic: 2 compresscodec: NONE crc: 3982704702
baseOffset: 635798 lastOffset: 635798 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 358904 CreateTime: 1590738831657 isvalid: true size: 51272 magic: 2 compresscodec: NONE crc: 3982704702
```
`--deep-iteration`用于查看每条**具体**的消息
```

$ ./kafka-dump-log.sh --files /tmp/kafka-logs/transaction-0/00000000000000635791.log --deep-iteration | head -n 25
Dumping /tmp/kafka-logs/transaction-0/00000000000000635791.log
Starting offset: 635791
offset: 635791 position: 0 CreateTime: 1590738831653 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635792 position: 51272 CreateTime: 1590738831657 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635793 position: 102544 CreateTime: 1590738831657 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635794 position: 153816 CreateTime: 1590738831657 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635795 position: 205088 CreateTime: 1590738831657 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635796 position: 256360 CreateTime: 1590738831657 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635797 position: 307632 CreateTime: 1590738831657 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635798 position: 358904 CreateTime: 1590738831657 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635799 position: 410176 CreateTime: 1590738831657 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635800 position: 461448 CreateTime: 1590738831658 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635801 position: 512720 CreateTime: 1590738831660 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635802 position: 563992 CreateTime: 1590738831660 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635803 position: 615264 CreateTime: 1590738831661 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635804 position: 666536 CreateTime: 1590738831661 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635805 position: 717808 CreateTime: 1590738831661 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635806 position: 769080 CreateTime: 1590738831662 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635807 position: 820352 CreateTime: 1590738831662 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635808 position: 871624 CreateTime: 1590738831663 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635809 position: 922896 CreateTime: 1590738831663 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635810 position: 974168 CreateTime: 1590738831663 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635811 position: 1025440 CreateTime: 1590738831668 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635812 position: 1076712 CreateTime: 1590738831668 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
offset: 635813 position: 1127984 CreateTime: 1590738831668 isvalid: true keysize: -1 valuesize: 51200 magic: 2 compresscodec: NONE producerId: -1 producerEpoch: -1 sequence: -1 isTransactional: false headerKeys: []
```
`--print-data-log`用于查看**消息里面的实际数据**
```
$ kafka-dump-log --files /tmp/kafka-logs/transaction-0/00000000000000000000.log --deep-iteration --print-data-log | head -n 8
Dumping /usr/local/var/lib/kafka-logs/transaction-0/00000000000000000000.log
Starting offset: 0
baseOffset: 0 lastOffset: 0 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 0 CreateTime: 1572434815538 size: 79 magic: 2 compresscodec: NONE crc: 234703942 isvalid: true
| offset: 0 CreateTime: 1572434815538 keysize: -1 valuesize: 11 sequence: -1 headerKeys: [] payload: hello world
baseOffset: 1 lastOffset: 1 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 79 CreateTime: 1572434920055 size: 69 magic: 2 compresscodec: NONE crc: 1305542871 isvalid: true
| offset: 1 CreateTime: 1572434920055 keysize: -1 valuesize: 1 sequence: -1 headerKeys: [] payload: w
baseOffset: 2 lastOffset: 16 count: 15 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 148 CreateTime: 1572435583918 size: 1241 magic: 2 compresscodec: LZ4 crc: 3547131642 isvalid: true
| offset: 2 CreateTime: 1572435583623 keysize: -1 valuesize: 1024 sequence: -1 headerKeys: [] payload: SSXVNJHPDQDXVCRASTVYBCWVMGNYKRXVZXKGXTSPSJDGYLUEGQFLAQLOCFLJBEPOWFNSOMYARHAOPUFOJHHDXEHXJBHWGSMZJGNLONJVXZXZOZITKXJBOZWDJMCBOSYQQKCPRRDCZWMRLFXBLGQPRPGRNTAQOOSVXPKJPJLAVSQCCRXFRROLLHWHOHFGCFWPNDLMWCSSHWXQQYKALAAWCMXYLMZALGDESKKTEESEMPRHROVKUMPSXHELIDQEOOHOIHEGJOAZBVPUMCHSHGXZYXXQRUICRIJGQEBBWAXABQRIRUGZJUUVFYQOVCDEDXYFPRLGSGZXSNIAVODTJKSQWHNWVPSAMZKOUDTWHIORJSCZIQYPCZMBYWKDIKOKYNGWPXZWMKRDCMBXKFUILWDHBFXRFAOPRUGDFLPDLHXXCXCUPLWGDPPHEMJGMTVMFQQFVCUPOFYWLDUEBICKPZKHKVMCJVWVKTXBKAPWAPENUEZNWNWDCACDRLPIPHJQQKMOFDQSPKKNURFBORJLBPCBIWTSJNPRBNITTKJYWAHWGKZYNUSFISPIYPIOGAUPZDXHCFVGXGIVVCPFHIXAACZXZLFDMOOSSNTKUPJQEIRRQAMUCTBLBSVPDDYOIHAOODZNJTVHDCIEGTAVMYZOCIVSKUNSMXEKBEWNZPRPWPUJABJXNQBOXSHOEGMJSNBUTGTIFVEQPSYBDXEXORPQDDODZGBELOISTRWXMEYWVVHGMJKWLJCCHPKAFRASZEYQZCVLFSLOWTLBMPPWPPFPQSAZPTULSTCDMODYKZGSRFQTRFTGCNMNXQQIYVUQZHVNIPHZWVBSGOBYIFDNNXUTBBQUYNXOZCSICGRTZSSRHROJRGBHMHEQJRDLOQBEPTOBMYLMIGPPDPOLTEUVDGATCGYPQOGOYYESKEGBLOCBIYSLQEYGCCIPBXPNSPKDYTBEWDHBHWVDPLOVHJPNYGJUHKKHDASNFGZDAIWWQEPPBRJKDGOSAFAPRLWFFXRVMZQTKRYF
```

## 查询消费者组位移
`CURRENT-OFFSET`表示该消费者当前消费的最新位移，`LOG-END-OFFSET`表示对应分区最新生产消息的位移
```
$ ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group transaction
```
参考：<http://zhongmingmao.me>