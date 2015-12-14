# elasticsearch 集群

##Zen Discovery
[https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html#master-election](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html#master-election "Document")

zen discover 默认构建在discovery模块中。它提供nicast discovery，可以扩展到支持云环境或者其他形式的发现。

Zen discovery聚合一些其他的模块，例如
节点间的交流都是通过transport模块。

它被分离成下面几个模块：  

* ping  
这是节点用来发现其他节点机器的工具
* unicast  
和这个有关*discovery.zen.ping.unicast*


master选举：

* discovery.zen.ping_timeout （defaults to 3s) 
* discovery.zen.join_timeout   

当一个node加入，它会给master发送一个加入请求（discovery.zen.join_timeout），超时时间默认是ping_timeout的20倍


当master node停止，或者有问题，集群的其他nodes开始一次次一直ping，最后选出新的master。pinging一直在整个网络绕圈巡视，防护网络故障（比如某个链路网络问题，影响某个node错误地认为master死去，找不到存活的master）。这种情况下，这个node可以从其他node上获知当前存活的master


node.master = false 这个node将不可能选举为master
node.client=true，node将不准许选举为master，而且node.master自动回设置为false  
discovery.zen.minimum_master_nodes 设置最小能竞选为master的nodes数量。  
如果集群实际数量小于这个值，masternode将下台，并重新开始选举master。


默认存活检测方法和是否选举开始的条件：  
1. master ping the other nodes in the cluster and verify that they are alive.  
2. each node pings to master to verify if its still alive or an election process needs to be initiated.

The following settings control the fault detection process using the ***discovery.zen.fd prefix***:

**Setting**　　　　**Description**
----------------------------------------  
`ping_interval`　　　　How often a node gets pinged. Defaults to 1s.  
`ping_timeout`　　　　How long to wait for a ping response, defaults to 30s.　
