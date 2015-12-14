###KAFKA notetext

启动kafka

    bin/kafka-server-start.sh config/server.properties  &

创建topic

    bin/kafka-topics.sh --zookeeper zk_host:port/chroot --create --topic my_topic_name --partitions 20 --replication-factor 3 --config x=y

> `ex:./kafka-topics.sh --zookeeper 127.0.0.1:2181/kafka_2_10 --alter --topic avrotest1 --config retention.ms=128 `

修改topic

    bin/kafka-topics.sh --zookeeper zk_host:port/chroot --alter --topic my_topic_name --partitions 40

>建topic 的时候可以指定topic 的自己的相关配置与集群配置冲突，优先走topic自己的配置，未配置的走集群配置

    ./kafka-topics.sh --zookeeper 127.0.0.1:2181/kafka_2_10 --create --topic avrotest1 --partitions 1 --replication-factor 1 --config max.message.bytes=64000 --config flush.messages=1  
    ./kafka-topics.sh --zookeeper 127.0.0.1:2181/kafka_2_10 --alter --topic avrotest1 --config retention.ms=128 

删除topic：  
  
1. 标记这个topic不可用：
	`kafka-topics.sh --delete --zookeeper host:port --topic topicname`
2. 删除kafka存储目录（server.properties文件log.dirs配置，默认为"/tmp/kafka-logs"）相关topic目录删除zookeeper"/brokers/topics/"目录下相关topic节点
3. 删掉元数据
	`bin/kafka-run-class.sh kafka.admin.DeleteTopicCommand --zookeeper localhost:2181 --topic test`

查看创建的kafka的topic

    bin/kafka-topics.sh --describe --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --topic my-replicated-topic5
    Topic:my-replicated-topic5	PartitionCount:5	ReplicationFactor:3	Configs:
    	Topic: my-replicated-topic5	Partition: 0	Leader: 0	Replicas: 0,1,2	Isr: 0
    	Topic: my-replicated-topic5	Partition: 1	Leader: 1	Replicas: 1,2,0	Isr: 1,2,0
    	Topic: my-replicated-topic5	Partition: 2	Leader: 2	Replicas: 2,0,1	Isr: 2,0,1
    	Topic: my-replicated-topic5	Partition: 3	Leader: 0	Replicas: 0,2,1	Isr: 0
    	Topic: my-replicated-topic5	Partition: 4	Leader: 1	Replicas: 1,0,2	Isr: 1,0,2
>pt-kfk-01.gg.sdodr.pri.sdo.com:2181,pt-kfk-02.gg.sdodr.pri.sdo.com:2181,pt-kfk-03.gg.sdodr.pri.sdo.com:2181
生产消息、消费消息

    bin/kafka-console-producer.sh --broker-list zk-01:9092,zk-02:9092,zk-03:9092 --topic my-replicated-topic5
    bin/kafka-console-consumer.sh --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --from-beginning --topic my-replicated-topic5



####集群试验（迁移）：

#####一、用迁移工具自动迁移：
    [root@node2 kafka]# bin/kafka-topics.sh --describe --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --topic my-replicated-topic5
    Topic:my-replicated-topic5	PartitionCount:5	ReplicationFactor:3	Configs:
    	Topic: my-replicated-topic5	Partition: 0	Leader: 0	Replicas: 0,1,2	Isr: 0,1,2
    	Topic: my-replicated-topic5	Partition: 1	Leader: 1	Replicas: 1,2,0	Isr: 0,1,2
    	Topic: my-replicated-topic5	Partition: 2	Leader: 2	Replicas: 2,0,1	Isr: 0,1,2
    	Topic: my-replicated-topic5	Partition: 3	Leader: 0	Replicas: 0,2,1	Isr: 0,1,2
    	Topic: my-replicated-topic5	Partition: 4	Leader: 1	Replicas: 1,0,2	Isr: 0,1,2
    	
    [root@node1 kafka]# cat topics-to-move.json 
    {"topics": [{"topic": "my-replicated-topic5"}],
     "version":1
    }
    
    [root@node1 kafka]# bin/kafka-reassign-partitions.sh --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --topics-to-move-json-file topics-to-move.json --broker-list "3" --generate 

