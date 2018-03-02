title: 源码编译安装Apache Ranger（二）
author: 南边
tags:
  - Apache Ranger
categories:
  - Hadoop
date: 2018-02-07 16:46:00
---
### 安装Ranger Admin
（1）编译ranger源码安装包
```
# cd ranger-0.6.0
# mvn clean compile package assembly:assembly install
```
若编译成功，则在target目录下出现以下压缩包：
<img src="/images/ranger_src.jpg" width=80% height=60% align=center/>
（3）修改安装配置文件
```
# cd ranger-0.6.0/security-admin/scripts/install.properties 
//设置数据库
DB_FLAVOR=POSTGRES
SQL_CONNECTOR_JAR=/usr/share/java/postgresql-jdbc.jar
//数据库管理员属性
db_root_user=postgres
db_root_password=123456
db_host=localhost
#ranger的数据库用户
db_name=rangerdb
db_user=ranger
db_password=123456
#审计存储设置
audit_store=solr
audit_solr_urls=http://localhost:6083/solr/ranger_audits
audit_solr_user=solr
audit_solr_password=123456
```
（4）运行安装脚本并启动服务
```
# ./setup.sh
# ranger-admin start
```
（5）通过使用浏览器访问http://<host_address>:6080验证，出现以下页面则表示安装成功：
<img src="/images/ranger_admin.png" width=40% height=40% align=center/>
### 配置Ranger UserSync
&ensp;&ensp;&ensp;Ranger UserSync实现了ranger用户和用户组的同步，具体配置过程如下：
（1）解压ranger usersync安装包：
```
# tar -zxvf ranger-0.6.0-tagsync.tar.gz
```
（2）修改安装配置文件（按实际情况设置）
```
# cd ranger-0.6.0-tagsync
# vim install.properties
ranger_base_dir =/opt/ranger
hadoop_conf=/home/hadoop-2.7.2/etc/Hadoop
logdir = /var/log/ranger/usersync
```
（3）安装并启动用户组同步服务
```
# ./setup.sh
# ./ranger-usersync-services.sh start
```
安装成功后，可以在ranger的管理界面上看见同步后的用户列表：
<img src="/images/ranger_usersync.png" width=80% height=40% align=center/>
### 配置HDFS插件
&ensp;&ensp;&ensp;Apache Ranger支持的组件很多，安装配置的过程类似，这里选取安装HDFS以作示例。<br>
（1）解压ranger-hdfs安装包：
```
# tar -zxvf ranger-0.6.0-hdfs-plugin.tar.gz
```
（2）修改安装配置文件（按实际情况设置）
```
# cd ranger-0.6.0-hdfs-plugin
# vim install.properties
POLICY_MGR_URL=http://localhost:6080
SQL_CONNECTOR_JAR=/usr/share/java/postgresql-jdbc.jar
REPOSITORY_NAME=hadoop_hadoop
#审计存储
XAAUDIT.SOLR.ENABLE=true
XAAUDIT.SOLR.URL=http://localhost:6083/solr/ranger_audits
XAAUDIT.SOLR.USER=solr
XAAUDIT.SOLR.PASSWORD=123456
XAAUDIT.SOLR.ZOOKEEPER=NONE
XAAUDIT.SOLR.FILE_SPOOL_DIR=/var/log/hadoop/hdfs/audit/solr/spool
XAAUDIT.SOLR.IS_ENABLED=true
XAAUDIT.SOLR.MAX_QUEUE_SIZE=1
XAAUDIT.SOLR.MAX_FLUSH_INTERVAL_MS=1000
XAAUDIT.SOLR.SOLR_URL=http://localhost:6083/solr/ranger_audits
```
（3）启动hdfs插件服务
```
# cd /ranger-hdfs-plugin
# ./enable-hdfs-plugin.sh
```
&ensp;&ensp;&ensp;<font color="red">注意：</font>根据脚本文件执行时，在插件安装位置找不到hadoop conf和lib文件夹，会导致插件安装失败。出现以下两个错误：<br>
* **ERROR:Unable to find the conf directory of component [hadoop]; dir [/opt/hadoop/conf] not found.**<br>
解决办法：创建一个符号链接使hadoop conf链接到目的目录：
```
# ln-s /home/hadoop-2.7.2/etc/hadoop/ /opt/hadoop/conf
```
* **ERROR:Unable to find the lib directory of component [hadoop]; dir [/opt/hadoop/lib] not found.**<br>
解决办法：将HDFS Plugin目录中的jar和Hadoop中包含HDFSDE jar都指向目的目录。
```
# cp ranger-hdfs-plugin/lib/ranger-hdfs-plugin-impl/*.jar  /home/hadoop-2.7.2/share/hadoop/hdfs/lib/
# mkdir /root/hadoop/lib
# ln -s /home/hadoop-2.7.2/share/hadoop/hdfs/lib/ /opt/hadoop/lib/
```

&ensp;&ensp;&ensp;解决了这些问题之后，Ranger为HDFS提供服务的插件就配置好了，现在就可以进入ranger管理界面添加对HDFS文件/文件夹访问控制策略了。
<img src="/images/ranger_hdfs_policy.png" width=70% height=40% align=center/>
