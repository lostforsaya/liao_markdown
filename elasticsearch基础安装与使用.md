#elasticsearch笔记

##1.安装配置

安装java环境

	yum install openjdk
	java -version
	echo $JAVA_HOME
安装es

	curl -L -O https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.0.0/elasticsearch-2.0.0.tar.gz
	tar -zxvf elasticsearch-2.0.0.tar.gz

安装marvel监控管理工具

	./bin/plugin -i elasticsearch/marvel/latest
>如果你不想marvel监控你本地的集群，你可以禁用数据收集  
>**echo 'marvel.agent.enabled: false' >> ./config/elasticsearch.yml**

安装完成后前台运行： ```./bin/elasticsearch```  
![](http://i.imgur.com/DgZiOB6.png)
如果想在后台运行: 可以加个*-d*可以作为服务在后台运行



测试是否正常启动：

    curl 'http://localhost:9200/?pretty'

	{
	  "name" : "Tigra",
	  "cluster_name" : "elasticsearch",
	  "version" : {
	    "number" : "2.0.0",
	    "build_hash" : "de54438d6af8f9340d50c5c786151783ce7d6be5",
	    "build_timestamp" : "2015-10-22T08:09:48Z",
	    "build_snapshot" : false,
	    "lucene_version" : "5.2.1"
	  },
	  "tagline" : "You Know, for Search"
	}
    

集群健康检查：

	curl 'localhost:9200/_cat/health?v'


    epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent 
    1447814372 10:39:32  elasticsearch green   2 2 10   5000 0  -100.0% 

查看各节点的状态

    curl 'localhost:9200/_cat/nodes?v'
    host  ipheap.percent ram.percent load node.role master name   
    127.0.0.1 127.0.0.16  10 0.00 d *  Kehl of Tauran 
    127.0.0.1 127.0.0.16  10 0.00 d m  Calypso   

查看索引状态

	curl 'localhost:9200/_cat/indices?v'

###创建索引

	curl -XPUT 'localhost:9200/customer?pretty'
	curl 'localhost:9200/_cat/indices?v'

创建完索引，需要往索引里丢数据记录document，例如json document: {"name":"john Doe"}

	curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
	{
	  "name": "John Doe"
	}'
之后会返回：

    {
      "_index" : "customer",
      "_type" : "external",
      "_id" : "1",
      "_version" : 1,
      "created" : true
    }

这样就成功的在index customer中创建了个记录。  
>**如果创建记录的时候不存在这个索引，则会自动创建**
  
现在我们来查出创建的记录 

    curl -XGET 'localhost:9200/customer/external/1?pretty'
    {
      "_index" : "customer",
      "_type" : "external",
      "_id" : "1",
      "_version" : 1,
      "found" : true, "_source" : { "name": "John Doe" }
    }


index是创建索引，document被index索引的数据文档

###删除索引：

    curl -XDELETE 'localhost:9200/customer?pretty'
    curl 'localhost:9200/_cat/indices?v'

    curl -XDELETE 'localhost:9200/customer?pretty'
    {
      "acknowledged" : true
    }
    curl 'localhost:9200/_cat/indices?v'
    health index pri rep docs.count docs.deleted store.size pri.store.size

已成功删除

我们再把流程回顾下：

	#创建customer索引
	curl -XPUT 'localhost:9200/customer'

	#向customer索引中创建文档记录
    curl -XPUT 'localhost:9200/customer/external/1' -d '
    {
      "name": "John Doe"
    }'

	#查看创建文档记录
	curl 'localhost:9200/customer/external/1'
	#删除customer索引
	curl -XDELETE 'localhost:9200/customer'


###修改数据

替换索引的数据

	#创建一个index
	curl -XPUT 'localhost:9200/customer?pretty'

    #对index创建索引的文档记录，类型为external，id为1
    curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
    {
      "name": "John Doe"
    }'
	#如果重复执行创建文档记录，类型仍未external,id为1
    curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
    {
      "name": "Jane Doe"
    }'
	则第二条记录会替换第一条记录

查看external,id为1的记录

    [root@JJ-SDO-PT-LOGSTASH-199 ~]# curl -XGET 'localhost:9200/customer/external/1?pretty'
    {
      "_index" : "customer",
      "_type" : "external",
      "_id" : "1",
      "_version" : 2,#这里是创建了2次
      "found" : true,
      "_source":
    {
    "name": "Jane Doe"
    }
    }

替换成功

如果创建id为2的external类型的文档记录，将id为1的则不受影响


更新索引文档记录的数据

    curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
    {
      "doc": { "name": "Jane Doe" }
    }'

或者增加field

    curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
    {
      "doc": { "name": "Jane Doe", "age": 20 }
    }'

    curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
    {
      "script" : "ctx._source.age += 5"
    }'


删除索引中记录：

    curl -XDELETE 'localhost:9200/customer/external/2?pretty'



###批量修改
>批量操作依靠的是_bulk这个api  

    curl -XPOST 'localhost:9200/customer/external/_bulk?pretty' -d '
    {"index":{"_id":"1"}}
    {"name": "John Doe" }
    {"index":{"_id":"2"}}
    {"name": "Jane Doe" }

如果想删除id 为2的记录

    curl -XPOST 'localhost:9200/customer/external/_bulk?pretty' -d '
    {"update":{"_id":"1"}}
    {"doc": { "name": "John Doe becomes Jane Doe" } }
    {"delete":{"_id":"2"}}

批量处理是按顺序逐条执行的，如果有哪条执行失败，他会跳过去，继续执行剩余的，最后会返回结果，看哪些失败。


导入json格式的数据集：
>这个account.json文件我下载到本地了

    curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary "@accounts.json"
    curl 'localhost:9200/_cat/indices?v'

查看导入结果

	[root@JJ-SDO-PT-LOGSTASH-199 ~]# curl 'localhost:9200/_cat/indices?v'
	health status index    pri rep docs.count docs.deleted store.size pri.store.size 
	yellow open   bank       5   1       1000            0    445.9kb        445.9kb 
	yellow open   customer   5   1          2            0      6.1kb          6.1kb 