>这里如果--broker-list "1,2,3",则会把编号3的broker加进来，但是副本数还是不变

`Partitions reassignment failed due to replication factor: 3 larger than available brokers: 1`
**这里迁移一个broker报错，会提示说复制因子>迁移的broker数量**

    kafka.admin.AdminOperationException: replication factor: 3 larger than available brokers: 1
    	at kafka.admin.AdminUtils$.assignReplicasToBrokers(AdminUtils.scala:70)
    	at kafka.admin.ReassignPartitionsCommand$$anonfun$generateAssignment$1.apply(ReassignPartitionsCommand.scala:97)
    	at kafka.admin.ReassignPartitionsCommand$$anonfun$generateAssignment$1.apply(ReassignPartitionsCommand.scala:96)
    	at scala.collection.immutable.Map$Map1.foreach(Map.scala:116)
    	at kafka.admin.ReassignPartitionsCommand$.generateAssignment(ReassignPartitionsCommand.scala:96)
    	at kafka.admin.ReassignPartitionsCommand$.main(ReassignPartitionsCommand.scala:45)
    	at kafka.admin.ReassignPartitionsCommand.main(ReassignPartitionsCommand.scala) 

---

    [root@node2 kafka]# bin/kafka-topics.sh --describe --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --topic my-replicated-topic1
	Topic:my-replicated-topic1	PartitionCount:3	ReplicationFactor:1	Configs:
	Topic: my-replicated-topic1	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: my-replicated-topic1	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: my-replicated-topic1	Partition: 2	Leader: 1	Replicas: 1	Isr: 1

topic1只有1个复制因子（包括leader)

    [root@node1 kafka]# cat topics-to-move.json 
    {"topics": [{"topic": "my-replicated-topic5"}],
     "version":1
    }
构建分区结构

    [root@node1 kafka]# bin/kafka-reassign-partitions.sh --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --topics-to-move-json-file topics-to-move.json --broker-list "2,3" --generate 
>这条命令作用是把扩容前后的partition中part的架构分布列出。保留扩容前的分区架构图是为了扩容后出现问题回滚

    	Current partition replica assignment
    
    {"version":1,
    "partitions":[{"topic":"my-replicated-topic1","partition":0,"replicas":[1]},
    			  {"topic":"my-replicated-topic1","partition":2,"replicas":[1]},
    			  {"topic":"my-replicated-topic1","partition":1,"replicas":[0]}]
    }
    Proposed partition reassignment configuration
    
    {"version":1,
    "partitions":[{"topic":"my-replicated-topic1","partition":0,"replicas":[2]},
    		  {"topic":"my-replicated-topic1","partition":2,"replicas":[2]},
    			  {"topic":"my-replicated-topic1","partition":1,"replicas":[3]}]
    }

把新分区表写入expand-cluster-reassignment.json文件中，执行下面命令就把broker 迁移到集群里了

    bin/kafka-reassign-partitions.sh --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --reassignment-json-file expand-cluster-reassignment.json --execute

再次查看topic

    [root@node1 kafka]# bin/kafka-topics.sh --describe --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --topic my-replicated-topic1
    Topic:my-replicated-topic1	PartitionCount:3	ReplicationFactor:1	Configs:
    	Topic: my-replicated-topic1	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
    	Topic: my-replicated-topic1	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
    	Topic: my-replicated-topic1	Partition: 2	Leader: 2	Replicas: 2	Isr: 2

看成功迁移了，这个topc1的复制因子是1，所以扩容了还是只有1个，Isr列表是当前和leader正在进行同步的节点，如果消息数据延迟差太多的话，没中断与leader的联系
这样就会从Isr列表中去掉，Isr列表中都是最接近leader的节点，leader挂了，一般新leader会从Isr列表中按顺序从做到右优先选取，挂点的节点恢复后会添加到Isr的右边末尾

验证迁移是否成功：

    [root@node1 kafka]# bin/kafka-reassign-partitions.sh --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --reassignment-json-file expand-cluster-reassignment.json --verify
    Status of partition reassignment:
    Reassignment of partition [my-replicated-topic1,0] completed successfully
    Reassignment of partition [my-replicated-topic1,2] completed successfully
    Reassignment of partition [my-replicated-topic1,1] completed successfully

