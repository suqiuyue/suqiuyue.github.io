title: 源码编译安装Apache Ranger（一）
author: 南边
tags:
  - Apache Ranger
categories:
  - Hadoop
date: 2018-02-07 09:42:00
---
### 环境准备
&ensp;&ensp;&ensp;这里首先确保Hadoop2.7版本以上的集群环境，安装可参考<http://www.cnblogs.com/qiuyuesu/p/8398091.html>。另外在源码编译Apache Ranger之前，需要依赖的环境：maven、git、gcc、数据库。
#### 安装maven<br>
（1）下载安装包：<https://maven.apache.org/download.cgi>
（2）解压安装包：
```
# tar -xzvf apache-maven-3.3.9.tar.gz
```
（3）设置环境变量,在文件中添加如下内容：
```
# vim /etc/profile
export MAVEN_HOME=/home/apache-maven-3.3.9
export PATH=$PATH:$MAVEN_HOME/bin
# source /etc/profile    //使修改生效
```
（4）查看maven版本：
```
# mvn -version
```
#### 安装git和gcc
&ensp;&ensp;&ensp;直接使用yum安装即可：
```
# yum install gcc
# yum install git
```
#### 安装数据库
&ensp;&ensp;&ensp;ranger支持mysql、postgrsql、oracle、sqlserver，这里选择postgresql作为ranger的数据存储仓库。<br>
（1）设置yum安装源
```
# yum install https://download.postgresql.org/pub/repos/yum/9.5/redhat/rhel-7-x86_64/pgdg-centos95-9.5-2.noarch.rpm
```
（2）安装postgresql和jdbc驱动
```
# yum install postgresql95-server postgresql95-contrib posgresql95
# yum install postgresql-jdbc*
```
（3）初始化数据库
```
# service postgresql-9.5 initdb
```
（4）配置远程访问
```
# cd /var/lib/pgsql/9.5/data
# vim pg_hba.conf 
//添加需要远程访问的主机
host all all x.x.x.x/32 trust
# vim postgresql.conf
listen_address="*"
```
（5）启动数据库
```
# service postgresql-9.5 start
```
#### 创建数据库
（1）修改初试密码
```
# su - postgres
-bash-4.1$ psql
psql（9.5.10）
postgres=# \password postgres  //修改postgres用户的密码
```
（2）创建ranger数据库
```
postgres=# create user ranger with password 'PASSWORD';
postgres=# create database  rangerdb owner ranger;
postgres=# grant all privileges on database rangerdb to ranger;
```
### 安装配置solr
#### 安装须知
&ensp;&ensp;&ensp;Apache Ranger0.6版本之后支持使用Apache Solr或HDFS文件来存储审计日志，其中solr存储日志并提供搜索。这里主要介绍使用solr来作为审计日志的存储检索工具。注意：solr必须在安装Ranger Admin或Ranger Plugins之前安装。其中solr的安装选项有两种形式：<br>
* **Solr -Standalone** ，Solr的单实例易于安装，并且与Zookeeper无关。建议使用此选项仅用于测试ranger。
* **SolrCloud**，这是Ranger的首选设置。SolrCloud是一种可扩展架构，可以作为单节点或多节点集群运行。它还具有其他功能，如复制和分片，可用于高可用性（HA）和可扩展性。您需要根据群集大小来规划部署。

#### 安装solr
（1）下载解压ranger安装包：
```
# tar -zxvf ranger-0.6.0.tar.gz
```
（2）利用ranger自带的安装脚本安装solr（这里也可以选择自行安装solr）,这里选择standalone模式安装solr：
```
# cd ranger-0.6.0/security-admin/contrib/solr_for_audit_setup
# vim install.properties    //修改solr的安装配置脚本
JAVA_HOME=/usr/java/jdk1.8.0_112
SOLR_USER=solr
SOLR_INSTALL=true
SOLR_DOWNLOAD_URL=http://archive.apache.org/dist/lucene/solr/6.5.1/solr-6.5.1.tgz
SOLR_INSTALL_FOLDER=/opt/solr
SOLR_RANGER_HOME=/opt/solr/ranger_audit_server
SOLR_DEPLOYMENT=standalone
SOLR_RANGER_DATA_FOLDER=/opt/solr/ranger_audit_server/data
```
（3）执行脚本安装solr
```
# ./setup.sh
```
（4）安装成功后启动solr服务：
```
# cd /opt/solr/ranger_audit_server/scripts/start_solr.sh
```
至此，已经安装好了ranger的审计服务部分。
