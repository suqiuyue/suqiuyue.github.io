title: 初识Apache Ranger
author: 南边
tags:
  - Apache Ranger
categories:
  - Hadoop
date: 2018-02-06 17:55:00
---
### 1. 介绍
&nbsp; &nbsp; &nbsp; &nbsp; Apache Ranger是Hadoop平台上集中式的安全管理框架，为企业核心安全认证、授权、审计以及数据保护提供了一个安全策略管理中心。Apache Ranger 的主要功能是为Hadoop生态圈组件提供细粒度的访问控制，它还具有如下功能：<br>
*  统一的管理界面，包括安全管理员管理、策略管理、日志审计、Ranger KMS管理、插件管理等；
*  基于策略的访问控制模型；
*  通用的策略同步与决策逻辑，方便插件的扩展；
*  通用的安全管理员访问日志审计逻辑，可定义的日志存储范围，如HDFS、Solr等；
*  支持用户和LDAP、Unix系统的用户同步；
*  支持对HDFS、Hive、HBase、Strom、YARN、Knox、Kafka等Hadoop生态圈组件的权限管控和审计。<br>
<img src="/images/ranger_architecture.png" width=80% height=50% align=center/>

### 2. 组成
&nbsp; &nbsp; &nbsp; &nbsp;Apache Ranger的组成结构主要由以下三个模块组成：
* **Ranger admin**：提供了安全管理员安全管理的web界面。安全管理员可以创建和更新策略文件，并将其存储在策略数据库中。每个组件内的插件刷新器定期查询这些策略文件。该门户还包括发送HDFS中或在关系数据库中从插件收集存储审计数据的审计服务器。
* **Ranger plugins**：插件是轻量级的java程序，其嵌入在需要安全控制的组件进程中。它提供了两种功能：一是从Ranger admin端拉取安全管理员配置的安全策略到本地。当安全管理员提出访问请求时，根据安全策略判断该用户是否有权限访问；二是从本地将用户访问的记录返回给Ranger服务进行审计。
* **User group sync**：提供从Unix、LDAP/AD拉取用户和用户组的功能，同步的用户和用户组能够展示在Ranger admin的web界面中。<br>

### 3. 实现原理
&nbsp; &nbsp; &nbsp; &nbsp;Apache Ranger 这个集中式的安全框架集成了认证、授权、审计的功能，各功能的实现介绍如下。<br>
1. 认证<br>
&nbsp;&nbsp;&nbsp;&nbsp;Ranger 支持使用Unix、File和LDAP/AD管理用户、用户组信息，Ranger Usersync提供了用户同步功能。
2. 授权<br>
&nbsp;&nbsp;&nbsp;&nbsp;Ranger 采用Web UI界面提供了安全管理员可视化地管理授权策略，由Ranger Admin响应web端的请求，并将策略存储在数据库中。Ranger以插件的形式集成到Hadoop组件当中，当组件启动时，相应的ranger插件也随之初始化，从Ranger Admin拉取策略保存到本地，以便对用户访问请求进行判定。<br>
<img src="/images/ranger_principle.png" width=60% height=50% align=center/><br>
&nbsp;&nbsp;&nbsp;&nbsp;Ranger使用了基于策略的访问控制模型，安全管理员可以通过操作授权策略对用户的访问权限进行管理。策略分为两种类型：基于资源的策略和基于标签的策略。基于标签的策略，直接将资源以标签名的形式授权给用户，进一步保证了数据的安全。这里标签可以使用文件和Apache Atlas来管理，Atlas提供了将HDFS、Hive等资源分类标记的功能。Ranger Tagsync提供了与标签管理工具中标签同步功能。
3. 审计<br>
&nbsp;&nbsp;&nbsp;&nbsp;Ranger审计对用户访问数据、安全管理员操作策略和服务、安全管理员登录web UI、以及插件的状态这几方面进行了实时的记录。在Ranger 0.6版本以后，只支持将访问日志保存在HDFS文件和Solr中。其中Ranger集成Solr，可以在Ranger web页面上对审计日志进行搜索查询。