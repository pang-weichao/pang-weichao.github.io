---
layout: post
title: "mybatis和hibernate的一点总结"
date: 2017-07-13
excerpt: "对于mybatis和hibernate两个orm层框架的粗总结"
tags: [java, hibernate, mybatis]
comments: true
---

### orm框架
orm(Object Relational Mapping)对象关系映射。我们的java应用程序是对象型而我们的数据库却是关系型数据库，那么我们的java应用和我们的数据库之间的数据交互一定存在着数据类型转换。而我们的orm框架就是实现java应用和数据库之间的数据交互功能，因为它是将我们存储在内存中的pojo对象的数据映射存入数据库中，所以orm框架我们又可以说是持久化框架。

***

### hibernate&mybatis
我们从最大粒度的角度去看hibernate和mybatis分别在ssh和ssm项目中的配置，其实两者差的不是很多，都是需要配置一个sessionfactory，然后通过这个factory去创建sqlsession，而sqlsession就是框架提供给我们的执行sql语句的顶层api。只是两者在底层实现上的不同导致了配置细节上的不同，hibernate创建sessionfactory是通过配置LocalSessionFactoryBean来实现，而mybatis是通过配置SqlSessionFactoryBean来创建sessionfactory。具体配置如下：
```
    <!--hibernate配置sessionFactory-->
    <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="packagesToScan">
            <value>cn.pwc.demo.entity</value>
        </property>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.format_sql">true</prop>
                <prop key="hibernate.use_sql_comments">true</prop>
            </props>
        </property>
    </bean>
```
```
    <!--mybatis配置sessionFactory-->
    <bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="mapperLocations" value="classpath:/mybatisXml/*.xml"/>
        <property name="configLocation" value="classpath:/mybatis-config.xml"/>
        <property name="typeAliasesPackage" value="cn.pwc.demo.entity"/>
    </bean>
```
当然除了配置sessionfactory之外我们还需要配置数据源，因为orm框架就是和数据库打交道的，他怎么能不知道数据库在哪呢。而dataSource的实现可以用c3p0,druid等来创建连接池，以减少频繁的创建和销毁连接对象而造成的资源损失。除此之外，对于数据库的操作，我们需要配置事务管理，只是在相应框架中的具体实现上不同而已。

***

### 总结
对于框架的使用上来说，我们只是需要配置好框架提供给我们的操作数据库的顶层api和设置相关参数就好，其他的比如事务的提交，sql语句的执行等等，都交给框架就行，对于我们而言可以说是透明的。
对于整个项目来说，这里就可以体会到mvc模式的解耦带来的便利，我们可以轻易的更换数据持久层的框架，而不用动其他地方的代码。

[ssh源码地址](https://github.com/pang-weichao/ssh)

[ssm源码地址](https://github.com/pang-weichao/ssm)