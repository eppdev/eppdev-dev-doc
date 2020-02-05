---
title: 单机版Hive安装配置说明
subtitle: 大数据平台安装配置说明文档
author: 郝金隆
version: V1.0
date: 2020-02-05
company: 郝金隆
header-right: 2020-02
---

# 文档概述

## 文档说明

本文是单机版本hive的安装配置说明文档，用于指导搭建最基础的hive运行环境，具体内容包括：

* 文档说明
* 基础准备工作
* 基础配置
* 服务启动

## 参考文献

* [Hadoop官方安装配置说明](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/)
* [Hive官方安装说明](https://cwiki.apache.org/confluence/display/Hive/GettingStarted)

## 保密要求

无

# 基础准备工作

## 安装jdk

需要安装jdk1.8以上版本，并配置JAVA_HOME环境变量，具体过程略

## 安装数据库

可选安装mysql或者mariadb，具体过程略

## 下载原生版本的hadoop

可以使用国内的镜像进行下载，如[清华镜像]()、
[华为镜像](https://mirrors.huaweicloud.com/apache/hadoop/common/)，
根据需要选择相应的版本，一般建议下载2.7以上版本，或者3.x版本，以2.9.2为例：

~~~shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz
~~~

## 下载原生版本的hive文件

可以使用国内镜像进行下载，如[清华镜像](https://mirrors.tuna.tsinghua.edu.cn/apache/hive/)、
[华为镜像](https://mirrors.huaweicloud.com/apache/hive/)等，
根据需要选择相应的版本进行下载，一般建议下载2.3或者3.1版本，以2.3.6为例：

~~~shell
wget https://mirrors.huaweicloud.com/apache/hive/hive-2.3.6/apache-hive-2.3.6-bin.tar.gz
~~~

# 安装配置hadoop

## 解压缩

~~~shell
tar xvfz hadoop-2.9.2.tar.gz
~~~

## hadoop的配置

### 创建单机版本的文件目录

~~~shell
cd hadoop-2.9.2
mkdir -pv data/{namenode,datanode}
~~~

### 修改配置

修改hadoop目录下的etc/hadoop/core-site.xml

~~~xml
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.proxyuser.hive.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.hive.groups</name>
        <value>*</value>
    </property>
</configuration>
~~~

修改hadoop目录下的etc/hadoop/hdfs-site.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.name.dir</name>
        <value>/home/jinlong.hao/02-proj/66-servers/hadoop-2.9.2/data/namenode</value>
    </property>
    <property>
        <name>dfs.data.dir</name>
        <value>/home/jinlong.hao/02-proj/66-servers/hadoop-2.9.2/data/datanode</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
</configuration>
~~~

## 初始化namenode data

~~~shell
bin/hdfs namenode -format
~~~

## hadoop的启动验证

启动hdfs服务：

~~~shell
sbin/start-dfs.sh
~~~

通过浏览器访问yarn和hdfs的web界面，即可确认是否正常安装：

* hdfs web访问地址：http://localhost:50070/
* yarn web访问地址：http://localhost:8088/


# Hive的安装与基础配置

## 数据库配置

使用root用户登录mysql或者mariadb数据库，创建hive metadata库，并配置安装权限

~~~sql
create database hive23 charset utf8;
grant all privileges on hive23.* to hive@localhost identified by 'hive';
flush privileges;
~~~


## 安装配置hive

### hive解压缩

~~~shell
tar xvfz apache-hive-2.3.6-bin.tar.gz
~~~

### 下载mysql驱动到hive的lib目录中

~~~shell
cd apache-hive-2.3.6-bin/lib
wget https://mirrors.huaweicloud.com/repository/maven/mysql/mysql-connector-java/5.1.48/mysql-connector-java-5.1.48.jar
~~~

### 配置hive

在hive目录下，创建data文件夹，用于存储hive中的数据

~~~shell
cd apache-hive-2.3.6-bin
mkdir data
~~~


在conf目录下创建hive-site.xml文件，内容如下：

~~~xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>hive.metastore.local</name>
        <value>true</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://localhost:3306/hive23?characterEncoding=UTF-8</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>hive</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>hive</value>
    </property>
</configuration>
~~~

### 初始化metadata

~~~shell
bin/schematool -initSchema -dbType mysql
~~~

### 初始化hdfs数据

在hadoop目录下执行以下命令：

~~~shell
bin/hadoop fs -mkdir       /tmp
bin/hadoop fs -mkdir       /user
bin/hadoop fs -mkdir       /user/hive
bin/hadoop fs -mkdir       /user/hive/warehouse
bin/hadoop fs -chmod g+w   /tmp
bin/hadoop fs -chmod g+w   /user/hive/warehouse
~~~

### 安装后的验证

~~~shell
bin/hive -e 'show tables;'
~~~

出现如下类似结果，即表示安装配置正常

~~~
% bin/hive -e 'show tables;'
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/Users/jinlong.hao/Project/09-servers/02-hive/hive-2.3.6/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/Users/jinlong.hao/Project/09-servers/01-hadoop/hadoop-2.9.2/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/Users/jinlong.hao/Project/09-servers/02-hive/hive-2.3.6/lib/hive-common-2.3.6.jar!/hive-log4j2.properties Async: true
OK
Time taken: 3.46 seconds
~~~

## 服务启动

### 启动hiveserver2

~~~
nohup bin/hiveserver2 &
~~~

### 服务验证

~~~shell
bin/beeline -u jdbc:hive2//
~~~

进入后执行show tables，可以有如下显示，即表示数据一切正常

~~~
0: jdbc:hive2://> show tables;
20/02/05 16:52:32 [HiveServer2-Background-Pool: Thread-43]: WARN conf.HiveConf: HiveConf of name hive.metastore.local does not exist
OK
+-----------+
| tab_name  |
+-----------+
| test      |
+-----------+
1 row selected (0.129 seconds)
~~~


