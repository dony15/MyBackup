---
title: FastDFS 基础原理
date: 2018-07-06
tags: 
categories: 框架

---



## FastDFS 基础原理 ##

[TOC]

### 深度优先搜索 ###

Depth First Search 

**无向图算法概念**(一种递归原理)

先按照一条边进行搜索,当遇到第一个节点时,对它相邻的其他节点进行搜索并标记为已查找的节点(会查找完第一条节点的最深层后返回)
详细见算法目录(持续更新)


!["无向图算法图片"的图片搜索结果](http://i.imgur.com/vJ23ZgT.jpg)

### FastDFS--结构 ### 

<h2 id="1">FastDFS 架构--结构</h2>


**FastDFS服务有三个角色:跟踪服务器(tracker server)、存储服务器(storage server)和客户端(client)**

主要解决了海量数据存储问题 ,特别适合以中小文件（建议范围：4KB < file_size <500MB）为载体的在线服务。 

#### 跟踪器Tracker :####  

主要做调度工作，相当于mvc中的controller的角色，在访问上起负载均衡的作用。跟踪器和存储节点都可以由一台或多台服务器构成，跟踪器和存储节点中的服务器均可以随时增加或下线而不会影响线上服务，其中**跟踪器**中的所有服务器都是**对等**的，可以根据服务器的压力情况随时增加或减少。 

跟踪器Tracker负责管理所有的Storage和group，本身不需要持久化任何数据,直接增加机器就可以拓展tracker,每个Storage在启动后会连接Tracker，并周期性保持联系.



#### 存储服务器Storage: ####

以group为最小单位,方便实现 应用隔离、负载均衡、副本数定制（group内storage server数量即为该group的副本数）,建议同一group内的配置尽量相同,减少资源浪费(storage依赖于本地文件系统)



#### 客户端Client: ####

**基本文件访问接口:**比如upload、download、append、delete等，以客户端库的方式提供给用户使用。 





### FastDFS--运行 ###



**tracker**


当集群中不止一个tracker server时，由于tracker之间是完全对等的关系，客户端在upload文件时可以任意选择一个trakcer。

**group**

当tracker接收到upload file的请求时，会为该文件分配一个可以存储该文件的group，支持如下选择group的规则：

1. Round robin，所有的group间轮询 
2. Specified group，指定某一个确定的group 
3. Load balance，剩余存储空间多多group优先


**storage**


当选定group后，tracker会在group内选择一个storage server给客户端，支持如下选择storage的规则： 
1. Round robin，在group内的所有storage间轮询 
2. First server ordered by ip，按ip排序 
3. First server ordered by priority，按优先级排序（优先级在storage上配置）


**storage path**


当分配好storage server后，客户端将向storage发送写文件请求，storage将会为文件分配一个数据存储目录，支持如下规则： 
1. Round robin，多个存储目录间轮询 
2. 剩余存储空间最多的优先


**Fileid**


选定存储目录之后，storage会为文件生一个Fileid，
由storage server ip、文件创建时间、文件大小、文件crc32和一个随机数拼接而成，
然后将这个二进制串进行base64编码，转换为可打印的字符串。


**选择两级目录**


当选定存储目录之后，storage会为文件分配一个fileid，每个存储目录下有两级256*256的子目录，storage会按文件fileid进行两次hash（猜测），路由到其中一个子目录，然后将文件以fileid为文件名存储到该子目录下。


**生成文件名** 


当文件存储到某个子目录后，即认为该文件存储成功，接下来会为该文件生成一个文件名，文件名由group、存储目录、两级子目录、fileid、文件后缀名（由客户端指定，主要用于区分文件类型）拼接而成。

**文件同步**


写文件时，客户端将文件写至group内一个storage server即认为写文件成功，
storage server写完文件后，会由后台线程将文件同步至 [同group] 内其他的storage server。
storage的同步进度会作为元数据的一部分汇报到tracker上，tracke在选择读storage的时候会以同步进度作为参考。


**Download file**


客户端upload file成功后，会拿到一个storage生成的文件名，接下来客户端根据这个文件名即可访问到该文件。




### FastDFS--特点 ###




#### 小文件合并存储 ####

**解决问题:**


1. 本地文件系统inode数量有限，从而存储的小文件数量也就受到限制。 
2. 多级目录+目录里很多文件，导致访问文件的开销很大（可能导致很多次IO） 
3. 按小文件存储，备份与恢复的效率低


FastDFS在V3.0版本里引入小文件合并存储的机制，可将多个小文件存储到一个大的文件（trunk file），为了支持这个机制，FastDFS生成的文件fileid需要额外增加16个字节 




#### HTTP访问支持 ####


客户端可以通过http协议来下载文件，tracker在接收到请求时，通过http的redirect机制将请求[重定向]至文件所在的storage上；除了内置的http协议外，FastDFS还提供了通过apache或nginx扩展模块下载文件的支持。




#### 负载均衡 ####


group机制本身可用来做负载均衡，但这只是一种静态的负载均衡机制，需要预先知道应用的访问特性；同时group机制也导致不可能在group之间迁移数据来做动态负载均衡。




### FastDFS--使用小结 ###



1. 分别配置Tracker地址(上传存储使用)和Storage地址(响应回显使用)
2. 接收前段file文件后,将名字拆分重塑后存储
3. 响应url则拼接Storage地址生成
4. 每次上传文件后都会返回一个地址，用户需要自己保存此地址。
5. 前段设计:将整个编辑内容存进一个由事件控制的表单,当图片上传的时候不会影响到表单的完整性,而且可以依靠上传时间来动态生成回显方案,将url放进input中,清除不必要的组件(可能影响表单提交完整性的部分)
6. 为了支持大容量，存储节点（服务器）采用了分卷（或分组）的组织方式。存储系统由一个或多个卷组成，卷与卷之间的文件是相互独立的，所有卷的文件容量累加就是整个存储系统中的文件容量。一个卷可以由一台或多台存储服务器组成，一个卷下的存储服务器中的文件都是相同的，卷中的多台存储服务器起到了冗余备份和负载均衡的作用。

**注意:spring-mvc中除了要配置上传解析器之外,还需要将String的字符串指定为UTF-8(默认8859-1)**


### FastDFS原理系列文章(转发)  ###
**https://blog.csdn.net/hfty290/article/details/42076205**




