---
title: Spring Framework 原理解析
author: Zan Wang
date: 2021-09-18 15:32:39
tags: [Java,Spring]
categories: [Java,Spring]
---
## Spring是什么？

> Spring框架是一个**开放源代码**的**J2EE应用程序框架**，由**Rod Johnson**发起，是针对bean的**生命周期**进行管理的**轻量级容器**。 Spring解决了开发者在J2EE开发中遇到的许多常见的问题，提供了功能强大**IOC**、**AOP**及**Web MVC**等功能。Spring框架主要由七部分组成，分别是 **Spring Core**、 **Spring AOP**、 **Spring ORM**、 **Spring DAO**、**Spring Context**、 **Spring Web**和 **Spring Web MVC**。

## 为什么要用Spring?

> "Spring makes programming Java quicker, easier, and safer for everybody. "

以上是官方给出的理由，翻译过来也非常直白，即Spring能让我们开发Java程序变得更加**快速**、简洁和**安全**。例如在日常开发中，我们可以：

- 用Spring的@Autowired，@Resource标签来获取类实例时，而不需要一层一层的传递这些实例。
- 用各种操作日志、数据库事务AOP来增强代码功能，而不需要不停的修改业务代码。
- 用JPA来进行数据库操作，而不需要重复的编写一条条的SQL语句。

总的来说，Spring的确做到了它承诺的一切。

## Spring的体系结构

【图】


那么，拥有上面这些功能的Spring是否是一款超级大的三方件？答案是否。

有意思的是，Spring本身就是为了解决企业应用程序开发的复杂性而创造的，它的基因决定了它一定不会奔着“大而全”而去。

实际上，Spring的哲学是《最少的侵入》。
从上面几个例子来看，你会发现，Spring实现的这些功能会有许多不同的实现方案，例如容器技术就有历史悠久的EJB，数据库连接就有我们常用的mybatis,Spring Boot对于有固定模板的我们公司来说，压根没用。但同时，我们还会发现，我们可以很轻易的替换掉Spring的功能，因为其Spring的代码侵入量极少。例如Bean管理只需要在配置文件或者类数据定义时编码即可，随时可以自己new一个对象。新版本更是可以通过构造函数来避免使用注解。
非侵入式，即允许在应用系统中自由选择和组装Spring框架的各个功能模块，并且不强制要求应用系统的类必须继承或实现Spring框架的类和接口来达到使用框架的目的，使得所开发出来的应用系统能够在不同的运行环境或平台上自由移植，仅需要进行适配修改，而不需要修改核心代码。


Spring的体系结构



Spring如何实现非侵入式
引用反射机制，通过动态调用的方式来提供各方面的功能，建立核心组件BeanFactory
配合使用Spring框架中的BeanWrapper和BeanFactory组件类最终达到对象的实例创建和属性注入

## Tomcat加载Spring MVC和Spring容器的过程
> 参考资料：https://www.cnblogs.com/top-housekeeper/p/14105297.html#_label2


一个典型包含spring MVC框架的webapp配置如下所示：

```xml
<!--以下为加载Spring需要的配置-->
<!--Spring配置具体参数的地方-->
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>
    classpath:applicationContext.xml
  </param-value>
</context-param>
<!--Spring启动的类-->
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

 <!--以下为加载SpringMVC需要的配置-->
<servlet>
  <servlet-name>project</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <load-on-startup>1</load-on-startup>   <!--servlet被加载的顺序，值越小优先级越高（正数）-->

  <servlet-mapping>
        <servlet-name>project</servlet-name>
        <url-pattern>*.html</url-pattern>
  </servlet-mapping>
</servlet>
```
我们知道，tomcat可以容纳多个webapp，每一个webapp对一个servlet，
tomcat根据所有的web.xml配置形成一个索引，根据url进行匹配并发送到对应的servlet，例如在上面的url-pattern为*.html。那么当tomcat接收到对应请求时，便会将请求发送到名为project的servlet。

从tomcat的生命周期我们可以知道，上面示例中首先会被加载的是ContextLoaderListener，其init方法为
```java
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    //省略其他方法

    /**
     * Initialize the root web application context.
     */
    @Override
    public void contextInitialized(ServletContextEvent event) {
        initWebApplicationContext(event.getServletContext());
    }
}
```
跟踪代码我们可以发现，ContextLoaderListener初始化了一个WebApplicationContext，这便是我们的根applicationContext。