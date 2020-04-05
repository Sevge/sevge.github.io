---
title: 高可用业务集群实验
date: 2018-09-11 08:47:18
categories:
- Linux
---

### 准备工作
老惯例，SELINX和IPTABLES先清一下
```
setenforce 0
service firewalld stop
```
整体架构如下：
![架构图](http://imglf5.nosdn0.126.net/img/c09lVS9TR3YrUFoyU01OdGZmc2E3OEVQUDBYeU5HRWZ0a3RUMnNJTlVVeEZlYmp3bWN1UDBBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
### 配置最外层的防火墙（路由）
1. 先在虚拟机上添加一块网卡，采用桥接模式
2. 复制/etc/sysconfig/network-script/ifcfg-ens33一份到/etc/sysconfig/network-script/ifcfg-ens37
3. 记得修改其中的内容为ens37，UUID也修改为唯一
4. 配置SNAT，让内部服务器可以上网，DNAT将端口映射出去：
```
# 安装iptables防火墙
yum install -y iptables-services
# 对iptables进行初始化
iptables -F
iptables -t nat -F
# 打开系统的IP转发功能
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf 
# 不用重启,立即生效

# 配置iptables的NAT转发（重点）
iptables -t nat -A POSTROUTING -s 192.168.177.0/24 -j SNAT --to 192.168.0.177
iptables -t nat -A PREROUTING -d 192.168.0.177 -p tcp --dport 22 -j DNAT --to 192.168.177.2
iptables -t nat -A PREROUTING -d 192.168.0.177 -p tcp --dport 80 -j DNAT --to 192.168.177.101
iptables -t nat -A PREROUTING -d 192.168.0.177 -p tcp --dport 3306 -j DNAT --to 192.168.177.102
...

# 保存并启动
iptables-save > /etc/sysconfig/iptables
systemctl start iptables
systemctl enable iptables
```


### lvs + keepalived + nginx + gunicorn部署自己的django项目并实现高可用性

#### 环境
* LoadBalancer master：192.168.177.3
* LoadBalancer backup：192.168.177.4
* Web1：192.168.177.5
* Web2：192.168.177.6
* NFS：192.168.177.7
* vip：192.168.177.101
```
vi /etc/sysconfig/network-script/ifcfg-ens33
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.177.3
NETMASK=255.255.255.0
GATEWAY=192.168.177.1
DNS1=119.29.29.29
```
* 配置两台Director节点
```
yum install -y ipvsadm keepalived
echo 1 > /proc/sys/net/ipv4/ip_forward # 开启路由转发
ifconfig ens33 down
ifconfig ens33 192.168.177.101 broadcast 192.168.177.101 netmask 255.255.255.255 up
route add -host 192.168.177.101 dev ens33
ipvsadm -C
ipvsadm -A -t 192.168.177.101:80 -s wrr
ipvsadm -a -t 192.168.177.101 -r 192.168.177.5:80 -g -w 3
ipvsadm -a -t 192.168.177.101 -r 1 92.168.166.6 -g -w 3
====
/etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   # vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
   state MASTER
    interface ens33
    virtual_router_id 17
    priority 110
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123123
    }
    virtual_ipaddress {
        192.168.177.101
    }
}

virtual_server 192.168.177.101 80 {　　#设置虚拟服务器，需要指定虚拟ip和服务端口
    delay_loop 3　　　　#健康检查时间间隔
    lb_algo rr　　　　 #负载均衡调度算法
    lb_kind DR　　　　#负载均衡转发规则
    persistence_timeout 50　　　　#设置会话保持时间，对动态网页非常有用
    protocol TCP　　　　#指定转发协议类型，有TCP和UDP两种

    real_server 192.168.177.5 80 {　　#配置服务器节点1，需要指定real server的真实IP地址和端口
        weight 1　　　　#设置权重，数字越大权重越高
    	TCP_CHECK { 　　　　#realserver的状态监测设置部分单位秒
	connect_timeout 3　　　　#超时时间
	nb_get_retry 3　　　　　　#重试次数
	delay_before_retry 3　　　　#重试间隔
	 connect_port 80    　　#监测端口
    	}
}
    real_server 192.168.177.6 80 {
   	 weight 1
    	TCP_CHECK {
    	connect_timeout 3
    	nb_get_retry 3
    	delay_before_retry 3
    	connect_port 80
        }
    } 

}
serviced keepalived start
```
* 配置两台Real Server：
```
ifconfig lo 192.168.177.101 broadcast 192.168.177.101 netmask 255.255.255.255 up
route add -host 192.168.177.101 lo
echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
```
外面访问`192.168.0.177`，当出现如下结果即成功搭建好了lvs+nginx的高可用web服务。
![nginx](http://imglf3.nosdn0.126.net/img/c09lVS9TR3YrUFoyU01OdGZmc2E3MkN2YkFkWVBHT2VERVdDTjJuei9ka1A4RjNhZUIrYVd3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
#### 搭建NFS服务
##### 环境
* NFS服务器 192.168.177.7
* 客户端（web服务器）192.168.177.5/6

配置服务器192.168.177.7
```
# scp /tmp/ali_research root@192.168.177.7:/NFS/django_project
yum -y install nfs-utils rpcbind
mkdir /NFS/django_project
chmod 666 /NFS/django_project
echo '/NFS/djagno_peoject 192.168.177.0/24(rw,no_root_squash,no_all_squash,sync)' > /etc/exports
# 配置生效
exportfs -r
# 启动rpcbind、nfs服务
service rpcbind start
service nfs start
# 查阅NFS服务
showmount -e localhost
```
* 客户端配置 192.168.177.5/6
```
yum install -y nfs-utils
mkdir /django_project
showmount -e 192.168.177.7
# 挂载
mount -t nfs
192.168.177.7:/NFS/django_peoject /django_project proto=tcp -o nolock
# 查看挂载结果
df -h
```

#### 部署django项目
安装必要组件、修改django项目settings文件：
```
yum install python36
python36 -m ensurepip  安装pip3软件
pip3 install django==1.11  -i https://pypi.tuna.tsinghua.edu.cn/simple/
pip3 install django-ckeditor Pillow  -i https://pypi.tuna.tsinghua.edu.cn/simple/
pip3 install gunicorn

vi /django_project/ali_research/ali_research/settings.py
    DEBUG=False
    ALLOWED_HOSTS = ['*',]
    STATIC_URL = '/static/'
    #STATICFILES_DIRS = [os.path.join(BASE_DIR, "static"), ]
    STATIC_ROOT = os.path.join(BASE_DIR, 'static')
cd /django_project/ali_research
gunicorn -w 4 -b 127.0.0.1:8000 -D --access-logfile=/var/log/gunicorn.log ali_research.wsgi:application
ps aux|grep gunicorn  查看进程


vim /etc/nginx/nginx.conf
user root #用户需要改为root，否则报403
location / {
                proxy_pass http://127.0.0.1:8000;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

location /static/ {
        root /django_project/ali_research/;
}


```
#### 安装supervisor监控gunicorn和nginx服务
```
# 参照https://www.cnblogs.com/gjack/p/8076419.html
yum install wget 
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
pip install supervisor
echo_supervisord_conf > /etc/supervisor.conf   # 生成 supervisor 默认配置文件

vim /etc/supervisor.conf                       # 修改 supervisor 配置文件，在末尾添加进程管理：
[program:aliapp]
command=gunicorn -w 4 -b 127.0.0.1:8000 -D --access-logfile=/var/log/gunicorn.log ali_research.wsgi:application    ; supervisor启动命令
directory=/django_project/ali_research                                                 ; 项目的文件夹路径
startsecs=0                                                                             ; 启动时间
stopwaitsecs=0                                                                          ; 终止等待时间
autostart=True                                                                        ; 是否自动启动
autorestart=True                                                                       ; 是否自动重启
stdout_logfile=/var/log/supervisor_gunicorn.log                           ; log 日志
stderr_logfile=/var/log/supervisor_gunicorn.err                           ; 错误日志

[program:nginx]
command=service nginx start
startsecs=0
stopwaitsecs=0
autostart=True
autorestart=True
stdout_logfile=/var/log/supervisor_nginx.log
stderr_logfile=/var/log/supervisor_nginx.err
supervisor的基本使用命令

supervisord -c supervisor.conf                             通过配置文件启动supervisor
supervisorctl -c supervisor.conf status                    察看supervisor的状态
supervisorctl -c supervisor.conf reload                    重新载入 配置文件
supervisorctl -c supervisor.conf start [all]|[appname]     启动指定/所有 supervisor管理的程序进程
supervisorctl -c supervisor.conf stop [all]|[appname]      关闭指定/所有 supervisor管理的程序进程

若报错“Error: Another program is already listening on a port that one of our HTTP servers is configured to use. Shut this program down first before starting”
解决方法：
find / -name supervisor.sock
unlink /name/supervisor.sock
```
### MysqlRouter+Mariadb实现主从而复制和读写分离
* 环境
  * 192.168.177.8:Mysql Router
  * 192.168.177.9:Mariadb Master
  * 192.168.177.10:Mariadb Slave
  * 192.168.177.11:Mariadb Backup

先配置主从复制功能：
#### 192.168.177.9:Master
```
yum install -y mariadb mariadb-server

# 在my.cnf添加如下内容：
vi /etc/my.cnf
#binary log
server-id=1
log-bin

service mariadb restart 
mysql-->
show master status;
# 建立授权用户
grant replication slave on *.* to 'rep'@'192.168.177.%' identified by '123123';
flush privileges;
show grants for 'rep'@'192.168.177.%';

```
#### 192.168.177.10:Slave
```
yum install -y mariadb mariadb-server

# 在my.cnf添加如下内容：
vi /etc/my.cnf
server-id=2
log-bin

service mariadb restart
mysql -->
change master to 
master_host='192.168.177.9',
master_user='rep',
master_password='123123',
master_port=3306,
master_log_file='mariadb-bin.000001',
master_log_pos=245;

show slave status\G; # 注意Slave_IO_Running，Slave_SQL_Running为Yes即成功开启
stop slave
start slave
```
#### 192.168.177.11：Backup
```
vi /etc/my.cnf
server-id=3
log-bin

mysql -->
change master to 
master_host='192.168.177.9',
**master_delay=300,** # 延时5min，以防止误操作的备份
# 需要mariadb10.3.8版本以上或Mysql
    vi /etc/yum.repo.d/CentOS-MariaDB.repo:
    # MariaDB 10.3 CentOS repository list - created 2018-05-26 07:55 UTC
    # http://downloads.mariadb.org/mariadb/repositories/
    [mariadb]
    name = MariaDB
    #baseurl = http://yum.mariadb.org/10.3/centos7-amd64
    baseurl = https://mirrors.tuna.tsinghua.edu.cn/mariadb//mariadb-10.3.9/yum/rhel74-amd64/
    yum clean all
    yum makecache
    yum install mariadb-sever
master_user='rep',
master_password='123123',
master_port=3306,
master_log_file='mariadb-bin.000001',
master_log_pos=245;

start slave
```
#### 192.168.177.8：Router
```
wget https://dev.mysql.com/get/Downloads/MySQL-Router/mysql-router-8.0.11-1.el7.x86_64.rpm
rpm -ivh  mysql-router-8.0.11-1.el7.x86_64.rpm
vi /eetc/mysqlrouter.conf
# 添加如下内容
[routing:read_write]
bind_address = 192.168.177.8  #是mysql-router服务器的ip地址
bind_port = 7001
mode = read-write
destinations = 192.168.177.9:3306
max_connections = 65535
max_connect_errors = 100
client_connect_timeout = 9
 
[routing:read_only]
bind_address = 192.168.177.8
bind_port = 7002
mode = read-only
destinations = 192.168.177.10:3306
max_connections = 65535
max_connect_errors = 100
client_connect_timeout = 9

# BACKUP由于是备份，数据有延时，因此既不提供读也不提供写。

service mysqlrouter start
```
##### 此时可以测试

* [root@mysql-master ~]# mysql -h 192.168.177.8 -uwong -p'123123' -P 7001    # 写的测试
* [root@mysql-master ~]# mysql -h 192.168.177.8 -uwong -p'123123' -P 7002    # 读的测试
* 修改Master数据库的内容，Slave马上同步，而Backup不回立即同步。

### 使用Docker容器的Nginx代理两个域名
* 安装redis
```
yum install epel-release
yum install docker
service docker start
docker pull redis
docker images
docker run -d -p 6379:6379 --name redis docker.io/redis
```
* 安装三个nginx容器，其中一个用来做代理转发，另外两个充当web服务器。
```
# https://blog.csdn.net/u011164906/article/details/73135836?locationNum=5&fps=1
docker pull nginx
docker run -d --name web1 -p 8081:80 -v /root/docker_nginx/web1.html:/usr/share/nginx/html/index.html docker.io/nginx
docker run -d --name web2 -v /root/docker_nginx_conf/web2.html:/usr/share/nginx/html/index.html -p 8082:80 docker.io/nginx
docker run -d --name nginx_agent --link web1:nginxa --link web2:nginxb -v /root/docker_nginx_conf/nginx.conf:/etc/nginx/nginx.conf -p 8080:80 docker.io/nginx
#docker的对应顺序为：真实机：容器
```
使用命令`docker inspect -f '\{\{\.Name\}\} - \{\{range \.NetworkSettings\.Networks\}\}\{\{\.IPAddress\}\}\{\{end\}\}' $(docker ps -aq)
`可以看到各个容器名对应的IP地址：
* 172.17.0.2:d_redis
* 172.17.0.3:web1
* 172.17.0.4:web2
* 172.17.0.5:nginx_agent

接下来配置/root/docker_nginx/nginx.conf：
```
worker_processes  1;
events {
  worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;


   # include /etc/nginx/conf.d/*.conf;

  server
    {
      listen 80; # 在容器中监听的端口为80
      server_name www.wangtao.com;
      location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://nginxa; # 大坑--link的名称nginxa，链接另一个容器的端口80
            <!--#root@8f63042bbc29:/# cat /etc/hosts -->
            <!--127.0.0.1	localhost-->
            <!--::1	localhost ip6-localhost ip6-loopback-->
            <!--fe00::0	ip6-localnet-->
            <!--ff00::0	ip6-mcastprefix-->
            <!--ff02::1	ip6-allnodes-->
            <!--ff02::2	ip6-allrouters-->
            <!--172.17.0.3	nginxa eba80887d700 web1-->
            <!--172.17.0.4	nginxb 247ed0475876 web2-->
            <!--172.17.0.5	8f63042bbc29-->
      }
    }

  server
    {
      listen 80;
      server_name www.sevge.com;
      location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://nginxb;
      }
    }
}
```
注意容器间的连接如下：
![docker容器间的连接](http://imglf3.nosdn0.126.net/img/c09lVS9TR3YrUFoyU01OdGZmc2E3L01jNCtBc2FsYm9WL0xFM1ozaXVEWHMyeFZOdUZCNWFRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

修改windows机器里的hosts文件，修改web1、web2对应的html文件，运行docker容器(docker run -d -p 8081:80 -v /root/html/web1.html:/usr/share/nginx/html/index.html --name web1 docker.io/nginx、docerk run -d -p 8082:80 -v /root/html/web2.html:/usr/share/nginx/html/index.html --name web2 docker.io/nginx、docker run -d -p 8080:80 -v /root/nginx_conf:/etc/nginx/nginx.conf --name nginx_agent docker.io/nginx），分别访问可以看到：
![web1](http://imglf4.nosdn0.126.net/img/c09lVS9TR3YrUFoyU01OdGZmc2E3eEJCditvUGJZSXZPRm9MR2hrcmloeXVWZEpPWmFqQUV3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)
![web2](http://imglf3.nosdn0.126.net/img/c09lVS9TR3YrUFoyU01OdGZmc2E3MmdrL2J4ZGlsTEZoRzFRZExHZGRpeUViWVRaTFB2dmFRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

### 配置和使用Ansible
* 192.168.177.14
```
yum install epel-release
yum install ansible
yum install expect
# 所有机器配置免秘钥认证，编写expect脚本
===
cat for.sh
#!/bin/bash

for i in {1..13}
do
	expect ssh_key.sh 192.168.177.${i}
done
===
cat ssh_key.sh
#!/usr/bin/expect

set ip [lindex $argv 0]

spawn ssh-copy-id -i /root/.ssh/id_rsa.pub root@$ip
expect "(yes/no)?"
send "yes\n"
expect "password:"
send "123123\n"
expect eof

exit
===
# 所有机器免秘钥认证完成，接下来配置ansible的hosts文件,在末尾添加vi /etc/ansible/hosts:
[weblb]
192.168.177.3
192.168.177.4

[webserver]
192.168.177.5
192.168.177.6

[nfs]
192.168.177.7

[sshd]
192.168.177.2

[dblb]
192.168.177.8

[db]
192.168.177.9
192.168.177.10
192.168.177.11

[docker]
192.168.177.12

[zabbix]
192.168.177.13
===
# 配置playbook并执行
cat my_play.yaml
- hosts: all
  remote_user: root
  tasks:
    - name: allow host
      shell: echo sshd:192.168.177.2 >> /etc/hosts.allow
    - name: allow host2
      shell: echo sshd:192.168.177.14 >> /etc/hosts.allow
    - name: deny host
      shell: echo sshd:all >> /etc/hosts.deny
- hosts: sshd
  remote_user: root
  tasks:
    - name: allow host
      shell: echo sshd:all >> /etc/hosts.allow
    
===
ansible-playbook my_play.yaml

PLAY [all] ******************************************************************************

TASK [Gathering Facts] ******************************************************************
ok: [192.168.177.13]
ok: [192.168.177.5]
ok: [192.168.177.12]
ok: [192.168.177.6]
ok: [192.168.177.8]
ok: [192.168.177.7]
ok: [192.168.177.2]
ok: [192.168.177.3]
ok: [192.168.177.4]
ok: [192.168.177.11]
ok: [192.168.177.9]
ok: [192.168.177.10]

TASK [allow host] ***********************************************************************
changed: [192.168.177.13]
changed: [192.168.177.2]
changed: [192.168.177.8]
changed: [192.168.177.5]
changed: [192.168.177.7]
changed: [192.168.177.6]
changed: [192.168.177.12]
changed: [192.168.177.9]
changed: [192.168.177.10]
changed: [192.168.177.11]
changed: [192.168.177.3]
changed: [192.168.177.4]

TASK [deny host] ************************************************************************
changed: [192.168.177.8]
changed: [192.168.177.13]
changed: [192.168.177.7]
changed: [192.168.177.2]
changed: [192.168.177.5]
changed: [192.168.177.6]
changed: [192.168.177.12]
changed: [192.168.177.9]
changed: [192.168.177.10]
changed: [192.168.177.11]
changed: [192.168.177.3]
changed: [192.168.177.4]

PLAY RECAP ******************************************************************************
192.168.177.10             : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.11             : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.12             : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.13             : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.2              : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.3              : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.4              : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.5              : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.6              : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.7              : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.8              : ok=3    changed=2    unreachable=0    failed=0   
192.168.177.9              : ok=3    changed=2    unreachable=0    failed=0   

```
### 配置Zabbix监控所有服务器
* 192.168.177.13
```
https://www.cnblogs.com/clsn/p/7885990.html
#安装zabbix源、aliyun YUM源
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm

#安装zabbix 
yum install -y zabbix-server-mysql zabbix-web-mysql

#安装启动 mariadb数据库
yum install -y  mariadb-server
systemctl start mariadb.service

#创建数据库
mysql -e 'create database zabbix character set utf8 collate utf8_bin;'
mysql -e 'grant all privileges on zabbix.* to zabbix@localhost identified by "zabbix";'

#导入数据
zcat /usr/share/doc/zabbix-server-mysql-3.0.21/create.sql.gz|mysql -uzabbix -pzabbix zabbix

#配置zabbixserver连接mysql
sed -i.ori '115a DBPassword=zabbix' /etc/zabbix/zabbix_server.conf

#添加时区
sed -i.ori '18a php_value date.timezone  Asia/Shanghai' /etc/httpd/conf.d/zabbix.conf

#解决中文乱码
yum -y install wqy-microhei-fonts
\cp /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf

#启动服务
systemctl start zabbix-server

#修改/etc/httpd/conf/http.conf的监听端口为1234
service httpd restart

# 路由添加一条规则（192.168.177.1）
iptables -t nat -A PREROUTING -d 192.168.0.177 -p tcp --dport 1234 -j DNAT --to 192.168.177.13
# 此时windows访问192.168.0.177:1234/zabbix可以看到zabbix的setup界面
<!--[root@localhost ~]# cat for_hosts.sh -->
<!--for i in {3..14}-->
<!--do-->
<!--	expect allow.sh 192.168.177.${i}-->
<!--done-->
<!--[root@localhost ~]# cat allow.sh -->
<!--set ip [lindex $argv 0]-->

<!--spawn ssh $ip-->
<!--expect "*password:"-->
<!--send "123123\n"-->
<!--expect "*#"-->
<!--send "echo 'sshd:192.168.177.14' >> /etc/hosts.allow\n"-->
<!--expect "*#"-->
<!--send "exit\n"-->
<!--expect eof-->
<!--exit-->
<!--# expect和send后的内容必须使用双引号，单引号报错！！！-->
```
* 客户端配置
```
# 为了监听177网段所有主机，需要借助ansible（192.168.177.14）一步配置所有服务器
# 编写部署zabbix-agent的脚本
[root@localhost ~]# cat zabbix_client_set.sh 
#安装zabbix源、aliyu nYUM源
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm

#安装zabbix客户端
yum install zabbix-agent -y
sed -i 's/Server=127.0.0.1/Server=192.168.177.13/g' /etc/zabbix/zabbix_agentd.conf
systemctl start  zabbix-agent.service
# 将这个脚本发送给各个机器
[root@localhost ~]# cat cp_sh.sh 
for i in {1..14}
do
	scp /root/zabbix_client_set.sh 192.168.177.${i}:/root
done
# ansible的playbook：
[root@localhost ~]# cat zabbix_client_set.yaml 
- hosts: all
  remote_user: root
  tasks:
    - name: zabbix agent
      shell: bash zabbix_client_set.sh

ansible-playbook zabbox_client-set.yaml
```
* 回到zabbix服务器，配置zabbix服务

登录这里，用户名是Admin（大写A），密码为zabbix。
![zabbix_login](http://imglf6.nosdn0.126.net/img/c09lVS9TR3YrUFoyU01OdGZmc2E3N1J6R3ZRdkQ0NWFEQU00SjFrRjhFdUhVOHlHc2hDODdBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)

接着添加主机，为主机添加模板，那里不会点哪里...
```
<!--zabbix_server中agent配置文件的Server地址为127.0.0.1-->
<!--看到状态和可用性亮绿灯即成功 -->
```
![zabbix_status](http://imglf6.nosdn0.126.net/img/c09lVS9TR3YrUFoyU01OdGZmc2E3NnllTkNGTGtKTW9BWlRpWm83UXZNYlJYUFo4eVFhQklnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)