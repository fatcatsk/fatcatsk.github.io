#CentOS7下安装Zaibbix3.0
[TOC]
##环境：
>**系统：**CentOS7
>**环境：**
           LAMP：Server version: Apache/2.4.6 (CentOS)，
           Server version:  5.7.19 MySQL Community Server (GPL)
           PHP 7.0.22 (cli) (built: Aug  9 2017 18:23:24) ( NTS )


##一、环境准备
###1、关闭selinux（server&agent机都必须要）
```powershell
[root@master ~]# setenforce 0

此命令只能临时关闭selinux
```
```powershell
[root@master ~]# sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
or
vi /etc/selinux/config

将selinux的参数改为“disabled”，这可以永久关闭selinux
```

###2、添加必要的软件
```powershell
[root@master ~]# yum install epel-release.noarch wget vim gcc gcc-c++ lsof chrony tree nmap unzip rsync -y
[root@master ~]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
```



##二、安装zabbix
###1、server机安装zabbix
```powershell
[root@centos-1 html]# rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
[root@centos-1 html]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
[root@centos-1 html]# yum install zabbix-server-mysql zabbix-web-mysql zabbix-get
```

###2、agent机安装zabbix
```powershell
[root@centos-1 html]# rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
[root@centos-1 html]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
[root@centos-1 html]# yum install zabbix-agent
```


##三、server机数据库上创建账号
```sql
mysql> CREATE DATABASE zabbix DEFAULT CHARACTER SET utf8 COLLATE utf8_bin;    <---创建数据库并制定默认编码为utf8，防止乱码
Query OK, 1 row affected (0.00 sec)

mysql> GRANT ALL ON zabbix.* TO 'zabbix'@'%' IDENTIFIED BY '775120@Zabbix';     <----为zabbix创建账号并授权
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> flush privileges;      <-----刷新
Query OK, 0 rows affected (0.01 sec)
```


##四、修改zabbix配置参数
###1、修改server机上的参数
```powershell
[root@centos-1 lcr]# vim /etc/zabbix/zabbix_server.conf
```

>**将下面几个参数修改为上一小节创建的数据及账号**
>DBHost=localhost              ##默认即可，除非数据库不在server机上
>DBName=zabbix                 ##数据库(database)名称
>DBUser=zabbix                 ##用户名user
>DBPassword=775120@Lai         ##密码password




###2、server机修改默认时区
```powershell
[root@centos-1 lcr]# vim /etc/httpd/conf.d/zabbix.conf
```
>**如下修改，注意删除注释符**
>php_value date.timezone Asia/Chongqing



###3、修改agent机上的参数
```powershell
[root@centos-1 lcr]# vim /etc/zabbix/zabbix_server.conf
```

>**修改如下几项参数**
Server=zabbix server ip                 <---server机的IP
ServerActive=zabbix server ip        
Hostname=本机Ip #不要用127.0.0.1
ListenPort=10050



##五、启动zabbix
###1、server机上
```powershell
[root@centos-1 ~]# systemctl restart httpd
[root@centos-1 ~]# systemctl start zabbix-server
```

###2、agent机上
```powershell
[root@centos-1 ~]# systemctl start zabbix-agent
```


##六、配置zabbix

>**server机上访问：**http://本机ip/zabbix



###1、启动界面
![启动界面](./zabbix启动.jpg)

###2、检测组件状况
![Alt text](./zabbix状态.jpg)

###3、配置zabbix数据库信息
![Alt text](./zabbix数据库配置.jpg)

>**至此安装配置完成**


##七、添加agent机
###1、进入添加页面
![Alt text](./agent1.png)


###2、填写IP&port
>可见的名称：页面显示的名称
>群租：将主机归入某个群组
>填写IP地址，端口，默认为10050
![Alt text](./agent2.png)




###3、选择模板
>输入“Linux”即可选择该模板，如果不是linux主机，酌情选择别的模板
![Alt text](./agent3.png)



###4、查看添加情况
![Alt text](./agent4.png)