####二、手动迁移
>（和自动不一样的地方，就是手动迁移json文件是要自己写出来，其他命令一致）

###三、增加副本
>本次实验室增加带有副本的broker节点（相当于既增加broker数又增加副本数，如果我只想增加broker数呢？**参考迁移的操作**）

当前分区结构：

    	[root@node1 kafka]# bin/kafka-topics.sh --describe --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --topic my-replicated-topic5
    Topic:my-replicated-topic5	PartitionCount:5	ReplicationFactor:3	Configs:
    	Topic: my-replicated-topic5	Partition: 0	Leader: 0	Replicas: 0,1,2	Isr: 1,2,0
    	Topic: my-replicated-topic5	Partition: 1	Leader: 1	Replicas: 1,2,0	Isr: 1,2,0
    	Topic: my-replicated-topic5	Partition: 2	Leader: 2	Replicas: 2,0,1	Isr: 1,2,0
    	Topic: my-replicated-topic5	Partition: 3	Leader: 0	Replicas: 0,2,1	Isr: 1,2,0
    	Topic: my-replicated-topic5	Partition: 4	Leader: 1	Replicas: 1,0,2	Isr: 1,2,0

编辑分区json文件（加入节点3）：

    vim increase-replication-factor.json 	
    {"version":1,
    "partitions":[{"topic":"my-replicated-topic5","partition":0,"replicas":[0,1,2,3]},
    			  {"topic":"my-replicated-topic5","partition":1,"replicas":[0,1,2,3]},
    			  {"topic":"my-replicated-topic5","partition":2,"replicas":[0,1,2,3]},
    			  {"topic":"my-replicated-topic5","partition":3,"replicas":[0,1,2,3]},
    			  {"topic":"my-replicated-topic5","partition":4,"replicas":[0,1,2,3]}]
    }	

执行扩容副本：

    bin/kafka-reassign-partitions.sh --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --reassignment-json-file increase-replication-factor.json --execute

验证：

    [root@node1 kafka]# bin/kafka-reassign-partitions.sh --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --reassignment-json-file increase-replication-factor.json --verify
    Status of partition reassignment:
    Reassignment of partition [my-replicated-topic5,4] completed successfully
    Reassignment of partition [my-replicated-topic5,0] completed successfully
    Reassignment of partition [my-replicated-topic5,2] completed successfully
    Reassignment of partition [my-replicated-topic5,1] completed successfully
    Reassignment of partition [my-replicated-topic5,3] completed successfully
    
    [root@node1 kafka]# bin/kafka-topics.sh --describe --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --topic my-replicated-topic5
    Topic:my-replicated-topic5	PartitionCount:5	ReplicationFactor:4	Configs:
    	Topic: my-replicated-topic5	Partition: 0	Leader: 0	Replicas: 0,1,2,3	Isr: 1,2,0,3
    	Topic: my-replicated-topic5	Partition: 1	Leader: 0	Replicas: 0,1,2,3	Isr: 1,2,0,3
    	Topic: my-replicated-topic5	Partition: 2	Leader: 0	Replicas: 0,1,2,3	Isr: 1,2,0,3
    	Topic: my-replicated-topic5	Partition: 3	Leader: 0	Replicas: 0,1,2,3	Isr: 1,2,0,3
    	Topic: my-replicated-topic5	Partition: 4	Leader: 0	Replicas: 0,1,2,3	Isr: 1,2,0,3


