#zabbix 笔记

###一、搭建手册

防火墙规划：

    iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
    iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
    iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 10051 -j ACCEPT
    iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 10050 -j ACCEPT
    #iptables -A INPUT -m state --state new -m tcp -p tcp --sport 10050 -j ACCEPT
    
>10050是agent端口，agent采用被动方式，server主动连接agent的10050端口  
>10051是server端口，agent采用主动或trapper方式，会连接server的10051端口

下载依赖包和软件

     wget http://sourceforge.net/projects/zabbix/files/ZABBIX%20Latest%20Stable/2.2.10/zabbix-2.2.10.tar.gz
     yum install gcc gcc-c++ make cmake
     yum install net-snmp-devel php-mysql mysql-server mysql-devel  libcurl libcurl-devel libxml2 libxml2-devel
     yum install php php-bcmath php-mbstring php-net-socket php-gd php-xml  php5-dom php-xmlwriter php-xmlreader php-ctype  php-session  php-gettext php-fpm  
	 yum install java-1.7.0-openjdk.x86_64 java-1.7.0-openjdk-devel.x86_64

创建用户和用户组：  

    groupadd zabbix 
    useradd -s /sbin/nologin -g zabbix zabbix

创建zabbix数据库，这里选用mysql：  

    create database zabbix character set utf8 collate utf8_bin;
    grant all on zabbix.* to zabbix@'localhost' identified by 'zabbix'; 
	flush privileges;

导入模板到zabbix库里：  

    cd /tmp/zabbix-2.0.6/database/
    mysql -uzabbix -pzabbix  zabbix <mysql/schema.sql 
    mysql -uzabbix -pzabbix  zabbix <mysql/images.sql
    mysql -uzabbix -pzabbix  zabbix <mysql/data.sql
>如果安装proxy只需要导入schema.sql

编译安装zabbix_master

    ./configure --prefix=/usr/local/zabbix --enable-server --enable-agent --enable-proxy --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2 --enable-java --with-ssh2
	make install 
安装zabbix_agent:

	./configure --prefix=/usr/local/zabbix  --enable-agent  --enable-java

开机自启设置：

    cp misc/init.d/fedora/core/zabbix_* /etc/init.d/
    sed -i 's/BASEDIR=\/usr\/local/BASEDIR=\/usr\/local\/zabbix/' /etc/init.d/zabbix_server
    sed -i 's/BASEDIR=\/usr\/local/BASEDIR=\/usr\/local\/zabbix/' /etc/init.d/zabbix_agentd
    chmod +x /etc/init.d/zabbix_server
    chmod +x /etc/init.d/zabbix_agentd
    chkconfig zabbix_server on
    chkconfig zabbix_agentd on


优化php配置：
    
    sed -i 's/^\(.*\)date.timezone =.*$/date.timezone = Asia\/Shanghai/g' /etc/php.ini
    sed -i 's/^\(.*\)post_max_size =.*$/post_max_size = 16M/g' /etc/php.ini
    sed -i 's/^\(.*\)max_execution_time =.*$/max_execution_time = 300/g'  /etc/php.ini
    sed -i 's/^\(.*\)max_input_time =.*$/max_input_time = 300/g' /etc/php.ini

>这两个看自己需求  
>memory_limit=128M  
mbstring.func_overload=2 


安装nginx：

    rpm -ivh http://nginx.org/packages/rhel/6/noarch/RPMS/nginx-release-rhel-6-0.el6.ngx.noarch.rpm
    yum install nginx
    vim /etc/nginx/conf.d/default.conf 



    server {
    listen   80;
    server_name  localhost;  
    charset utf-8;   
    
    root   /data/site/www/zabbix;
    index  index.php index.html index.htm;
    
    location ~ \.php {
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include fastcgi_params;
    }
    }
    
     mkdir -p /data/site/www/zabbix
     
     cp -rp ~/zabbix-2.2.10/frontends/php/* /data/site/www/zabbix/


###ztree
[ztree下载地址](https://github.com/spide4k/zatree/tree/master/zabbix-2.2.x)

1.下载文件：  

    git clone https://github.com/spide4k/zatree.git zatree

2.复制相关文件：

    假如zabbix web目录位置在/var/www/zabbix,定义zabbix目录
    
    ZABBIX_PATH=/var/www/zabbix
    
    复制相关文件和目录
    
    cp -rf zatree/zabbix-2.2.x $ZABBIX_PATH/zatree
    
    cd $ZABBIX_PATH/zatree/addfile
    
    cp -f CLineGraphDraw_Zabbix.php CGraphDraw_Zabbix.php CImageTextTable_Zabbix.php $ZABBIX_PATH/include/classes/graphdraw/
    
    cp -f zabbix.php zabbix_chart.php $ZABBIX_PATH/
    
    cp -f CItemValue.php $ZABBIX_PATH/api/classes/
    
    cp -f menu.inc.php $ZABBIX_PATH/include/
    
    cp -f main.js $ZABBIX_PATH/js/
    
    cp -f API.php $ZABBIX_PATH/include/classes/api/
    
    3：支持web interface,修改配置文件
    
    vi $ZABBIX_PATH/zatree/zabbix_config.php
    
    'user'=>'xxx', //web登陆的用户名
    
    'passowrd'=>'xxx', //web登陆的密码



ztree github项目：[ztree](https://github.com/spide4k/zatree/tree/master/zabbix-2.2.x)
