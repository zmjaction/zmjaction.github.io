---
title: 大数据 Hbase伪分布式集群安装
date: 2020-06-20 15:27:40
categories:
    - 大数据
    - Hbase
tags:
    - 大数据
    - Hbase
---

#### HBase伪分布式环境部署

```base
hadoop@hadoop001:~ $ cd ~/softeware
hadoop@hadoop001:~/software $ tar -zxvf hbase-1.2.4-bin.tar.gz –C ../app
hadoop@hadoop001:~/software $ cd ../app
hadoop@hadoop001:~/app $ cd hbase-1.2.4/conf
hadoop@hadoop001:~/app/hbase-1.2.4/conf $ cp ~/app/hadoop-2.7.3/etc/hadoop
/hdfs-site.xml .
hadoop@hadoop001:~/app/hbase-1.2.4/conf $ cp ~/app/hadoop-2.7.3/etc/hadoop
/core-site.xml .
hadoop@hadoop001:~/app/hbase-1.2.4/conf $ vi hbase-env.sh
export JAVA_HOME=~/app/jdk1.8.0_161
# export HBASE_MASTER_OPTS=”$HBASE_MASTER_OPTS –XX:PermSize=128m –XX:MaxPermSize
=128m”
# export HBASE_REGIONSERVER_OPTS=”$HBASE_REGIONSERVER_OPTS –XX:PermSize=128m –XX:
MaxPermSize=128m”
hadoop@hadoop001:~/app/hbase-1.2.4/conf $ vi hbase-site.xml
<configuration>
    <property>
        <name>hbase.root.dir </name>
        <value>hdfs://localhost:9000/hbase</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/home/hadoop/hadoop_data/zookeeper </value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
</configuration>
hadoop@hadoop001:~/app/hbase-1.2.4/conf $ cd ../bin
hadoop@hadoop001:~/app/hbase-1.2.4/bin $ ./start-hbase.sh
hadoop@hadoop001:~/app/hbase-1.2.4/bin $ jps
13792 HQuorumPeer
13968 HRegionServer
13864 HMaster
12156 NameNode
12556 SecondaryNameNode
14317 Jps
12335 DataNode
hadoop@hadoop001:~/app/hbase-1.2.4/bin $ ./hbase shell
hbase(main):001:0> status
1 active master, 0 backup master, 1 servers, 0 dead, 2.0000 average load 
hadoop@hadoop001:~/app/hbase-1.2.4/bin $ cd ~/app/hadoop-2.7.3/bin
hadoop@hadoop001: ~/app/hadoop-2.7.3/bin $ ./hdfs dfs –ls /
Found 2 items
drwxr-xr-x  - hadoop supergroup 0 2018-07-15 17:23 /hbase
drwxr-xr-x  - hadoop supergroup 0 2018-07-15 17:08 /test
```

