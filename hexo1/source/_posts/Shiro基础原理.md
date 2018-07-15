---
title: Shiro基础原理
date: 2018-07-04
tags: 
categories: 框架

---



## Shiro基础原理 ##

[TOC]



### 1.简介  ###

**shiro是apache的一个开源框架，实现 |认证|授权|为核心的一系列权限管理框架.**

- Web 应用程序一般做法通过表单提交用户名及密码达到认证目的。 
- “授权”即是否允许已认证用户访问受保护资源。 

### 2.对比 ###

**Shiro与Spring Security**

1. 简单性:shiro更加简单,更容易理解
2. 灵活性:shiro可以使用在 |Web|EJB|IoC| 等大部分的应用环境,而Spring Security必须和Spring一起集成使用
3. 拔插性:shiro干净的API(工具类集合)和设计模式(单例+工厂)使它可以方便的和许多其他框架整合,Spring Security则只能与Spring一起集成

### 3.组成 ###

![img](file:///C:/Users/Administrator/Documents/My%20Knowledge/temp/1b7bc2c3-9f2c-4ea2-8ed7-97820c205482/128/index_files/871676-20160722213407794-1894786938.png)

#### 1.三个核心组件 ####	

1. Subject: 令牌与项目的登录关系,Shiro保证了项目整体的安全性,是**Shiro对外API的核心**
2. Security Manager:负责安全认证预授权等  **Shiro的核心** 
3. Realm:整个框架中**必须**由设计者自行实现的模块之一.并且Shiro支持多个**Realm数据源**,最为重要的一种实现方式--->数据库查询,当需要多个数据库组合验证时,多个数据源的效果就体现出来

------
#### 2.主要功能 ####

1. Authentication:    身份认证
2. Authorization:  授权,权限验证
3. Session Manager: 会话管理
4. Cryptography:加密
5. Web Support: web支持
6. Caching:缓存
7. Concurrency:多线程验证
8. Testing:提供测试支持
9. Run As:允许一个用户假装另一个用户访问
10. Remember Me: 记住我



#### 3.组件和内容流程分析 ####

##### 1.subject #####

外部API核心,存储用户数据和返回数据

```java
Subject currentUser = SecurityUtils.getSubject();
```

**获得Subject的方法,有了Subject才能和Shiro做深入的交互**

##### 2.SessionManager #####

1. Shiro的Session提供了HttpSession常规的大部分功能,但是又有区别,即:Session不依赖于**HTTP环境**,可以在程序任何地方使用

2. Shiro的Session可以在任何的环境下使用**相同的API**,而且是**自动启动SessionManager**


如果希望在**当前与应用程序会话期间*,为用户提供内容,则可以设置Session

```
Session session = currentUser.getSession();
session.setAttribute( "someKey", "aValue" );
```

##### 3.登录认证Authentication #####

Shiro的认证功能,会根据Subject的信息进行判断,如果认证过,则直接进入/如果没认证,则需要先认证

```
if ( !currentUser.isAuthenticated() ) {
    //collect user principals and credentials in a gui specific manner
    //such as username/password html form, X509 certificate, OpenID, etc.
    //We'll use the username/password example here since it is the most common.
    //(do you know what movie this is from? ;)
    UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
    //this is all you have to do to support 'remember me' (no config - built in!):
    token.setRememberMe(true);
    currentUser.login(token);
}
```

**UsernamePasswordToken**(username/password)

以特定的方式收集用户的**主体**和**凭证**

**Remember Me**  no config - built in!(true/false)

Shiro内置功能,记住用户(详情待更新)

**登录尝试失败的反馈--异常**

```
try {
    currentUser.login( token );
    //if no exception, that's it, we're done!
} catch ( UnknownAccountException uae ) {
    //username wasn't in the system, show them an error message?
} catch ( IncorrectCredentialsException ice ) {
    //password didn't match, try again?
} catch ( LockedAccountException lae ) {
    //account for that username is locked - can't login.  Show them a message?
}
    ... more types exceptions to check if you want ...
} catch ( AuthenticationException ae ) {
    //unexpected condition - error?
}
```

**Shiro中使用多种异常完善认证**

- 将Subject的.login(token)进行捕获,从而的到许多种异常提醒,根据相应的异常判断用户登录的错误信息
- 注意:Shiro有丰富的认证异常设定并支持自定义异常,在Realm中通过判断条件,抛出异常的方式,可以在Controller中接收需要的异常数据来完善程序的开发





------

![img](file:///C:/Users/Administrator/Documents/My%20Knowledge/temp/b9f81ac7-ad34-4adc-b4c2-cb2f73a4f2dd/128/index_files/48cae49c-8924-4a4e-8efd-bdbf38f07c97.jpg)	

1. **注意:Shiro不会自己维护用户|权限;**
2. **需要开发者去 设计|提供 ;**
3. **然后通过接口注入给Shiro即可**

### 4.源码 ###

 #### Token认证 ####

**JdbcRealm**

Shiro -->JdbcRealm封装的固定sql

 [1.封装根据用户名查询密码的SQL语句]

 `````java
 	/**
     * The default query used to retrieve account data for the user.
     */
    protected static final String DEFAULT_AUTHENTICATION_QUERY = "select password from users where username = ?";
 `````

    [2.盐加密&&authenticationQuery验证查询(判断)] 

```java
/**
     * Sets the salt style.  See {@link #saltStyle}.
     * 
     * @param saltStyle new SaltStyle to set.
     */
    public void setSaltStyle(SaltStyle saltStyle) {
        this.saltStyle = saltStyle;
        if (saltStyle == SaltStyle.COLUMN && authenticationQuery.equals(DEFAULT_AUTHENTICATION_QUERY)) {
            authenticationQuery = DEFAULT_SALTED_AUTHENTICATION_QUERY;
        }
    }
```

  [3.发现源代码中使用预编译的原生JDBC,并根据索引查找对比,所以要求自定义语句时不能乱写,根据规则走] 

```
PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            ps = conn.prepareStatement(authenticationQuery);
            ps.setString(1, username);

            // Execute query
            rs = ps.executeQuery();

            // Loop over results - although we are only expecting one result, since usernames should be unique
            boolean foundResult = false;
            while (rs.next()) {

                // Check to ensure only one row is processed
                if (foundResult) {
                    throw new AuthenticationException("More than one user row found for user [" + username + "]. Usernames must be unique.");
                }

                result[0] = rs.getString(1);     //索引查询
                if (returningSeparatedSalt) {
                    result[1] = rs.getString(2);
                }

                foundResult = true;
            }
```

**new SimpleAuthenticationInfo()(存放唯一认证) 源码分析**

principal: 整个Shiro中唯一的标识符,可以存用户名,也可以存ID

credentials: 唯一标识符的密码

realmName: 当前数据源的名字

```java
public SimpleAuthenticationInfo(Object principal, Object credentials, String realmName) {
        this.principals = new SimplePrincipalCollection(principal, realmName);
        this.credentials = credentials;
    }
```


![20180122131153906](C:\Users\Administrator\Desktop\20180122131153906.png)

使用了工厂模式来对SecurityManager进行生成和配置  

生成过程是使用**单例+工厂** 

提供对外的**工具类**来使用，包含获取SecurityManager的方法和获取Subject的方法  


![3](C:\Users\Administrator\Desktop\3.png)

(代码略)

subject的使用是通过传入AuthenticationToken接口（注意是接口，其实扩展接口rememnverMeaut…和HostAutho…），

该接口目前的实现类是UserPasswordToken，当然也可以自己扩展实现自定义的认证Token 

#### 测试加密算法 ####

**盐值加密如果几个人密码一样，那么加密后的密码则一致。这样不安全，要解决这个问题，可以在密码上加盐。一般会选择不重复的值作为盐值，例如 用户名。**(部分代码)

```java
//构造方法：
        //第一个参数：散列算法
        //第二个参数：明文，原始密码
        //第三个参数：盐，通过使用随机数
        //第四个参数：散列的次数，比如散列两次，相当 于md5(md5(''))
        SimpleHash simpleHash = new SimpleHash("md5", source, salt, hashIterations);
        String md5 =  simpleHash.toString();
        System.out.println(md5);
```

```java
shiro-realm-md5.ini
---------------------
[main]
定义凭证匹配器
credentialsMatcher=org.apache.shiro.authc.credential.HashedCredentialsMatcher
散列算法
credentialsMatcher.hashAlgorithmName=md5
散列次数
credentialsMatcher.hashIterations=1024
开启加盐（无需设置，realm中使用的SimpleAuthenticationInfo 是 SaltedAuthenticationInfo 接口的实现类，默认开启的加盐功能）
credentialsMatcher.hashSalted=true
自定义 realm
customRealm=com.qfedu.shirodemo.realm.CustomRealmMd5
customRealm.credentialsMatcher=$credentialsMatcher
将realm设置到securityManager，相当 于spring中注入
securityManager.realms=$customRealm
```



#### 授权流程原理 ####

**授权**

授权，也叫访问控制，即在应用中控制谁能访问哪些资源（如访问页面/编辑数据/页面操作等）。在授权中需了解的几个关键对象：主体（Subject）、资源（Resource）、权限（Permission）、角色（Role）。

**主体（Subject）**

主体，即访问应用的用户，在Shiro中使用Subject代表该用户。用户只有授权后才允许访问相应的资源。

**资源**

在应用中用户可以访问的任何东西，比如JSP 页面、某些数据、某个业务方法等等都是资源。用户只要授权后才能访问。

**角色**

角色代表了操作集合，可以理解为权限的集合，一般情况下我们会赋予用户角色而不是权限，即这样用户可以拥有一组权限，赋予权限时比较方便。

典型的如：项目经理、技术总监、CTO、开发工程师等都是角色，不同的角色拥有一组不同的权限。

**权限**

权限表示在应用中用户能不能访问某个资源，

如：访问用户列表页面查看/新增/修改/删除用户数据（即很多时候都是CRUD（增查改删）式权限控制）打印文档等等。。。

##### 判断是否授权的方式 #####

Shiro 支持三种方式的授权判断：

**编程式**

通过写if/else 授权代码块完成：

Subject subject = SecurityUtils.getSubject();

if(subject.hasRole(“admin”)) {

​      //有权限

} else {

​     //无权限

}

**注解式**

通过在执行的Java方法上放置相应的注解完成：

@RequiresRoles("admin")

public void hello() {

   //有权限

}

没有权限将抛出相应的异常；

**JSP 标签**

在JSP 页面通过相应的标签完成：

```javas
<shiro:hasRole name="admin">

<!— 有权限—>

< /shiro:hasRole >

```



##### 自定义realm授权 #####

**从认证的realm拷贝，改变继承的抽象父类，添加新的方法**

 



### 5.程序分析 ###

**程序分析：从应用程序角度的来观察如何使用Shiro完成工作** 

1. 应用代码通过Subject来进行认证和授权，而Subject又委托给SecurityManager； 
2. 我们需要给Shiro的SecurityManager注入Realm，从而让SecurityManager能得到合法的用户及其权限进行判断。
3. 可以看到：应用代码直接交互的对象是Subject，也就是说Shiro的对外API核心就是Subject； 

![img](file:///C:/Users/Administrator/Documents/My%20Knowledge/temp/e66c475a-c763-4097-85a0-b0219938fe7f/128/index_files/4a5f51ff-ef16-4a61-a834-9a9b06da850b.jpg)



**Shiro内部结构**

![img](file:///C:/Users/Administrator/Documents/My%20Knowledge/temp/1b7bc2c3-9f2c-4ea2-8ed7-97820c205482/128/index_files/9b959a65-799d-396e-b5f5-b4fcfe88f53c.png)







**详细原理深入和运用:http://jinnianshilongnian.iteye.com/blog/2018398**



### 6.Shiro认证技巧整理 ###



#### 工具类接口的使用 ####

建立一个工具类接口Constants,以常量字符串的方式,专门存放Shiro中自定义的**标识符**

```
public interface Constants {
    // md5(用户密码+PASSWORD_SALT_KEY)保存到数据库中。
    String PASSWORD_SALT_KEY = "Shiro.admin.2017";
    //Shiro的session中存放用户的key
    String SESSION_USER_KEY = "SESSION_USER_KEY";
    //redis中存放的用户权限菜单的key
    String SESSION_USER_MANU = "SESSION_USER_MANU";
    //Shiro存放的角色信息
    String SESSION_USER_ROLE = "SESSION_USER_ROLE";
}
```

**接口工具类的思路不仅限于Shiro,灵活的定义接口,将冗余和容易混淆的部分抽离出来统一管理,可以极大的提高开发和维护的效率**





#### 认证优化技巧 ####

Controller层登录方法中,接收到用户名和密码后先进行

```
Subject currentUser = SecurityUtils.getSubject();
if (!currentUser.isAuthenticated()) {
    ...认证
}
```

直接获取subject,先进性判断该用户是否认证过,如果认证过则直接跳出即可

如果没有认证过,再进入认证环节

该逻辑可以减少服务器和数据库的压力,提高服务器的并发能力



#### ..shiro.xml 拦截器设置 ####

**Shiro主过滤器本身功能十分强大,其强大之处就在于它支持任何基于URL路径表达式的、自定义的过滤器的执行**

-  /login.html=anon 静态资源的方式屏蔽过滤器
- /**=authc 该路径下需要认证才能访问
- ...

过滤器的完整参考：

<http://blog.csdn.net/jadyer/article/details/12172839>



#### 登录认证使用原理 ####

**动态权限控制**

**RBAC（Role-Based Access Control ）基于角色的访问控制**

1. 配置好环境和工具类
2. 自定义Realm和异常
3. service中添加通过用户名查找用户信息
4. 在Controller层认证登录
5. - UsernamePasswordToken token = new UsernamePasswordToken(name, password);
   - Subject subject = SecurityUtils.getSubject();
   - ubject.login(token);
6. 将真正的验证交给封装的底层-->AuthenticationToken实现.(自定义Realm中)
7. 通过此时token的username去数据库查询用户信息
8. 用户信息存在,则存入SimpleAuthenticationInfo,否则  抛出用户对应的异常



**Shiro的分布式认证结构(shiro认证将账号密码的比较环节封装到AuthenticationToken中)**

​	Realm放在Controller中,在分布式中Controller使用Dubbo服务端接口,而dubbo接口通过service实现类来发布,这个角度看realm与dao隔层交互设计不太合理

​	验证成功则返回SimpleAuthenticationInfo(存放唯一标识(id或者username),密码,Realm名



### 7.Shiro授权技巧整理 ###

1. 通过用户登录的唯一标识principals 查找到用户有哪些菜单权限(ID)
2. 将这些ID存到SimpleAuthorizationInfo中
3. 在自定义ShiroFilterFactoryBean中获取所有菜单列表,并将id加入到section中
4. 底层自动对比,哪些ID用户有,则允许访问,没有的话"authc"拦截
5. Controller中查询用户拥有的菜单数据返回前段即可,此时没有权限的数据已经被拦截

#### 易错集合 ####

##### 1.授权URL #####

​	注意使用Shiro权限设置后的url如果需要访问,逻辑路径需要放在 **前端** 拼接,(后台对逻辑路径没有识别,也没有必要识别,不能将逻辑路径放在后台和数据库中)

##### 2.权限顺序 #####

​	运行时先执行**MyRealm**中的权限,然后拼接**MyShiroFilterFactoryBean**中的权限

​	注意:限制范围较大的往后排,特别是全部拦截的/** 如果需要的话尽量放在**MyShiroFilterFactoryBean**中




### 8.Remember me功能简述 ###

Shiro的Remember Me可以很轻松的实现自动登录的功能,方便快捷

#### 实现过程 ####

```
UsernamePasswordToken token = new UsernamePasswordToken(username, password);
		Subject subject = SecurityUtils.getSubject();
		if(loginForm.getRememberMe() != null && "Y".equals(loginForm.getRememberMe())){
			token.setRememberMe(true);	
		}
		subject.login(token);
```

```
/** = user
```

Remember Me只需要在登录时将token的RememberMe功能开启,本来的拦截级别为/ ** = authc 将拦截设置(降级)为**user级别**即可使用

Remember Me功能开启使用后,Shiro会生成一个叫**RememberMe**的Cookie保存在浏览器中,当subject.loginout退出或者过期后失效,改参数是base64加密的字符串如下

```
名称：	rememberMe
内容：	6gYvaCGZaDXt1c0xwriXj/Uvz6g8OMT3VSaAK4WL0Fvqvkcm0nf3CfTwkWWTT4EjeSS/EoQjRfCPv4WKUXezQDvoNwVgFMtsLIeYMAfTd17ey5BrZQMxW+xU1lBSDoEM1yOy/i11ENh6eXjmYeQFv0yGbhchGdJWzk5W3MxJjv2SljlW4dkGxOSsol3mucoShzmcQ4VqiDjTcbVfZ7mxSHF/0M1JnXRphi8meDaIm9IwM4Hilgjmai+yzdVHFVDDHv/vsU/fZmjb+2tJnBiZ+jrDhl2Elt4qBDKxUKT05cDtXaUZWYQmP1bet2EqTfE8eiofa1+FO3iSTJmEocRLDLPWKSJ26bUWA8wUl/QdpH07Ymq1W0ho8EIdFhOsELxM66oMcj7a/8LVzypJXAXZdMFaNe8cBSN2dXpv4PwiktCs3J9P9vP4XrmYees5x27UmXNqYFk86xQhRjFdJsw5A9ctDKXzPYvJmWFouo3qT5hugX0uxWALCfWg8MHJnG9w7QgVKM8oy3Xy4Ut8lSvYlA==
```

Shiro的RememberMe设计时!=已经登录,因为该cookie被序列化后可以不同的浏览器之间访问,并且可能被黑客复制截取等,因此使用该功能的话尽量以非关键性资源为主,当牵扯到**资金**等关键资源时,选择再次登录即可

开发时如果使用Session域对象,则自动登录后Session中不会再有数据,如果需要用到,那么需要重写isAccessAllowed 方法

**详细参考自:https://blog.csdn.net/nsrainbow/article/details/36945267/**

**(本次Remember Me尚未指定配置更改,待更新)**




