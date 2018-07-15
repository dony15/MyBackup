---
title: Solr基础原理
date: 2018-07-06
tags: 
categories: 框架

---



### Solr基础原理 ###

[TOC]

#### 1.目录核心组成 ####

**1.core**

​	solr的索引库,可以理解为数据库,需要手动创建(文件夹),core可以根据需要建立多个索引库,索引库的内容可以在后台看到也可以在core中看到

**2.solrhome**

​	solr的配置目录,solr服务器所有的配置文件存放的目录(**core创建在solrhome中**)

**3.collection**

​	solr的逻辑索引(逻辑意义上的完整索引),由多个shard的组成,每个shard又由一个leadereplica和多个replica,每个replica都是物理索引,即每个replica都对应着一个core,collection本质是可以跨越多个核的索引,包含冗余的索引.



**参考https://blog.csdn.net/zhousenshan/article/details/51799567**



#### 2.配置详解 ####



##### 1.配置中文分词 #####

1. <fieldType name="text_ik" class="solr.TextField">   <!-- text_ik 中文分词包的引用名 -->
2.   <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>  <!-- 中文分词包 -->
3. </fieldType>
4. <!-- 字段title 使用中文分词 stored="true"下载并索引 -->
5. <field name="item_title" type="text_ik" indexed="true" stored="true"/> 
6. <!-- 字段price 使用long类型 -->
7. <field name="item_price"  type="long" indexed="true" stored="true"/>
8. <!-- city_id 使用long类型 -->
9. <field name="item_city_id"  type="long" indexed="true" stored="true"/> 
10. <!-- city_name 因为城市名固定,所以不需要分词,String即可 -->
11. <field name="item_city_name" type="string" indexed="true" stored="true" />
12. <!-- image 图片地址,String类型 -->
13. <field name="item_image" type="string" indexed="true" stored="true" />
14. <!-- content 中文分词 stored="false"(不下载,但可以索引) -->
15. <field name="item_content" type="text_ik" indexed="true" stored="false" />
16. <!-- item_keywords 自定义查询名(关键字);可以根据title||city_name||content来查询 -->
17. <field name="item_keywords" type="text_ik" indexed="true" stored="false" multiValued="true"/>
18. <copyField source="item_title" dest="item_keywords"/>
19. <copyField source="item_city_name" dest="item_keywords"/>
20. <copyField source="item_content" dest="item_keywords"/>





1. <!-- 默认使用item_keywords(Solr后台分词查询中显示自定义的字段) -->
2. <requestHandler name="/select" class="solr.SearchHandler">
3. ​    <!-- default values for query parameters can be specified, these
4. ​         will be overridden by parameters in the request
5. ​      -->
6. ​     <lst name="defaults">
7. ​       <str name="echoParams">explicit</str>
8. ​       <str name="df">item_keywords</str>
9. ​        <int name="rows">10</int>



1. <!-- 默认使用item_keywords(开启查询) -->
2. <requestHandler name="/query" class="solr.SearchHandler">
3. ​     <lst name="defaults">
4. ​       <str name="echoParams">explicit</str>
5. ​       <str name="wt">json</str>
6. ​       <str name="indent">true</str>
7. ​       <str name="df">item_keywords</str>
8. ​     </lst>
9. </requestHandler>



##### 2.配置Solr Dataimport #####

1. <!-- dataimport 开启Solr连接数据库功能 -->
2. <requestHandler  name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler"> 
3. ​    <lst name="defaults">      
4. <!-- dataimport读取data-config.xml设定的JDBC配置文件 -->   
5. ​         <str name="config">data-config.xml</str> 
6. ​     </lst>         
7. </requestHandler>  



1. <dataConfig>       
2. <!-- JDBC配置 -->
3. ​    <dataSource type="JdbcDataSource" driver="com.mysql.jdbc.Driver" url="jdbc:mysql://localhost:3306/travel_db?characterEncoding=utf-8" user="root" password="root" batchSize="-1"/>   
4. ​    <document>   
5. <!-- 查询语句(全表查询)字段,并匹配分词中设定的name -->  
6. ​        <entity name="hotel" query="select ID,TITLE,PRICE,IMAGE,CITY_NAME, CITY_ID from HOTEL" dataSource="JdbcDataSource">        
7. ​            <field column="ID" name="id" /> 
8. ​            <field column="TITLE" name="item_title" /> 
9. ​                            <field column="PRICE" name="item_price" /> 
10. ​                            <field column="CONTENT" name="item_content" />
11. <field column="IMAGE" name="item_image" />
12. ​                            <field column="CITY_NAME" name="item_city_name" />
13. <field column="CITY_ID" name="item_city_id" />
14. ​         </entity>       
15. ​    </document>        
16. </dataConfig>  



##### 3.Solr后台的使用 #####

第一次先Dataimport-->Execute导入,然后Refresh刷新状态即可

**Query:查询功能** 

​	q  * ; *    -->第一个 * 表示字段; 第二个 * 表示字段的内容;  

​	如    item_keywords:北京   分词中有"北京"关键字的内容

​		item_price:[* TO 200]  价格是200以内的内容

​		item_price:[100 TO 700]  价格是100-200的内容



#### 3.java中的作用 ####

建立一个新的索引模块 index,接口层和实现发布层

写Solr**更新**和**搜索**两个方法dubbo发布即可在controller中使用

(一般与MQ一起使用,如activeMQ,见**activeMQ基础与运用章节**)







