---
title: Spring基础原理
date: 2018-07-15
tags: 
categories: Spring全家桶

---



## Spring基础原理 ##
[TOC]

### 1.Spring概念 ###

**提供一个ioc容器来管理Bean,并通过AOP方式来增加Bean的功能**

基于Spring-IoC和AOP来构建多层Java EE 项目,能构将项目内的组件进行解耦分离,大大提高了开发效率和维护效率



**通过反射+XML实现的对象管理工厂(大容器)**

**目的:**

1. 解耦,简化开发
2. AOP编程的支持
3. 声明事务的支持
4. 方便程序测试
5. 继承各种优秀框架
6. 降低Java EE API的使用难度

### 2.核心概念 ###

1. IoC: Inversion of Control 控制反转 (通过反射机制创建对象实例)
2. DI: Dependency Injection  依赖注入(将bean之间的关系交给spring容器管理,我们可以在service注入dao层的实例,controller中注入service层实例)
3. AOP: Aspect Oriented Programming  面向切面

### 3.Spring组成 ###

Spring框架的功能大概由20多个模块组成,这些模块按组可以分为以下几部分

**| 核心容器 | 数据访问/集成 | WEB | AOP | 设备 | 消息 | 测试 |**

​	核心模块:Beans Core Context spEL对应项目初始化时需要的四个核心包
![Spring架构](https://i.imgur.com/4dxxIk6.png)

### 4.Spring说明 ###

1. **bean元素:需要spring管理的对象,是Spring中最基础的单位(包括数据源/SessionFactory/事务管理等)**
2. **class属性: 需要spring管理对象的全类名**
3. **name属性:给被管理者起个引用名,根据该引用名就可以使用该对象(bean对象的标识)**
4. id属性:bean对象的唯一标识(和name的区别是整个spring中不可重复)
5. **lazy-init:是否延时加载,默认false(开启后只对单例有效)**
6. init-method:对象初始化方法
7. destory:对象销毁方法
8. scope:**singleton**(默认,适用实际开发的大部分情况)还是**prototype**



#### 1.Spring管理对象原理 ####

##### 1.Spring容器 #####

Spring要管理对象,就需要把对象加入到自己的容器中

**Spring容器**是Spring的核心,主要的责任便是**管理Spring中java的组件**



1. 对象加入到Spring容器的三种方法(依赖注入) **|无参构造+setter注入|有参构造方法注入|动静态工厂注入|**
2. 使用bean对象时,实例化容器的两种方法**|ClassPathXml..xml实例化|FileSystem...绝对路径实例化|**
3. ApplicationContext容器实例化后,默认会实例化内部的所有bean,通过getBean即可获取bean对象的使用权



##### 2.Spring容器与对象Bean #####

通常情况下,Bean是被动的接收Spring容器创建的实例,具有使用权,即Bean不对Spring进行访问

如果想让Bean对Spring进行访问,则需要手动配置,让Bean实现**BeanFactoryAware **接口...

**该操作非常不推荐,污染代码,使Bean和Spring耦合在一起,若非特别需要,否则不要用**



##### 3.Spring容器中Bean的继承特性与Java中继承区别 #####

spring中可以使用抽象bean,即abstract 属性为true,抽象bean不能被实例化,作用是**Spring中的继承**

**spring中的继承:**

主要用于bean的数量越来越多,许多属性配置冗余,此时可以使用继承

**spring的继承无法继承如下属性:**

- depends-on,aotuwirwe,dependency-check,singleton,scope,lazy-iniyt这些属性总是子Bean定义，或采用默认值。 

**Bean继承与java中继承的区别： **

- Spring中的子bean和父Bean可以是不同类型，但java中的继承则可保证子类是一种特殊的父类； 
- Spring中的Bean的继承是实例之间的关系，因此只要表现在参数值的延续，而java中的继承是类之间的关系，主要表现为方法、属性之间的延续； 
- Spring中的子Bean不可以作为父Bean使用，不具备多态性，java中的子类完全可以当成父类使用。 



##### 4.Spring容器中Bean模式的生命周期 #####

**singleton：** Spring容器能够准确的追踪其创建/使用/销毁

**prototype：**Spring容器仅负责其创建,无法追踪其使用和销毁,每次创建后都会委托给客户端,不再对其追踪 



##### 5.强制初始化Bean(使用较少) #####
Spring有一个默认的规则，总是先初始化主调Bean，然后在初始化依赖Bean。
为了指定Bean在目标Bean之前初始化，可以使用depends-on属性

##### 6.自动装配 #####
- Spring能自动装配Bean与Bean之间的依赖关系，即使无需使用ref显式指定依赖Bean。
- 自动装配可以减少配置文件的工作量，但是降低了依赖关系的透明性和依赖性。
- 可以根据指定属性值类型来缩小自动装配的范围(很少指定)
-  **| no | byName | byType | constructor | autodetect | **

##### 7.依赖检察(使用较少) #####
Spring提供一种依赖检查的功能，可以防止出现配置手误，或者其他情况的错误。
dependency-check="all" 该属性值可以为**| none | simple | objects | all | **


#### 2.Spring核心机制:IoC/DI ####

无论是**IoC**(控制反转) 还是**DI**(依赖注入),其含以上是互相包容的,在控制反转时便实现依赖注入

- **传统方法:**当一个java实例(A)需要调用另一个java实例(B)时,需要在A中new一个B来构建
- **Spring管理中:**B对象的创建交给了Spring来管理(控制反转) | 将B对象注入给A对象的过程称为(依赖注入)

#### 4.Spring AOP ####

##### 1.概念 #####

 **AOP**（Aspect Oriented Programming），即面向切面编程，基于**面向过程**

是**OOP**（Object Oriented Programming，面向对象编程）的补充和完善。 

**(OOP无法关注到程序的切入点,AOP具有更强大的切面控制力)**

例如:

- 面向对象的流程是用户注册信息,然后插入到数据库中
	 面向切面的流程是用户注册信息,在插入数据库前进行控	制,在插入数据库后进行控制


**AOP使用横切技术,把软件系统分成两部分**
**核心关注点**,特点:纵向关系为主,建立对象层次结构
**横切关注点**,特点:横向关系为主,分布在核心关注点的周围,能够切开封装对象,对内部重复的部分进行重新封装,降低耦合性,如权限认证、日志、事务,AOP可以分离系统中的关注点,达到更高效的开发和运行



##### 2.五种横切方案 #####
1. 前置通知

2. 后置正常通知
3. 后置异常通知
4. 后置始终通知
5. 环绕通知



##### 3.八大核心概念 #####

1. Joinpoint 连接点
    AOP执行程序的特定位置

2. PointCut 切点
    AOP通过切点来确定特定的连接点位置

3. Advice 增强
    织入目标类连接点上的一段代码,可以描述程序,也可以确定方位,
       只有通过切点信息和方位信息,才能确定特定连接点并执行增强逻辑

4. Target 目标对象
    增强逻辑的织入目标类

5. Introduction 引介
    特殊增强,给指定类添加属性和方法,也可以动态的添加接口的实现业务逻辑

6. weaving 织入

    增强 添加到 目标类 的 连接点 的过程,将目标,增强和引介很好的结合在了一起,AOP常用的三种织入技术
    		编译期织入,这要求使用特殊的Java编译器.

    ​		类装载期织入,这要求使用特殊的类装载器.

    ​		动态代理织入,在运行期为目标类添加增强生成子类的方式.
    		Spring采用动态代理织入,而AspectJ采用编译期织入和类装载器织入.

7. Proxy  代理

    目标被增强织入后生成一个新的代理类,该代理类可能和原类是一个接口,也可能是原类的子类,可以采用和原类相同的方法调用

8. Aspect 切面

    由增强和切点组成, [既包含横切逻辑的定义,也包括连接点的定义] ,能够把这两种定义织入到指定连接点中



##### 4.AOP底层调用的两种代理机制 #####

- **JDK动态代理:**针对接口类产生代理
- **CGlib动态代理:**针对没有实现接口的类产生代理,应用的是底层字节码增强技术,生成当前类的子类对象



### 5.Spring深入拓展 ###

#### 1.利用后处理器扩展Spring容器... ####

#### 2.Spring的“零配置”支持(注解) ####
	**| Component | Controller | Service | Repository | Scope | Resource | Autowired | Qualifier | JsonIgnore |**

#### 3.资源访问... ####


**深入拓展传送门:https://www.cnblogs.com/shijiaoyun/p/7458341.html**

 