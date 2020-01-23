---
title: Cenotos7搭建部署OSSEC-server+agent
copyright: true
categories: 环境搭建
tags:
  - OSSEC
  - Centos7
abbrlink: d1ee79fb
date: 2020-01-07 10:57:39
---

<blockquote class="blockquote-center">大鹏一日同风起，扶摇直上九万里
</blockquote>

　　OSSEC是一款开源的基于主机的入侵检测系统，可以简称为HIDS。它具备日志分析，文件完整性检查，策略监控，rootkit检测，实时报警以及联动响应等功能。它支持多种操作系统：Linux、Windows、MacOS、Solaris、HP-UX、AIX。属于企业安全之利器。

<!-- more -->

### OSSEC简介

　　OSSEC是一款开源的多平台的入侵检测系统，可以运行于Windows、Linux、OpenBSD/FreeBSD、以及 MacOS等操作系统中。主要功能有日志分析、完整性检查、rootkit检测、基于时间的警报和主动响应。除了具有入侵检测系统功能外，它还一般被用在SEM/SIM（安全事件管理SEM： Security Event Management)/（安全信息管理SIM：SecurityInformation Management）解决方案中。因其强大的日志分析引擎，ISP（网络服务提供商Internet service provider）、大学和数据中心用其监控和分析他们的防火墙、入侵检测系统、网页服务和验证等产生的日志。



### 前提环境准备

　　首先我们安装需要用到的关联库和软件，由于我们最终是需要把日志导入到MySQL中进行分析，以及需要通过web程序对报警结果进行展示，同时需要把本机当做SMTP，所以需要在本机安装MySQL、Apache和sendmail服务。在当前的终端中执行如下命令：

```
yum install wget gcc make mysql mysql-server mysql-devel httpd php php-mysql sendmail
```

启动httpd、mysql、sendmail服务

```
for i in {httpd,mysqld,sendmail}; do service $i restart; done
```


创建数据库方面后面的安装配置，连接到本机的MySQL，然后执行

```
mysql -uroot -p
mysql> create database ossec;
mysql> CREATE USER 'ossec'@'localhost';
mysql> grant INSERT,SELECT,UPDATE,CREATE,DELETE,EXECUTE on ossec.* to ossec@localhost;
mysql> set password for ossec@localhost =PASSWORD('ossec');
mysql> flush privileges;
mysql> exit
```


　　这里使用的是比较简单的密码，MySQL5.6.6版本之后增加了密码强度验证插件validate_password，相关参数设置的较为严格。使用了该插件会检查设置的密码是否符合当前设置的强度规则，若不满足则拒绝设置。所以需要修改密码强度的验证机制：

#### 1） 查看mysql全局参数配置

这里与mysql的validate_password_policy的值有关

查看一下msyql密码相关的几个全局参数：

```
mysql> select @@validate_password_policy;
+----------------------------+
| @@validate_password_policy |
+----------------------------+
| MEDIUM                     |
+----------------------------+
1 row in set (0.00 sec)


mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
6 rows in set (0.08 sec)
```

#### 2）参数解释

`validate_password_dictionary_file`

插件用于验证密码强度的字典文件路径



`validate_password_length`

密码最小长度，参数默认为8，它有最小值的限制，最小值为：validate_password_number_count + validate_password_special_char_count + (2 * validate_password_mixed_case_count)



`validate_password_mixed_case_count`

密码至少要包含的小写字母个数和大写字母个数



`validate_password_number_count`

密码至少要包含的数字个数



`validate_password_policy`

密码强度检查等级，0/LOW、1/MEDIUM、2/STRONG。有以下取值：

Policy                 Tests Performed                                                                            

0 or LOW               Length                                                                                      

1 or MEDIUM         Length; numeric, lowercase/uppercase, and special characters

2 or STRONG        Length; numeric, lowercase/uppercase, and special characters; dictionary file  

默认是1，即MEDIUM，所以刚开始设置的密码必须符合长度，且必须含有数字，小写或大写字母，特殊字符

 

`validate_password_special_char_count`

密码至少要包含的特殊字符数



#### 3）修改mysql参数配置

```
mysql> set global validate_password_policy=0;
Query OK, 0 rows affected (0.05 sec)

mysql> 
mysql> 
mysql> set global validate_password_mixed_case_count=0;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password_number_count=0;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password_special_char_count=0;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password_length=0;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password_dictionary_file    |       |
| validate_password_length             | 0     |
| validate_password_mixed_case_count   | 0     |
| validate_password_number_count       | 0     |
| validate_password_policy             | LOW   |
| validate_password_special_char_count | 0     |
+--------------------------------------+-------+
6 rows in set (0.00 sec)
```



