---
layout: post
title: 快速安装MySQL与为Lapis修改mysql字符集
description: 快速安装MySQL与为Lapis修改mysql字符集
date:   2016-09-17 22:50:18 +0800 
tags: [mysql,lapis]
categories: [topic,mysql]
---


## 快速安装MySQL



下载安装Yum源

```
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```
安装的MySQL与服务器端
```
yum -y install mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql-community-server
```

启动关闭 MySQL服务
```
systemctl start  mysqld.service
systemctl status mysqld.service
```

修改root密码
```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new password';
```


## 修改mysql字符集

第一种方式：

```
show variables like '%char%';  
set character_set_connection=gbk;
set character_set_client=gbk;
set character_set_results=gbk;
```

这种情况，在Mysql重新起动之后，值还会恢复到原先值。

第二种方式：
my.ini

```
[mysqld]
#default-character-set = utf8
init-connect='SET NAMES utf8'
character-set-server = utf8

[mysql]
default-character-set = utf8

```

```
SET NAMES utf8
```

它相当于下面的三句指令：
```
SET character_set_client = utf8;
SET character_set_results = utf8;
SET character_set_connection = utf8;
```



这种方式在Mysql得起之后，会安装my.ini中的设置重新设定。
特别是Lapis框架，要从中文数据库中取数据，必须设置中文字符集。