
# zabbix
## 环境

|系统| 角色|ip地址|
|rocky9|zabbix-server|192.168.153.129|
|centos7|zabbix-agent|192.168.153.130|


## 安装zabbix LTS6.0
### 安装zabbix服务端
* 修改rocky国内源
```shell
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/rocky-extras.repo \
    /etc/yum.repos.d/rocky.repo
```
* 添加zabbix软件源
```shell
 rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/9/x86_64/zabbix-release-6.0-4.el9.noarch.rpm
```
* 安装zabbix服务端喝前端， 前端发布采取nginx
```shell
dnf install zabbix-server-mysql zabbix-web-mysql zabbix-nginx-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent
```
* 安装mariadb数据库
```shell
dnf install mariadb-server mariadb -y
```
* 启动数据库
```shell
systemctl start mariadb && sudo systemctl enable mariadb
```
* 创建数据库以及数据表
```shell
# mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'zabbix';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;
```
* 导入zabix的相关数据
```shell
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```
* 配置Zabbix server数据库
编辑配置文件 /etc/zabbix/zabbix_server.conf
```shell
DBPassword=zabbix
```
* 配置Zabbix前端PHP
编辑配置文件 /etc/nginx/conf.d/zabbix.conf uncomment and set 'listen' and 'server_name' directives.
```shell
listen 8080;
server_name example.com;
```
*  启动Zabbix server和agent进程
```shell
systemctl enable zabbix-server zabbix-agent nginx php-fpm  --now
```

### 配置zabbix-agent
* 添加zabbix软件源
```shell
 rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/9/x86_64/zabbix-release-6.0-4.el9.noarch.rpm
```
* 安装agent2
```shell
 dnf install zabbix-agent2 zabbix-agent2-plugin-*
```
* 修改agent配置文件
打开/etc/zabbix/agent2.conf, 将server 以及ServerActive地址指向zabbix服务端
* 重启zagent
```shell
systemctl enable zabbix-agent2 --now
```

### web页面添加主机
 略。
### 安装常见问题 
* rocky9 在web页面卡在 php gd的解决方案
```shell
dnf install gd-devel -y
```

### 测试获取数据

```shell
[root@rocky9 ~]# zabbix_get  -s 192.168.153.130 -k "system.hostname"
centos7
```
通过zabbix_get成功获取到agent的主机名