添加复制因子成功

   	
配置文件：

    Here is our server production server configuration:
    # Replication configurations
    num.replica.fetchers=4   #用于从leader复制消息的线程数，增加这个值会增加follow的io并行度压力
    replica.fetch.max.bytes=1048576 #在一次发向leader要求复制的请求中，试图取出消息的最大字节数
    replica.fetch.wait.max.ms=500 #向leader发完要求复制的请求后，等待leader回复的最大的等待时间
    replica.high.watermark.checkpoint.interval.ms=5000  #
    replica.socket.timeout.ms=30000
    replica.socket.receive.buffer.bytes=65536
    replica.lag.time.max.ms=10000   #follower在这个时间内，没有发送拉取消息的请求，leader就会自动在Isr列表去掉它，认为它已经死掉
    replica.lag.max.messages=4000  #如果follower落后leader这么多的消息数量，leader也会自动在Isr列表去掉它
    
    controller.socket.timeout.ms=30000
    controller.message.queue.size=10
    
    
    
    
    # Log configuration
    num.partitions=8 #默认创建topic分区的数量
    message.max.bytes=1000000
    auto.create.topics.enable=true
    log.index.interval.bytes=4096
    log.index.size.max.bytes=10485760
    log.retention.hours=168
    log.flush.interval.ms=10000
    log.flush.interval.messages=20000
    log.flush.scheduler.interval.ms=2000
    log.roll.hours=168
    log.retention.check.interval.ms=300000
    log.segment.bytes=1073741824
    
    # ZK configuration
    zookeeper.connection.timeout.ms=6000
    zookeeper.sync.time.ms=2000
    
    # Socket server configuration
    num.io.threads=8
    num.network.threads=8
    socket.request.max.bytes=104857600
    socket.receive.buffer.bytes=1048576
    socket.send.buffer.bytes=1048576
    queued.max.requests=16
    fetch.purgatory.purge.interval.requests=100
    producer.purgatory.purge.interval.requests=100
    Our client configuration varies a fair amount between different use cases.

    平衡分配leader的功能
    
    auto.leader.rebalance.enable=true #开启自动平衡leader分配的功能
    leader.imbalance.check.interval.seconds
    leader.imbalance.per.broker.percentage

