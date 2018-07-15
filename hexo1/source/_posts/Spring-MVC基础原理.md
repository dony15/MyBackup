---
title: Spring-MVC基础原理
date: 2018-07-15
tags: 
categories: Spring全家桶
---



## 1.Spring-MVC基础原理 ##

[TOC]

### 1.概念 ###

优秀的Web框架,具有**松散耦合**,**拔插组件结构**,**注解驱动**,**REST风格支持**等特性,比其他web框架更具有扩展性和灵活性

在**数据绑定,视图解析,本地化处理,静态资源处理**上有不俗的表现,远超Struts2,WebWork等MVC框架

![springmvc](https://i.imgur.com/0vWxf3N.png)



### 2.MVC框架 ###

MVC全称 Model veiw Controller(模型视图控制器)  **软件级的解耦分离**

- M:主要包含**service**(核心业务逻辑)和**dao**(数据库访问)
- V:静态资源,如**HTML5,JS,CSS**等
- C:**servlet**(主要处理**页面的转发和重定向**,**数据的接收**,**域对象的操作**,)和**jsp**(本身也是servlet)



MVC 分层有助于管理复杂的应用程序，因为您可以在一个时间内专门关注一个方面。例如，您可以在**不依赖业务逻辑的情况下专注于视图设计**。同时也**让应用程序的测试更加容易**。 

MVC 分层同时也简化了分组开发。不同的开发人员可同时开发**视图**、**控制器逻辑**和**业务逻辑**。 



**(扩展)**Spring MVC是基于 Model 2实现的技术框架,Model 2是经典的MVC(model,view,control)模型在WEB应用中的变体.这个改变主要源于HTTP协议的无状态性,Model 2 的目的和MVC一样,也是利用处理器分离模型,视图和控制,达到不同技术层级间松散层耦合的效果,提高系统灵活性,复用性和可维护性.大多情况下,可以将Model 2 与 MVC等同起来. 

**(扩展)三层架构基础**

- **物理三层架构:**客户端(如浏览器)/Web服务器/数据库服务器

- **逻辑三层架构:**表现层/业务逻辑层/数据库访问层




### 3.Spring MVC体系概述 ###

Spring-MVC围绕着**DispatcherServlet(前段控制器)**这个核心展开,所有的前端请求都会**拦截经过这里**分发到Spring MVC的各个处理器中处理,**(扩展)**如注解驱动控制器,请求及响应的信息处理,视图解析,本地化解析,上传文件解析,异常处理及表单标签绑定内容等...

 

### 4.Spring MVC核心组件 ###

- **DispatcherServlet：**作为前端控制器，整个流程控制的中心，控制其它组件执行，统一调度，降低组件之间的耦合性，提高每个组件的扩展性。 
- **HandlerMapping：**通过扩展处理器映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。 
- **HandlAdapter：**通过扩展处理器适配器，支持更多类型的处理器,调用处理器传递参数等工作! 
- **ViewResolver：**通过扩展视图解析器，支持更多类型的视图解析，例如：jsp、freemarker、pdf、excel等。 



### 5.**Spring MVC执行流程**  ###

![mvc流程](https://i.imgur.com/gfPibwt.png)

### 6.DispatcherServlet ###

#### 1.核心 ####

1. DispatcherServlet 是Spring-MVC的核心构成,负责协调所有mvc的处理器,
2. **DispatcherServlet可以和Spring-IoC无缝集成,获得Spring的所有好处**
3. 使用时需要在web.xml中对DispatcherServlet进行配置



#### 2.DispatcherServlet继承关系图 ####

![继承关系图](https://i.imgur.com/isTWyBE.png)





#### 3.DispatcherServlet的责任 ####

主要负责调度Spring-mvc的工作,并控制MVC的流程

1. 文件上传解析，如果请求类型是multipart将通过MultipartResolver进行文件上传解析；
2. 通过HandlerMapping，将请求映射到处理器（返回一个HandlerExecutionChain，它包括一个处理器、多个HandlerInterceptor拦截器）；
3. 通过HandlerAdapter支持多种类型的处理器(HandlerExecutionChain中的处理器)；
4. 通过ViewResolver解析逻辑视图名到具体视图实现；
5. 本地化解析；
6. 渲染具体的视图等；
7. 如果执行过程中遇到异常将交给HandlerExceptionResolver来解析。

#### 4.DispatcherServlet核心代码 ####

#### 5.DispatcherServlet辅助类 ####

#### 传送门:https://xuzhongcn.github.io/#top



### 7.常用注解(实用重点) ###

#### 1.@RequestMapping 请求方式 ####

#### 2.@RequestParam 处理请求参数 ####

#### 3.@PathVariable 路径传参 ####
