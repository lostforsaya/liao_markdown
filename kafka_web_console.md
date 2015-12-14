#kafka web console install 

###功能介绍：
* brokers列表  
* 连接kafka的zk集群列表  
* 所有topic列表，操作相应topic可以浏览查看相应message生产和消费流量图.

####1. 下载Kafka Web Console

    https://github.com/claudemamo/kafka-web-console

####2. 安装sbt
    
    DEBIAN/UBUNTU:
    echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
    sudo apt-get update
    sudo apt-get install sbt
    
    CENTOS/RHEL：
    curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
    sudo yum install sbt

####3. 配置kafka web console

数据库配置：  
`yum install mysql-connector-java mysql-server`

这两个包一定要装：  
*mysql-connector-java-5.1.17-6.el6.noarch*  
*mysql-connector-odbc-5.1.5r1144-7.el6.x86_64*  

创建数据库用户：

    create database db_kafka character set utf8 collate utf8_bin
    grant all on db_kafka.* to kafka@'localhost' identified by 'kafka';

在build.sbt中添加mysql支持：  

    name := "kafka-web-console"
    
    version := "2.1.0-SNAPSHOT"
    
    libraryDependencies ++= Seq(
      jdbc,
      cache,
      "org.squeryl" % "squeryl_2.10" % "0.9.5-6",
      "com.twitter" % "util-zk_2.10" % "6.11.0",
      "com.twitter" % "finagle-core_2.10" % "6.15.0",
      "org.quartz-scheduler" % "quartz" % "2.2.1",
      "org.apache.kafka" % "kafka_2.10" % "0.8.1.1",
      "mysql" % "mysql-connector-java" % "5.1.17"
    	exclude("javax.jms", "jms")
    	exclude("com.sun.jdmk", "jmxtools")
    	exclude("com.sun.jmx", "jmxri")
    )
    
    play.Project.playScalaSettings


配置mysql的jdbc驱动：

    vim conf/application.conf
    
    db.default.driver=com.mysql.jdbc.Driver
    db.default.url="jdbc:mysql://localhost:3306/db_kafka"
    db.default.user=kafka
    db.default.pass=kafka

导入sql语句：  

    CREATE TABLE zookeepers (
    id BIGINT NOT NULL AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    host VARCHAR(255) NOT NULL,
    chroot VARCHAR(255),
    port BIGINT NOT NULL,
    statusId BIGINT NOT NULL,
    groupId BIGINT NOT NULL,
    PRIMARY KEY (id),
    UNIQUE (name)
    );
    
    CREATE TABLE status (
    id BIGINT,
    name VARCHAR(255),
    PRIMARY KEY (id)
    );
    
    CREATE TABLE groups (
    id BIGINT,
    name VARCHAR(255),
    PRIMARY KEY (id)
    );
    
    CREATE TABLE offsetHistory (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    zookeeperId BIGINT,
    topic VARCHAR(255),
    FOREIGN KEY (zookeeperId) REFERENCES zookeepers(id),
    UNIQUE (zookeeperId, topic)
    );
    
    CREATE TABLE offsetPoints (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    consumerGroup VARCHAR(255),
    timestamp TIMESTAMP,
    offsetHistoryId BIGINT,
    partition BIGINT,
    offset BIGINT,
    logSize BIGINT,
    FOREIGN KEY (offsetHistoryId) REFERENCES offsetHistory(id)
    );
    
    CREATE TABLE settings (
    key_ VARCHAR(255) PRIMARY KEY,
    value VARCHAR(255)
    );
    
    INSERT INTO status (id, name) VALUES (0, 'CONNECTING');
    INSERT INTO status (id, name) VALUES (1, 'CONNECTED');
    INSERT INTO status (id, name) VALUES (2, 'DISCONNECTED');
    INSERT INTO status (id, name) VALUES (3, 'DELETED');
    
    INSERT INTO groups (id, name) VALUES (0, 'ALL');
    INSERT INTO groups (id, name) VALUES (1, 'DEVELOPMENT');
    INSERT INTO groups (id, name) VALUES (2, 'PRODUCTION');
    INSERT INTO groups (id, name) VALUES (3, 'STAGING');
    INSERT INTO groups (id, name) VALUES (4, 'TEST');
    
    INSERT INTO settings (key_, value) VALUES ('PURGE_SCHEDULE', '0 0 0 ? * SUN *');
    INSERT INTO settings (key_, value) VALUES ('OFFSET_FETCH_INTERVAL', '30');