#### 4）修改简单密码

```
mysql> SET PASSWORD FOR 'ossec'@'localhost' = PASSWORD('ossec');
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

### ossec-server安装

在官网下载目前最稳定的版本v3.3.0(https://github.com/ossec/ossec-hids/archive/3.3.0.tar.gz)

![](https://img-blog.csdnimg.cn/20200106190548674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

```
wget https://github.com/ossec/ossec-hids/archive/2.8.1.tar.gz
tar zxf 2.8.1.tar.gz
cd ossec-hids-2.8.1
```


为了使ossec支持mysql，这里还需要在安装前执行一下命令：

```
cd src; make setdb; cd ..
```


　　（一开始装的是目前最稳定的3.3.0版本，但是make不支持mysql，一直报错：make: *** 没有规则可以创建目标“setdb”。 停止。查阅文档发现从3.0.0版本开始，编译方式不一样，也参考过使用make TARGET=server DATABASE=mysql install，但是还是会提示OSSEC not compiled with support for 'mysql'，只能用回2.8.1的版本）

　　进入安装步骤，执行install.sh脚本，同时按照下面的信息进行填写，红色部分是我们需要输入的，其余部分按回车继续即可：

![](https://img-blog.csdnimg.cn/20200106190711148.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

1- 您希望哪一种安装 (server, agent, local or help)? `server`

- 选择了 Server 类型的安装.

2- 正在初始化安装环境.

请选择 OSSEC HIDS 的安装路径 [/var/ossec]:`/var/ossec`

- OSSEC HIDS 将安装在 /var/ossec .

3- 正在配置 OSSEC HIDS.

3.1- 您希望收到e-mail告警吗? (y/n) [y]: `y`

请输入您的 e-mail 地址?  ****@***.com

- 我们找到您的 SMTP 服务器为: alt1.gmail-smtp-in.l.google.com.

- 您希望使用它吗? (y/n) [y]: `n`

- 请输入您的 SMTP 服务器IP或主机名 ? `127.0.0.1`

3.2- 您希望运行系统完整性检测模块吗? (y/n) [y]: `y`

- 系统完整性检测模块将被部署.

3.3- 您希望运行 rootkit检测吗? (y/n) [y]: `y`

- rootkit检测将被部署.

3.4- 关联响应允许您在分析已接收事件的基础上执行一个已定义的命令.

例如,你可以阻止某个IP地址的访问或禁止某个用户的访问权限.

更多的信息,您可以访问:

http://www.ossec.net/en/manual.html#active-response

- 您希望开启联动(active response)功能吗? (y/n) [y]: y
  - 关联响应已开启
- 默认情况下, 我们开启了主机拒绝和防火墙拒绝两种响应.
  第一种情况将添加一个主机到 /etc/hosts.deny.

第二种情况将在iptables(linux)或ipfilter(Solaris,

FreeBSD 或 NetBSD）中拒绝该主机的访问.

- 该功能可以用以阻止 SSHD 暴力攻击, 端口扫描和其他

一些形式的攻击. 同样你也可以将他们添加到其他地方,

例如将他们添加为 snort 的事件.

- 您希望开启防火墙联动(firewall-drop)功能吗? (y/n) [y]:  `y`
  - 防火墙联动(firewall-drop)当事件级别 >= 6 时被启动

- 联动功能默认的白名单是:
  - 192.168.30.1

- 您希望添加更多的IP到白名单吗? (y/n)? [n]:  `y`

- 请输入IP (用空格进行分隔): `192.168.30.130`

3.5- 您希望接收远程机器syslog吗 (port 514 udp)? (y/n) [y]: `y`

- 远程机器syslog将被接收.

3.6- 设置配置文件以分析一下日志:

-- /var/log/messages

-- /var/log/secure

-- /var/log/maillog

-如果你希望监控其他文件, 只需要在配置文件ossec.conf中

添加新的一项.

任何关于配置的疑问您都可以在 http://www.ossec.net 找到答案.

--- 按 ENTER 以继续 ---

4- 正在安装系统

-正在运行Makefile

INFO: Little endian set.

…………省略编译输出…………

- 系统类型是 Redhat Linux.
  - 修改启动脚本使 OSSEC HIDS 在系统启动时自动运行
  - 已正确完成系统配置.
  - 要启动 OSSEC HIDS:

/var/ossec/bin/ossec-control start

- 要停止 OSSEC HIDS:

/var/ossec/bin/ossec-control stop

- 要查看或修改系统配置,请编辑 /var/ossec/etc/ossec.conf

感谢使用 OSSEC HIDS.

如果您有任何疑问,建议或您找到任何bug,

[email protected] 或邮件列表 [email protected] 联系我们.

( http://www.ossec.net/en/mailing_lists.html ).

您可以在　http://www.ossec.net 获得更多信息

--- 请按　ENTER 结束安装 (下面可能有更多信息). ---

直到碰到上面内容，说明安装完成。

![](https://img-blog.csdnimg.cn/20200106190745336.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### ossec-server配置

安装好服务端之后，还需要配置，执行下面命令启用数据库支持：

```
[root@Centos ossec-hids-2.8.1]# /var/ossec/bin/ossec-control enable database
```


然后导入MySQL表结构到MySQL中：

```
[root@Centos ossec-hids-2.8.1]# mysql -u ossec -p ossec < ./src/os_dbd/mysql.schema
```


修改部分配置文件的权限，否则会启动服务失败：

```
[root@Centos ossec-hids-2.8.1]# chmod u+w /var/ossec/etc/ossec.conf
```


编辑ossec.conf文件，在ossec_config文件中添加mysql配置

```
<ossec_config>
  <database_output>
    <hostname>192.168.30.130</hostname>
    <username>ossec</username>
    <password>ossec</password>
    <database>ossec</database>
    <type>mysql</type>
  </database_output>