####kafka内置工具





    手动平衡
    KAFKA_HOME/bin/kafka-preferred-replica-election.sh --zookeeper localhost:2181
    

    ./bin/kafka-replica-verification.sh 
    $KAFKA_HOME/bin/kafka-replica-verification.sh，该工具用来验证所指定的一个或多个Topic下每个Partition对应的所有Replica是否都同步。
    可通过topic-white-list这一参数指定所需要验证的所有Topic，支持正则表达式。
    
    Option  Description
    ------  -----------
    --broker-list <hostname:port,...,   REQUIRED: The list of hostname and 
      hostname:port>  port of the server to connect to.
    --fetch-size <Integer: bytes>   The fetch size of each request.
      (default: 1048576)   
    --max-wait-ms <Integer: ms> The max amount of time each fetch  
      request waits. (default: 1000)   
    --report-interval-ms <Long: ms> The reporting interval. (default:  
      30000)   
    --time <Long: timestamp/-1(latest)/-2   Timestamp for getting the initial  
      (earliest)> offsets. (default: -1)   
    --topic-white-list <Java regex (String) White list of topics to verify replica 
      >   consistency. Defaults to all topics. 
       (default: .*)   
    
    
    [root@node1 kafka]# ./bin/kafka-replica-verification.sh --broker-list 192.168.81.128:9092,192.168.81.129:9092,192.168.81.130:9092 --topic-white-list my-replicated-topic5
    2015-08-27 14:07:44,985: verification process is started.
    2015-08-27 14:08:15,364: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:08:45,750: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:09:16,041: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:09:46,258: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:10:16,786: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:10:47,176: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:11:17,380: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:11:47,640: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:12:17,899: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:12:48,090: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:13:18,304: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:13:48,440: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    2015-08-27 14:14:18,610: max lag is 0 for partition [my-replicated-topic5,4] at offset 42 among 5 paritions
    
    
    
    bin/kafka-preferred-replica-election.sh 
    用途
    有了Replication机制后，每个Partition可能有多个备份。某个Partition的Replica列表叫作AR（Assigned Replicas），
    AR中的第一个Replica即为“Preferred Replica”。创建一个新的Topic或者给已有Topic增加Partition时，Kafka保证Preferred Replica被均匀分布到集群中的所有Broker上。
    理想情况下，Preferred Replica会被选为Leader。以上两点保证了所有Partition的Leader被均匀分布到了集群当中，这一点非常重要，因为所有的读写操作都由Leader完成，
    若Leader分布过于集中，会造成集群负载不均衡。但是，随着集群的运行，该平衡可能会因为Broker的宕机而被打破，该工具就是用来帮助恢复Leader分配的平衡。
    
    This tool causes leadership for each partition to be transferred back to the 'preferred replica', it can be used to balance leadership among the servers.
    Option  Description
    ------  -----------
    --path-to-json-file <list ofThe JSON file with the list of 
      partitions for which preferred  partitions for which preferred   
      replica leader election needs to be replica leader election should be
      triggered>  done, in the following format -  
    {"partitions": 
    	[{"topic": "foo", "partition": 1},
    	 {"topic": "foobar", "partition": 2}] 
    }  
    Defaults to all existing partitions
    --zookeeper <urls>  REQUIRED: The connection string for
      the zookeeper connection in the form 
      host:port. Multiple URLS can be  
      given to allow fail-over.   
    
    
    bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --topic my-replicated-topic5 --group web-console-consumer-79012 查看消费情况
    
    
    [root@node2 kafka]# bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper zk-01:2181,zk-02:2181,zk-03:2181 --topic my-replicated-topic5 --group web-console-consumer-79012
    Group   Topic  Pid Offset  logSize Lag Owner
    web-console-consumer-79012 my-replicated-topic5   0   68  68  0   none
    web-console-consumer-79012 my-replicated-topic5   1   19  19  0   none
    web-console-consumer-79012 my-replicated-topic5   2   17  17  0   none
    web-console-consumer-79012 my-replicated-topic5   3   5   9   4   none
    web-console-consumer-79012 my-replicated-topic5   4   10  10  0   none
    
    
    
    同一个消费者组消费者不能比parttition多。
    
    
    
    压力测试工具 zookeeper 2G x3 node / kafka 2G x 3 node 
    
    
    
    [root@NH-SDO-PT-KFK-113 kafka]# bin/kafka-topics.sh   --zookeeper 10.127.26.113:2181,10.127.26.114:2181,10.127.26.115:2181 --describe
    Topic:test_topic01  PartitionCount:5ReplicationFactor:3 Configs:
    Topic: test_topic01 Partition: 0Leader: 1003Replicas: 1003,1001,1002Isr: 1003,1001,1002
    Topic: test_topic01 Partition: 1Leader: 1001Replicas: 1001,1002,1003Isr: 1003,1001,1002
    Topic: test_topic01 Partition: 2Leader: 1002Replicas: 1002,1003,1001Isr: 1003,1001,1002
    Topic: test_topic01 Partition: 3Leader: 1003Replicas: 1003,1002,1001Isr: 1003,1001,1002
    Topic: test_topic01 Partition: 4Leader: 1001Replicas: 1001,1003,1002Isr: 1003,1001,1002
    Topic:test_topic02  PartitionCount:5ReplicationFactor:3 Configs:
    Topic: test_topic02 Partition: 0Leader: 1003Replicas: 1003,1002,1001Isr: 1003,1001,1002
    Topic: test_topic02 Partition: 1Leader: 1001Replicas: 1001,1003,1002Isr: 1001,1003,1002
    Topic: test_topic02 Partition: 2Leader: 1002Replicas: 1002,1001,1003Isr: 1001,1003,1002
    Topic: test_topic02 Partition: 3Leader: 1003Replicas: 1003,1001,1002Isr: 1003,1001,1002
    Topic: test_topic02 Partition: 4Leader: 1001Replicas: 1001,1002,1003Isr: 1001,1003,1002
    Topic:test_topic03  PartitionCount:5ReplicationFactor:2 Configs:
    Topic: test_topic03 Partition: 0Leader: 1002Replicas: 1002,1003 Isr: 1002,1003
    Topic: test_topic03 Partition: 1Leader: 1003Replicas: 1003,1001 Isr: 1003,1001
    Topic: test_topic03 Partition: 2Leader: 1001Replicas: 1001,1002 Isr: 1001,1002
    Topic: test_topic03 Partition: 3Leader: 1002Replicas: 1002,1001 Isr: 1002,1001
    Topic: test_topic03 Partition: 4Leader: 1003Replicas: 1003,1002 Isr: 1003,1002
    Topic:test_topic04  PartitionCount:6ReplicationFactor:3 Configs:
    Topic: test_topic04 Partition: 0Leader: 1003Replicas: 1003,1002,1001Isr: 1003,1002,1001
    Topic: test_topic04 Partition: 1Leader: 1001Replicas: 1001,1003,1002Isr: 1001,1003,1002
    Topic: test_topic04 Partition: 2Leader: 1002Replicas: 1002,1001,1003Isr: 1002,1001,1003
    Topic: test_topic04 Partition: 3Leader: 1003Replicas: 1003,1001,1002Isr: 1003,1001,1002
    Topic: test_topic04 Partition: 4Leader: 1001Replicas: 1001,1002,1003Isr: 1001,1002,1003
    Topic: test_topic04 Partition: 5Leader: 1002Replicas: 1002,1003,1001Isr: 1002,1003,1001
    Topic:test_topic05  PartitionCount:6ReplicationFactor:3 Configs:
    Topic: test_topic05 Partition: 0Leader: 1003Replicas: 1003,1002,1001Isr: 1003,1002,1001
    Topic: test_topic05 Partition: 1Leader: 1001Replicas: 1001,1003,1002Isr: 1001,1003,1002
    Topic: test_topic05 Partition: 2Leader: 1002Replicas: 1002,1001,1003Isr: 1002,1001,1003
    Topic: test_topic05 Partition: 3Leader: 1003Replicas: 1003,1001,1002Isr: 1003,1001,1002
    Topic: test_topic05 Partition: 4Leader: 1001Replicas: 1001,1002,1003Isr: 1001,1002,1003
    Topic: test_topic05 Partition: 5Leader: 1002Replicas: 1002,1003,1001Isr: 1002,1003,1001
    
    --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    bin/kafka-producer-perf-test.sh --messages 500000 --message-size 1000  --batch-size 1000 --topics test_topic02 --threads 4 --broker-list 10.127.26.113:9092,10.127.26.114:9092,10.127.26.115:9092
    
    kafka-producer-perf-test.sh中参数说明：
    messages生产者发送总的消息数量
    message-size每条消息大小
    batch-size每次批量发送消息的数量
    topics生产者发送的topic
    threads生产者使用几个线程同时发送
    broker-list安装kafka服务的机器ip:port列表
    producer-num-retries一个消息失败发送重试次数
    request-timeout-ms一个消息请求发送超时时间
    
    [root@NH-SDO-PT-KFK-113 kafka]# bin/kafka-producer-perf-test.sh --messages 500000 --message-size 1000  --batch-size 1000 --topics test_topic02 --threads 4 --broker-list 10.127.26.113:9092,10.127.26.114:9092,10.127.26.115:9092
    start.time, end.time, compression, message.size, batch.size, total.data.sent.in.MB, MB.sec, total.data.sent.in.nMsg, nMsg.sec
    2015-09-10 15:53:43:996, 2015-09-10 15:54:00:553, 0, 1000, 1000, 476.84, 28.7997, 500000, 30198.7075 
    
    #这里看到50w条消息，每条1000字节，每次发送1000条消息，只用了17s就完成了 50W x 1000b / (1024x1024) =476M 的数据量
    
    bin/kafka-consumer-perf-test.sh中参数说明：
    
    zookeeperzk配置
    messages消费者消费消息总数量
    topic消费者需要消费的topic
    threads消费者使用几个线程同时消费
    group消费者组名称
    socket-buffer-sizesocket缓冲大小
    fetch-size每次向kafka broker请求消费大小
    consumer.timeout.ms消费者去kafka broker拿去一条消息超时时间
    
    bin/kafka-consumer-perf-test.sh --zookeeper 10.127.26.113:2181,10.127.26.114:2181,10.127.26.115:2181 --messages 500000 --topic test_topic02 --threads 3
    
    [root@NH-SDO-PT-KFK-113 kafka]# bin/kafka-producer-perf-test.sh --messages 500000 --message-size 1000  --batch-size 1000 --topics test_topic02 --threads 4 --broker-list 10.127.26.113:9092,10.127.26.114:9092,10.127.26.115:9092
    start.time, end.time, compression, message.size, batch.size, total.data.sent.in.MB, MB.sec, total.data.sent.in.nMsg, nMsg.sec
    2015-09-10 15:53:43:996, 2015-09-10 15:54:00:553, 0, 1000, 1000, 476.84, 28.7997, 500000, 30198.7075 # 用17s
    
    [root@NH-SDO-PT-KFA-114 kafka]# bin/kafka-consumer-perf-test.sh --zookeeper 10.127.26.113:2181,10.127.26.114:2181,10.127.26.115:2181 --messages 500000 --topic test_topic02 --threads 3
    start.time, end.time, fetch.size, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec
    2015-09-10 16:11:33:539, 2015-09-10 16:11:42:170, 1048576, 476.8372, 131.3239, 500000, 137703.1121 用了9秒全部消费完50w消息
    
    
    
    
    
    [root@NH-SDO-PT-KFK-113 kafka]# bin/kafka-producer-perf-test.sh --messages 500000 --message-size 1000  --batch-size 1000 --topics test_
    topic03 --threads 4 --broker-list 10.127.26.113:9092,10.127.26.114:9092,10.127.26.115:9092
    start.time, end.time, compression, message.size, batch.size, total.data.sent.in.MB, MB.sec, total.data.sent.in.nMsg, nMsg.sec
    2015-09-11 11:35:50:398, 2015-09-11 11:36:00:753, 0, 1000, 1000, 476.84, 46.0490, 500000, 48285.8522 #用10秒
    
    [root@NH-SDO-PT-KFA-114 kafka]# bin/kafka-consumer-perf-test.sh --zookeeper 10.127.26.113:2181,10.127.26.114:2181,10.127.26.115:2181 --
    messages 500000 --topic test_topic03 --threads 3
    start.time, end.time, fetch.size, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec
    2015-09-11 11:37:51:404, 2015-09-11 11:37:59:269, 1048576, 476.8372, 166.4353, 500000, 174520.0698 #用8s
    
    
    
    [root@NH-SDO-PT-KFK-113 kafka]# bin/kafka-producer-perf-test.sh --messages 500000 --message-size 1000  --batch-size 1000 --topics test_
    topic05 --threads 4 --broker-list 10.127.26.113:9092,10.127.26.114:9092,10.127.26.115:9092
    start.time, end.time, compression, message.size, batch.size, total.data.sent.in.MB, MB.sec, total.data.sent.in.nMsg, nMsg.sec
    2015-09-11 11:53:36:937, 2015-09-11 11:53:50:311, 0, 1000, 1000, 476.84, 35.6540, 500000, 37385.9728 #14s
    
    [root@NH-SDO-PT-KFA-114 kafka]# bin/kafka-consumer-perf-test.sh --zookeeper 10.127.26.113:2181,10.127.26.114:2181,10.127.26.115:2181 --
    messages 500000 --topic test_topic05 --threads 3
    start.time, end.time, fetch.size, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec
    2015-09-11 11:55:41:692, 2015-09-11 11:55:49:765, 1048576, 476.8372, 155.1699, 500000, 162707.4520 #用8s



