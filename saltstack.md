#salt学习笔记

[官方文档请点这里](http://docs.saltstack.cn/zh_CN/latest/topics/tutorials/walkthrough.html#setting-up-the-salt-master)  
[salt code example](https://github.com/craig5/salt-essentials-utils "salt code example")
###安装

**Debian/Ubuntu:**  

1.添加源仓库:

    sudo apt-get install software-properties-common
    sudo apt-get install python-software-properties
    sudo add-apt-repository ppa:saltstack/salt
    sudo apt-get update

    sudo apt-get install salt-master
    sudo apt-get install salt-minion
    sudo apt-get install salt-syndic

**Centos/Rhel :**

    rpm -Uvh http://mirror.pnl.gov/epel/5/i386/epel-release-5-4.noarch.rpm
    rpm -Uvh http://ftp.linux.ncsu.edu/pub/epel/6/i386/epel-release-6-8.noarch.rpm

    yum install salt-master
    yum install salt-minion


