---
title: python -- 序列类
date: 2019-06-12 11:11:23
categories:
    - python
tags:
    - python
    - kafka-python
---

#### kafka-python的安装

```
python3 -m pip install kafka-python
pipenv install kafka-python
```

#### kafka-python的基本使用

## KafkaConsumer

##### 消费端

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer("first", bootstrap_servers=['192.168.1.86:9092','192.168.1.86:9093','192.168.1.86:9094'], consumer_timeout_ms=1000)  # 连接kafka
for msg in consumer:
    print(msg)
# consumer_timeout_ms（ms毫秒）(非必选)
# 默认会一直等待，若指定则超时返回，不在等待
```

<!-- more -->

##### 消费端-加入消费者组

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer('first', group_id= 'group1', bootstrap_servers= [['192.168.1.86:9092','192.168.1.86:9093','192.168.1.86:9094'])
for msg in consumer:
    print(msg)
```

##### 消费者-手动分配分区

```python
from kafka import KafkaConsumer
from kafka import TopicPartition

consumer = KafkaConsumer(group_id= 'group1', bootstrap_servers= ['192.168.1.86:9092','192.168.1.86:9093','192.168.1.86:9094'])
consumer.assign([TopicPartition(topic= 'first', partition= 2)])
for msg in consumer:
    print(msg)

```

##### 消费者-订阅多个topic

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(group_id= 'group1', bootstrap_servers= ['192.168.1.86:9092','192.168.1.86:9093','192.168.1.86:9094'])
consumer.subscribe(topics= ['my_topic', 'topic_1'])
# 或者使用正则
# consumer.subscribe(pattern= '^my.*')
for msg in consumer:
    print(msg)
```

##### 消费者-msgpack消息

> MessagePack 是一个二进制序列化格式，多语言间数据转换
>
> 比json快，序列化结果比json小
>
> <https://msgpack.org/>

```python
from kafka import KafkaConsumer
consumer = KafkaConsumer('first', group_id= 'group1', bootstrap_servers= [['192.168.1.86:9092','192.168.1.86:9093','192.168.1.86:9094'],value_deserializer=msgpack.loads)
consumer.subscribe(['msgpackfoo'])
for msg in consumer:
    print(msg)
```

##### 消费者-earliest和latest

> `auto_offset_reset`这个参数，只有在一个`group`第一次运行的时候才有作用，从第二次运行开始，这个参数就失效了。
>
> 假设现在你的 Topic 里面有100个数据，你设置了一个全新的 group_id 为`test2`。`auto_offset_reset`设置为 `earliest`。那么当你的消费者运行的时候，Kafka 会先把你的 offset 设置为0，然后让你从头开始消费的。
>
> 假设现在你的 Topic 里面有100个数据，你设置了一个全新的 group_id 为`test3`。`auto_offset_reset`设置为 `latest`。那么当你的消费者运行的时候，Kafka 不会给你返回任何数据，消费者看起来就像卡住了一样，但是 Kafka 会直接强制把前100条数据的状态设置为已经被你消费的状态。所以当前你的 offset 就直接是99了。直到生产者插入了一条新的数据，此时消费者才能读取到。这条新的数据对应的 offset 就变成了100。
>
> 假设现在你的 Topic 里面有100个数据，你设置了一个全新的 group_id 为`test4`。`auto_offset_reset`设置为 `earliest`。那么当你的消费者运行的时候，Kafka 会先把你的 offset 设置为0，然后让你从头开始消费的。等消费到第50条数据时，你把消费者程序关了，把`auto_offset_reset`设置为`latest`，再重新运行。此时消费者依然会接着从第51条数据开始读取。不会跳过剩下的50条数据。
>
> 所以，auto_offset_reset的作用，是在你的 group 第一次运行，还没有 offset 的时候，给你设定初始的 offset。而一旦你这个 group 已经有 offset 了，那么auto_offset_reset这个参数就不会再起作用了。
>
> 以上注解为：公众号:未闻Code 如何使用Python读写Kafka？

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer("first", bootstrap_servers=['192.168.1.86:9092','192.168.1.86:9093','192.168.1.86:9094'],group_id='test',auto_offset_reset='earliest')  
# 默认为latest
for msg in consumer:
    print(msg)
```



## KafkaProducer

##### 生产者

```python
from kafka import KafkaProducer
import time
import datetime
SERVER = ['192.168.1.86:9092', '192.168.1.86:9093', '192.168.1.86:9094']
TOPIC = 'first'
producer = KafkaProducer(bootstrap_servers=SERVER)
for _ in range(10):
    producer.send('foobar', b'some_message_bytes')
```

##### 生产者- Block直到单条消息发送完成（或超时）

```
future = producer.send('foobar', b'another_message')
result = future.get(timeout=60)
```

##### 生产者-*Block直到所有阻塞的消息发送到网络*

```
producer.flush()
```



##### 生产者-json数据

```python
from kafka import KafkaProducer
import time
import datetime
SERVER = ['192.168.1.86:9092', '192.168.1.86:9093', '192.168.1.86:9094']
TOPIC = 'first'
producer = KafkaProducer(bootstrap_servers=SERVER
                         ,value_serializer=lambda v: json.dumps(v).encode('utf-8'))
for i in range(10):
    data = {'num': i, 'tims': datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
    producer.send("first", data)
    time.sleep(1)
```

##### 生产者-序列化字符串keys

```python
producer = KafkaProducer(key_serializer=str.encode)
producer.send('flipflap', key='ping', value=b'1234')
```

##### 生产者-*发送压缩消息*

```python
from kafka import KafkaProducer
import time
import datetime
SERVER = ['192.168.1.86:9092', '192.168.1.86:9093', '192.168.1.86:9094']
TOPIC = 'first'
producer = KafkaProducer(bootstrap_servers=SERVER
                         ,value_serializer=lambda v: json.dumps(v).encode('utf-8'), compression_type='lz4')  # 连接kafka
for i in range(10):
    data = {'num': i, 'tims': datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
    producer.send("first", data)
    time.sleep(1)
```

##### 生产者-*消息记录携带header*

````python
producer.send('foobar', value=b'c29tZSB2YWx1ZQ==', headers=[('content-encoding', b'base64')])
````

> 参考文章：
>
> <https://github.com/dpkp/kafka-python>
>
> <https://kafka-python.readthedocs.io/en/master/index.html>