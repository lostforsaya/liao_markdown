##redis 分析监控
目前我看到的redis有5台机器，分别是：  

* 10.127.26.41
* 10.127.26.42
* 10.127.26.145
* 10.127.26.146
* 10.158.132.53


先看10.127.26.41这台：  

	redis_home_dir=/opt/app/redisserver
	[root@NH-SDO-OP-REDIS-41 redisserver]# ls
	appendonly.aof  conf  db4kfc    db4lpe02   db4lpe102  db4lpe201  db4lsc4gauth  db4ran          dump.rdb
	bin             db    db4lpe01  db4lpe101  db4lpe103  db4lpe202  db4payweb     db4routeserver  var

>db开头的都是存放数据，配置文件统一存放conf下

先看**redis4kfc.conf**的配置：

    daemonize yes
    pidfile "../var/redis4lpe01.pid"
    port 11311
    timeout 0
    tcp-keepalive 0
    loglevel notice
    logfile "/opt/logs/redis4lpe01/redis.log"
    databases 16
    stop-writes-on-bgsave-error yes
    rdbcompression yes
    rdbchecksum yes
    dbfilename "dump.rdb"
    dir "/opt/app/redisserver/db4lpe01"
    slave-serve-stale-data yes
    slave-read-only yes
    repl-ping-slave-period 10
    repl-timeout 60
    repl-disable-tcp-nodelay yes
    repl-backlog-size 500mb
    repl-backlog-ttl 600
    slave-priority 1
    maxclients 10000
    maxmemory 29296875kb
    maxmemory-policy volatile-lru
    maxmemory-samples 3
    appendonly yes
    appendfsync no
    no-appendfsync-on-rewrite yes
    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 5000mb
    lua-time-limit 5000
    slowlog-log-slower-than 10000
    slowlog-max-len 128
    notify-keyspace-events ""
    hash-max-ziplist-entries 512
    hash-max-ziplist-value 64
    list-max-ziplist-entries 512
    list-max-ziplist-value 64
    set-max-intset-entries 512
    zset-max-ziplist-entries 128
    zset-max-ziplist-value 64
    activerehashing yes
    client-output-buffer-limit normal 0 0 0
    client-output-buffer-limit slave 512mb 64mb 300
    client-output-buffer-limit pubsub 32mb 8mb 60
    hz 10
    aof-rewrite-incremental-fsync yes