</ossec_config>
```


　　在前面的安装过程中支持接受远程机器的syslog，所以我们还需要对ossec.conf文件中的syslog部分进行配置，修改ossec.conf文件，按照下面的内容进行修改，把我们的网段全添加进去：

```
<remote>
  <connection>syslog</connection>
  <allowed-ips>192.168.0.0/16</allowed-ips>
</remote>
```


重启ossec服务进行生效

```
[root@Centos ossec-hids-2.8.1]# /var/ossec/bin/ossec-control restart
```

添加ossec客户端并导出Key

在服务器上添加客户端，执行如下命令，按照提示进行输入，红色部分是我们输入的：

```
[root@Centos ossec-hids-2.8.1]# /var/ossec/bin/manage_agents
```

![](https://img-blog.csdnimg.cn/2020010619095649.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

Key的作用是在客户端中导入并使得服务端与客户端达到联动的效果，这里记得把密钥复制一下保存起来。

查看ossec服务端的状态

```
/var/ossec/bin/agent_control -lc
```

### ossec-agent安装

这里安装的方式跟上面server安装方式是一样的，然后执行./install.sh

![](https://img-blog.csdnimg.cn/20200106191048165.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200106191058923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

稍等一会就会看到安装成功的提示

![](https://img-blog.csdnimg.cn/20200106191124181.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### ossec-agent配置

#### linux

把刚刚生成的key导入到客户端中

```
root@Ubuntu:~/ossec-hids-2.8.1# ./bin/manage_agents
```


启动客户端

```
root@Ubuntu:~/ossec-hids-2.8.1# /var/ossec/bin/ossec-control start
```

![](https://img-blog.csdnimg.cn/20200106191226605.png)

#### windows

下载客户端exe: 

https://bintray.com/artifact/download/ossec/ossec-hids/ossec-agent-win64-2.8.1.exe 

或者

https://bintray.com/artifact/download/ossec/ossec-hids/ossec-agent-win32-2.8.1.exe 

然后默认安装 

注意：运行时要以管理员权限运行，然后粘贴你复制下来的秘钥

![](https://img-blog.csdnimg.cn/20200106191258974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

最后别忘了运行一下

![](https://img-blog.csdnimg.cn/20200106191312109.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### 安装第三方的web界面（ossec-wui 或 analogi）

```
wget -O /var/www/html/ossec-wui-0.9.tar.gz https://github.com/ossec/ossec-wui/archive/0.9.tar.gz
tar -xzf ossec-wui-0.9.tar.gz
mv ossec-wui-0.9 ossec
cd ossec
./setup.sh
```

![](https://img-blog.csdnimg.cn/20200106191326211.png)

对ossec.config进行配置，添加虚拟目录

```
vim /etc/httpd/conf.d/ossec.conf
```

```
Alias ossec/ "/var/www/html/ossec/"
<Directory "/var/www/html/ossec/">
AuthName "OSSEC AUTH"
Require valid-user
AuthType Basic
AuthUserFile /var/www/html/ossec/.htpasswd
</Directory>
```

重启apache

```
service httpd restart
```


重启完就可以看到web的界面了

![](https://img-blog.csdnimg.cn/20200106191337956.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

这里一开始遇到了报错，提示ossec目录没有权限，别忘了需要对ossec授权

```
chmod -R 777 /var/ossec/
```

