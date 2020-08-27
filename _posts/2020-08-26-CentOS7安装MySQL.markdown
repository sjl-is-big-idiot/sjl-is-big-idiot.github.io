---
layout: single
title: "[转载]CentOS7安装MySQL"
---

原文链接 [centos7安装mysql（完整）](https://www.cnblogs.com/lzhdonald/p/12511998.html)
<br>
原文链接 [Linux下安装mysql-5.7.24](https://www.cnblogs.com/lzhdonald/p/12511998.htm://www.jianshu.com/p/276d59cbc529)
# 1.安装前准备
## 1.1 检查是否安装过MySQL,执行命令
```py
rmp -qa |grep mysql
```
检查是否安装有系统自带的mariadb-lib
```py
rpm - qa |grep mariadb
```
如果有的话，将这二者都卸载
```py
rpm -e --nodeps mysql-libs-5.1.73-5.el6_6.x86_64
rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
```
卸载完成后，执行上面的命令，查看是否删除
## 1.2 查询所有Mysql对应的文件夹
```py
[root@localhost /]# whereis mysql
mysql: /usr/bin/mysql /usr/include/mysql
[root@localhost lib]# find / -name mysql
/data/mysql
/data/mysql/mysql
```
删除相关目录及文件
```py
[root@localhost /]#  rm -rf /usr/bin/mysql /usr/include/mysql /data/mysql
/data/mysql/mysql 
```
验证是否删除完毕
```py
[root@localhost /]# whereis mysql
mysql:
[root@localhost /]# find / -name mysql
[root@localhost /]# 
```
## 1.3 检查mysql用户组和用户是否存在，如果没有，则创建
```py
[root@localhost /]# cat /etc/group | grep mysql
[root@localhost /]# cat /etc/passwd |grep mysql
[root@localhost /]# groupadd mysql
[root@localhost /]# useradd -r -g mysql mysql
[root@localhost /]# 
```

## 1.4 下载安装包
[官网5.7版本](https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar)
[官网其他版本](https://dev.mysql.com/downloads/mysql/)

当然可以直接在CentOS7系统中使用`wget`或`curl`命令下载，也可以在Windows系统中下载好之后使用xftp上传到CentOS7
中。

# 2. 解压安装包并安装
## 2.1 执行解压命令
```py
#看是tar包还是tar.gz包
tar xvf mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar
tar zxvf mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar.gz
```
解压之后，是如下这些rpm包。
```py
mysql-community-client-5.7.29-1.el7.x86_64.rpm
mysql-community-common-5.7.29-1.el7.x86_64.rpm
mysql-community-devel-5.7.29-1.el7.x86_64.rpm
mysql-community-embedded-5.7.29-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.29-1.el7.x86_64.rpm
mysql-community-embedded-devel-5.7.29-1.el7.x86_64.rpm
mysql-community-libs-5.7.29-1.el7.x86_64.rpm
mysql-community-libs-compat-5.7.29-1.el7.x86_64.rpm
mysql-community-server-5.7.29-1.el7.x86_64.rpm
mysql-community-test-5.7.29-1.el7.x86_64.rpm
```
解压完成后，将解压的目录移动到/opt/下，并将文件夹名称修改为mysql。

>如果/opt/下已经存在mysql，请将已存在mysql文件修改为其他名称，否则后续步骤可能无法正确进行。

更改mysql目录下所有目录及文件所属的用户组和用户，及权限
```py
[root@localhost /]# chown -R mysql:mysql /opt/mysql
[root@localhost /]# chmod -R 755 /opt/mysql
```
严格按照顺序安装：
 1.mysql-community-common-5.7.29-1.el7.x86_64.rpm
 2.mysql-community-libs-5.7.29-1.el7.x86_64.rpm 
 3.mysql-community-client-5.7.29-1.el7.x86_64.rpm
 4.mysql-community-server-5.7.29-1.el7.x86_64.rpm

如果安装过程中出现这个错误就在后面添加 --force --nodeps，这可能是由于yum安装了旧版本的GPG keys造成的
# 3.配置数据库
打开`/etc/my.cnf`
```py
vim /etc/my.cnf
```
在该文件中添加下面这三行。第一行表示：跳过登陆验证
```py
skip-grant-tables
character_set_server=utf8
init_connect='SET NAMES utf8'
```
# 4. 启动MySQL服务
设置开机启动
```py
systemctl start mysqld.service
```
启动mysql
```py
mysql
```
# 5.设置密码和开启远程登陆
## 5.1 设置密码
```sql
update mysql.user set authentication_string=password('123456') where user='root';
```
刷新权限
```sql
flush privileges;
```
退出mysql并停止mysql服务
```py
systemctl stop mysqld.service
```
编辑my.cnf配置文件将：skip-grant-tables这一行注释掉
重启mysql服务
```py
systemctl start mysqld.service
```
再次登录mysql
```py
mysql -uroot -p123456
```
如果输入其他命令出错，再重设密码
```py
set password=password('123456');
```
## 5.2 设置密码策略（这步可以跳过）
*如果想要设置简单一点的密码就要设置密码策略，否则设置简单的密码会出错*

查看密码策略
```py
 SHOW VARIABLES LIKE 'validate_password%';
 ```

 1. validate_password_length  固定密码的总长度；
 2. validate_password_dictionary_file 指定密码验证的文件路径；
 3. validate_password_mixed_case_count 整个密码中至少要包含大/小写字母的总个数；
 4. validate_password_number_count  整个密码中至少要包含阿拉伯数字的个数；
 5. validate_password_policy 指定密码的强度验证等级，默认为 MEDIUM；

设置密码的验证强度等级，设置validate_password_policy的全局参数为LOW
```py
set global validate_password_policy=LOW;
```
只要设置密码的长度小于 3 ，都将自动设值为 4
```py
set global validate_password_length=4;
```
## 5.3 开放3306端口
```py
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```
重启防火墙
```py
firewall-cmd --reload
```
## 5.4 开启远程登陆
```py
grant all privileges on *.* to 'root'@'%' identified by '123123' with grant option;
```
 by后面的就是远程登录密码，远程登录密码可以和用户密码不一样。