或者用这个模板：
    
    CREATE TABLE zookeepers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE,
    host VARCHAR(255) NOT NULL,
    port INT NOT NULL,
    statusId INT NOT NULL,
    groupId INT NOT NULL,
    chroot VARCHAR(255)
    );
    
    CREATE TABLE groups (
    id INT,
    name VARCHAR(255),
    PRIMARY KEY (id)
    );
    
    CREATE TABLE status (
    id INT,
    name VARCHAR(255),
    PRIMARY KEY (id)
    );
    
    CREATE TABLE offsetHistory (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    zookeeperId INT,
    topic VARCHAR(255),
    FOREIGN KEY zkId (zookeeperId) REFERENCES zookeepers(id) ON DELETE CASCADE,
    UNIQUE zkTopic (zookeeperId, topic)
    );
    
    CREATE TABLE offsetPoints (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    consumerGroup VARCHAR(255),
    timestamp DATETIME,
    offsetHistoryId BIGINT,
    partition INT,
    offset BIGINT,
    logSize BIGINT,
    FOREIGN KEY offhistId (offsetHistoryId) REFERENCES offsetHistory(id) ON DELETE CASCADE
    );
    
    CREATE TABLE settings (
    key_ VARCHAR(255) PRIMARY KEY,
    value VARCHAR(255)
    );
    
    INSERT INTO groups (id, name) VALUES (0, 'ALL');
    INSERT INTO groups (id, name) VALUES (1, 'DEVELOPMENT');
    INSERT INTO groups (id, name) VALUES (2, 'PRODUCTION');
    INSERT INTO groups (id, name) VALUES (3, 'STAGING');
    INSERT INTO groups (id, name) VALUES (4, 'TEST');
    
    INSERT INTO status (id, name) VALUES (0, 'CONNECTING');
    INSERT INTO status (id, name) VALUES (1, 'CONNECTED');
    INSERT INTO status (id, name) VALUES (2, 'DISCONNECTED');
    INSERT INTO status (id, name) VALUES (3, 'DELETED');
    
    INSERT INTO settings (key_, value) VALUES ('PURGE_SCHEDULE', '0 0 0 ? * SUN *');
    INSERT INTO settings (key_, value) VALUES ('OFFSET_FETCH_INTERVAL', '30');

删表参考:

    DROP TABLE IF EXISTS settings;
    DROP TABLE IF EXISTS offsetPoints;
    DROP TABLE IF EXISTS offsetHistory;
    DROP TABLE IF EXISTS status;
    DROP TABLE IF EXISTS groups;
    DROP TABLE IF EXISTS zookeepers;




####4. 编译安装


编译：
 
    sbt package

>在~/.ivy2下会下载很多包，最好预先下载好打包

生成一个standalone包：

	sbt run

>有java环境就可以允许

生成一个tar包：

	sbt dist  
>这个命令生成可以发布的包，在target/universal目录下。  

    解压
    unzip kafka-web-console-2.1.0-SNAPSHOT.zip  
    cd kafka-web-console-2.1.0-SNAPSHOT/bin  
    
    第一次启动时要加个参数：
    ./kafka-web-console -DapplyEvolutions.default=true 
    
    不然会报错：
    [warn] play - Run with -DapplyEvolutions.default=true if you want to run them automatically (be careful)  
    Oops, cannot start the server.  
    @6k1jkg3be: Database 'default' needs evolution!  
    at play.api.db.evolutions.EvolutionsPlugin$$anonfun$onStart$1$$anonfun$apply$1.apply$mcV$sp(Evolutions.scala:484) 
    
    
    ​查看帮助 和 后台运行：
    
    ./kafka-web-console -h  
    nohup ./kafka-web-console >/dev/null 2>&1 &  
    

    修改http服务端口：
    
    默认是9000端口。
    
    修改conf/application.conf 里的http.port，貌似不起作用。。
    
    可以通过命令行传递参数进去
    
    ./kafka-web-console  -Dhttp.port=9001  
    
    sbt设置代理
    http://stackoverflow.com/questions/13803459/how-to-use-sbt-from-behind-proxy