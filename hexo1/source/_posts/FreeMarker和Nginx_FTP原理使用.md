---
title: FreeMarker结合Nginx|FTP基础使用
date: 2018-07-02
tags: 
categories: 组件

---



### FreeMarker结合Nginx|FTP基础使用 ###

[TOC]

目前做页面模板引擎主流使用**Thymeleaf**(效率更高),本文主要介绍**FreeMarker**的相关知识

**传送门:Thymeleaf基础原理https://blog.csdn.net/abap_brave/article/details/53009149**

**传送门:Thymeleaf快速使用https://blog.csdn.net/u014042066/article/details/75614906**

#### 1.FreeMarker ####
##### 1.介绍 #####

1. FreeMarker是一个模板引擎,一个基于模板生成文件的通用工具,由**纯java**代码编写
2. FreeMarker不是一个Web框架,而是**适合Web框架**的一个组件
3. FreeMarker与容器无关,更加**通用**,而且免费

数值类型:**| String | 数值(不区分浮点) | boolean | 日期 | 集合 |**等大部分类型



##### 2.使用 #####

1. 引入jar包
2. 建立模板
3. 进行输出

#### 2.FTP结合 ####

##### 1.介绍 #####

FTP连接分主动模式和被动模式

**主动模式(port)**使用N(发送数据) 和N+1(发送FTP命令)两个端口,一般20 21

固定端口,可能造成数据被拦截窃取

**被动模式(pasv)**使用21 和P>1024所有端口

不固定大范围端口,可能造服务器服务器被攻击

**解决方案分析:https://blog.csdn.net/u014774781/article/details/48376633**

**FTP传送门(待更新)**

##### 2.使用 #####

FreeMarker生成静态页面后,可以通过FTP发送到服务器指定的位置存放

#### 3.Nginx结合 ####

##### 1.介绍 #####

Nginx是一个Http服务器,可以将服务器上的**静态文件通过Http协议**展现给客户端

##### 2.使用 #####

Nginx将FreeMarker发送来的静态页面以url的方式发送到客户端完成一套静态部署

**Nginx传送门(待更新)**

