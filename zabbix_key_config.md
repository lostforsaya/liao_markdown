###zabbix监控设置（工作记录sdo）


**1. mysql ：**  

mysql添加zabbix用户，设置usage权限后zabbix只能查看mysql status等，无法查看其它库

    grant usage on *.* to zabbix@‘zabbix agentd的内网ip’ identified by ‘zabbix’;
	flush privileges

zabbix用户参数设置	


    UserParameter=mysql.status[*],echo "show global status where Variable_name='$1';" | mysql -N -uzabbix -pzabbix | awk '{print $$2}'
    UserParameter=mysql.ping, mysqladmin -uzabbix -pzabbix ping | grep -c alive
    UserParameter=mysql.version,mysql -V

**2. io ：**

    #monitor DISK IO
    UserParameter=custom.vfs.dev.read.ops[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$4}'
    UserParameter=custom.vfs.dev.read.ms[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$7}'
    UserParameter=custom.vfs.dev.write.ops[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$8}'
    UserParameter=custom.vfs.dev.write.ms[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$11}'
    UserParameter=custom.vfs.dev.io.active[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$12}'
    UserParameter=custom.vfs.dev.io.ms[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$13}'
    UserParameter=custom.vfs.dev.read.sectors[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$6}'
    UserParameter=custom.vfs.dev.write.sectors[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$10}'


**3. redis ：**

    UnsafeUserParameters=1
    UserParameter=redis.discovery,/opt/app/python27/bin/python /opt/app/zabbix/bin/redis_port.py
    #redis
    UserParameter=redis[*],/opt/app/redisserver/bin/redis-cli -h 127.0.0.1 -p $2 info|grep -w $1|cut -d : -f2
    #UserParameter=redis[*],/opt/app/zabbix/bin/redis_status.sh $1 $2 


lld脚本：

    #!/usr/bin/env python
    import os
    import json
    t=os.popen("""netstat -natp|awk -F: '/redis-server/&&/LISTEN/{print $2}'|grep -v "^$"|awk '{print $1}'""")
    ports = []
    for port in  t.readlines():
            r = os.path.basename(port.strip())
            ports += [{'{#REDISPORT}':r}]
    print json.dumps({'data':ports},sort_keys=True,indent=4,separators=(',',':'))

执行生成一个json文件，包含所有运行的redis实例端口

    [root@NH-SDO-OP-REDIS-42 zabbix_agentd.conf.d]# /opt/app/python27/bin/python  ../../bin/redis_port.py
    {
        "data":[
            {
                "{#REDISPORT}":"11111"
            },
            {
                "{#REDISPORT}":"11051"
            },
            {
                "{#REDISPORT}":"11211"
            },
            {
                "{#REDISPORT}":"11052"
            },
            {
                "{#REDISPORT}":"11053"
            },
            {
                "{#REDISPORT}":"11054"
            },
            {
                "{#REDISPORT}":"11311"
            }
       ]  
    }




**4. nginx :**  

首先nginx配置文件启用stub_status：  
`/opt/conf/nginx/vhost/default.outer`