##疑问
kafka每个broker有leader，分区也有leader




##删除topic

    kafka-topics --zookeeper zk_host:port/chroot --delete --topic my_topic_name
    The topic will be "marked for deletion" and should be removed soon. What happens is an /admin/delete_topics/<topic> node is created in zookeeper and triggers a deletion. When the broker sees this update, the topic will no longer accept a new produce/consume request and eventually the topic will be deleted. If the delete command doesn't work right away, try restarting the Kafka service.
    To delete a Kafka topic after the Broker has lost connection to the topic:
    Manually delete the topic:
    If the delete commands fails (marked for deletion forever):
    1. Stop the Kafka Broker.
    2. Delete the topic from disk.
    For example:
    sudo mv /space1/kafka/FourSquareSearchApiResults-0 /space1/kafka/FourSquareSearchApiResults-0.bak
    3. Delete the topic znode from ZooKeeper:
     
    Here are the ZooKeeper commands to use to delete the topic from ZK:
    1. Log onto zkcli:
    hbase zkcli
    2. Search for topics:
    
    ls /brokers/topics
    rmr /brokers/topics/my_topic_name
    ls /brokers/topics
    3. Start Kafka.  #不需要重启 ，先用kafka-topic --delete 标记要删除的topic，之后再zk删掉topic，之后删掉kafka数据文件
    4. You can then recreate the Kafka topic.
    