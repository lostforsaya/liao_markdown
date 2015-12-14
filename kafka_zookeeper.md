#Kafka+zookeeper搭建手册

###安装准备

三实体机: 
 
* 10.127.26.113
* 10.127.26.114
* 10.127.26.115

应用环境:
 
* jdk-7u51-linux-x64.gz
* kafka_2.10-0.8.2.2.tgz
* zookeeper-3.4.6.tar.gz

zookeeper集群3个节点，kafka集群3个节点

系统环境：*CentOS release 6.5*  

###安装过程

安装jdk：  
    mkdir /usr/lib/jvm
    tar –zxvf jdk-7u80-linux-x64.tar.gz –C /usr/lib/jvm
    
    vim /etc/profile.d/java.sh
    export PATH="$PATH:/usr/lib/jvm/jdk1.7.0_80/bin:/usr/lib/jvm/jdk1.7.0_80/jre/bin"
    export JAVA_HOME="/usr/lib/jvm/jdk1.7.0_80"
    export JAVA_JRE="/usr/lib/jvm/jdk1.7.0_80/jre"
    export CLASSPATH=".:/usr/lib/jvm/jdk1.7.0_80/lib"
    
    source /etc/profile

