[https://cloud.tencent.com/developer/article/2308888](https://cloud.tencent.com/developer/article/2308888)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746151889060-aad3d9e1-6935-4d29-94d4-bc0c622fb914.png)

spring 简介

Spring是分层的 Java SE/EE full-stack 轻量级开源框架，以IOC（Inverse of Control，控制反转）和AOP（Aspect Oriented Programming，面向切面编程）为内核，使用基本的JavaBean完成以前只可能由EJB完成的工作，取代了EJB臃肿和低效的开发模式。

在实际开发中，通常服务器端采用分层架构——表现层（web）、业务逻辑层（service）、持久层（dao）。

Spring对每一层都提供了技术支持，在表现层提供了与Struts2框架的整合，在业务逻辑层可以管理事务和记录日志，在持久层可以整合Hibernate和JdbcTemplate等技术。

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746152254037-650d52d4-ba73-451d-a102-4b5c5af5f471.png)

# IOC代码推到
DAO

User接口和实现

```plain
package com.musical.dao;

public interface UserDao {
    void getUser();

}

```

```plain
package com.musical.dao;

public class UserDaoImpl implements UserDao {
    public void getUser() {
        System.out.println("获得用户数据");
    }
}


```

service层

```plain
public interface UserService {
    void getUser();

}
```

```plain
package com.musical.service;

import com.musical.dao.UserDao;
import com.musical.dao.UserDaoImpl;

public class UserServiceImpl {
        private UserDao userDao = new UserDaoImpl();

        public void getUser(){
            userDao.getUser();

        }
    }

```

业务层想要实现getUser方法 通过 实例化dao层对象，然后去调用

用户

```plain
import com.musical.service.UserService;
import com.musical.service.UserServiceImpl;

public class Mytest {
    public static void main(String[] args) {
        //用户调用业务层，不调用dao层
        UserService userService = new UserServiceImpl();
        userService.getUser();
    }
}
```

### 面临问题
增加一个实现dao Mysql的话

```plain
package com.musical.dao;

public class UserDaoMysqlImpl implements UserDao{
    public void getUser(){
        System.out.println("Mysql获取数据");
    }
}
```

service层想要切换到mysql

```plain
package com.musical.service;

import com.musical.dao.UserDao;
import com.musical.dao.UserDaoImpl;
import com.musical.dao.UserDaoMysqlImpl;

public class UserServiceImpl implements UserService {
        //换了实例化的接口

        private UserDao userDao = new UserDaoMysqlImpl();

        public void getUser(){
            userDao.getUser();

        }
    }
```

要么只能多写一个实例化

```plain
package com.musical.service;

import com.musical.dao.UserDao;
import com.musical.dao.UserDaoImpl;
import com.musical.dao.UserDaoMysqlImpl;

public class UserServiceImpl implements UserService {
        private UserDao userDao = new UserDaoImpl();
        private UserDao userDaoMysql = new UserDaoMysqlImpl();

        public void getUser(){
            userDao.getUser();

        }
        public void getUserMysql(){
            userDaoMysql.getUser();

        }
    }
```

同时用户那边也要多写一个

```plain
import com.musical.service.UserService;
import com.musical.service.UserServiceImpl;

public class Mytest {
    public static void main(String[] args) {
        //用户调用业务层，不调用dao层
        UserService userService = new UserServiceImpl();
        UserService userServiceMysql = new UserServiceImpl();
        userService.getUser();
        userServiceMysql.getUser();
    }
}
```

正常的写法想到同时实现调用USer和sql服务的化只能选择多写一倍的代码

客户的需求导致我们要改原有的代码

### ioc出现
ioc 程序不动 客户操作

让用户自己去实例化对象，我们业务层只负责调用对象的方法

service不实例化dao层对象 只是通过set方法去接受用户实例化后传入他想实例化的对象，然后去调用该对象的代码

xml实现然后通过 实例化ApplicantsContext getbean获得对象

对象由spring 创建 管理 

bean.xml

```plain
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- new UserDaoMysqlImpl-->
    <bean id="mysqlImpl" class="com.musical.dao.UserDaoMysqlImpl"/>

    <bean id="userServiceImpl" class="com.musical.service.UserServiceImpl" >
        <!--
         ref 引用容器创建好的对象
         set方法
         -->
        <property name="userDao" ref="mysqlImpl"/>


    </bean>

</beans>
```

实现

```plain
import com.musical.service.UserServiceImpl;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class Mytest {
    public static void main(String[] args) {
        //用户调用业务层，不调用dao层
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserServiceImpl userServiceImpl = (UserServiceImpl) context.getBean("userServiceImpl");
        userServiceImpl.getUser();

    }
}
```

## ioc创建对象的方式
使用无参构造

```plain
    <bean id="userServiceImpl" class="com.musical.service.UserServiceImpl" >
        <!--
         ref 引用容器创建好的对象
         set方法
         -->
        <property name="userDao" ref="mysqlImpl"/>


    </bean>
```



有参构造方法

```plain
<!--下标赋值 -->
<bean id="user" class="com.musical.pojo.User">
        <constructor-arg index="0" value="Muscial有参构造"/>
</bean>
```

type构造

```plain
<bean id="user" class="com.musical.pojo.User">
    <constructor-arg type="java.lang.String" value="type赋值"/>
    
</bean>
```

name构造

```plain
<bean id="user" class="com.musical.pojo.User">
    <constructor-arg name="name" value="name构造"/>
</bean>
```

## 注入方式
set

```plain
    <bean id="userServiceImpl" class="com.musical.service.UserServiceImpl" >
        <property name="userDao" ref="mysqlImpl"/>


    </bean>
```

p命名注入 省掉了property 直接注入属性 无参构造

```plain
<bean id="user" class="com.musical.pojo.User" p:name="P命名注入"/>
```

c命名注入 有参构造器注入 省掉了construct 

```plain
            <bean id="user" class="com.musical.pojo.User" c:name="c命名注入"/>
```



# AOP
### 静态代理
理解 给一个方法 设置一个中间层，实现的时候可以去添加功能(前置日志)，而不是在源码上改

租房子模型

```plain
package com.muscial.demo1;
//租房
public interface Rent {
    public void rent();
}
```

房东host提供房子

```plain
package com.muscial.demo1;

//房东
public class Host implements Rent {
    public void rent(){
        System.out.println("房东要出租房子");
    }

}
```

客户找中介租房子

```plain
package com.muscial.demo1;

public class Client {
    public static void main(String[] args) {
        Host host = new Host();
//      host.rent();
        Proxy proxy = new Proxy(host);
        proxy.rent();


    }
}
```

中介在房东租房的前提下 新加了看房字的功能

```plain
package com.muscial.demo1;

public class Proxy implements Rent {
    private Host host;
    public Proxy(){

    }
    public Proxy(Host host) {
        this.host = host;
    }
    public void rent(){
        seeHouse();
        host.rent();

    }
    //看房
    public void seeHouse(){
        System.out.println("中介带你看房");

    }
}
```

代理模式的好处:

可以使真实角色的操作更加纯粹!不用去关注一些公共的业务

公共也就就交给代理角色!实现了业务的分工!

公共业务发生扩展的时候,方便集中管理!

缺点:

一个真实角色就会产生一个代理角色;代码量会翻倍-开发效率会变低

### 动态代理
不同的UserService 需要同时添加前置日志功能 这个时候这么办 是一个UserService写一个代理的构造方法

面临的问题 每次添加一个对象 都要代码量翻倍

多一个对象需要代理类去代理 代理类就得多一次 去实现这个类的构造方法

解决的问题是切面性的

```plain
package com.muscial.demo1;

public class Proxy implements Rent {
    private Host host;
    private Famliy famliy;
    public Proxy(){

    }
    public Proxy(Famliy famliy) {
        this.famliy = famliy;
    }
    public Proxy(Host host) {
        this.host = host;
    }
    public void rent(){
        seeHouse();
        host.rent();

    }
    //看房
    public void seeHouse(){
        System.out.println("中介带你看房");

    }
}
```



动态代理的代理类是动态生成的 不是我们直接写好的

这个时候 反射就来了

Proxy代理 InvocationHander

```plain
package com.demoproxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;


public class ProxyInvocationHander implements InvocationHandler{
    private Object target;
    //设置代理的接口 Clinet 传入

    public void setTarget(Object target) {
        this.target = target;
    }
    //生成得到代理类
    public Object getProxy(){
       return Proxy.newProxyInstance(this.getClass().getClassLoader(), target.getClass().getInterfaces(),this);
    }



    //处理代理示例 返回结果
    public Object invoke(Object Proxy , Method method,Object[] args) throws  Throwable{
        //使用反射
        sout("前置日志");
        Object result = method.invoke(target, args);
        return result;

    }

}
```

好处就是 代理类代码不变 通过set方法 传入实例化 想要代理的对象 代理类通过反射 获得代理类和处理代理实例的方法 类似IOC

 至于为什么会实现invoke 方法取代 直接调用反射出来的方法

1. **<font style="color:rgb(64, 64, 64);">动态代理类的生成</font>**<font style="color:rgb(64, 64, 64);">  
</font><font style="color:rgb(64, 64, 64);">当通过</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">Proxy.newProxyInstance()</font>**`<font style="color:rgb(64, 64, 64);">创建代理对象时，JVM会</font>**<font style="color:rgb(64, 64, 64);">动态生成一个代理类</font>**<font style="color:rgb(64, 64, 64);">，该类继承</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">Proxy</font>**`<font style="color:rgb(64, 64, 64);">并实现指定的接口。例如，若接口有方法</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">doSomething()</font>**`<font style="color:rgb(64, 64, 64);">，代理类会生成对应的实现方法。</font>

**<font style="color:rgb(64, 64, 64);">代理类的方法逻辑</font>**<font style="color:rgb(64, 64, 64);">  
</font><font style="color:rgb(64, 64, 64);">代理类中所有接口方法的重写逻辑类似以下伪代码：</font>

2. <font style="color:rgb(82, 82, 82);background-color:rgb(250, 250, 250);">java</font><font style="color:rgb(82, 82, 82);background-color:rgb(250, 250, 250);">复制</font><font style="color:rgb(82, 82, 82);background-color:rgb(250, 250, 250);">下载</font>

```plain
public final ReturnType doSomething(Args...) {
    return handler.invoke(
        this,                // 代理对象自身
        method,              // 目标方法的反射对象（如doSomething）
        args                 // 方法参数
    );
}
```

<font style="color:rgb(64, 64, 64);">每个方法调用都会委托给</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">InvocationHandler</font>**`<font style="color:rgb(64, 64, 64);">实例的</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">invoke</font>**`<font style="color:rgb(64, 64, 64);">方法。</font>

3. **<font style="color:rgb(64, 64, 64);">InvocationHandler的作用</font>**<font style="color:rgb(64, 64, 64);">  
</font><font style="color:rgb(64, 64, 64);">开发者通过实现</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">InvocationHandler</font>**`<font style="color:rgb(64, 64, 64);">的</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">invoke</font>**`<font style="color:rgb(64, 64, 64);">方法，可以</font>**<font style="color:rgb(64, 64, 64);">拦截代理对象的方法调用</font>**<font style="color:rgb(64, 64, 64);">。在示例中：</font>
    - `**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">method.invoke(target, args)</font>**`<font style="color:rgb(64, 64, 64);">通过反射调用真实对象的方法。</font>
    - <font style="color:rgb(64, 64, 64);">在调用前后可插入自定义逻辑（如日志、事务等），实现AOP。</font>

### AOP实现
Spring依靠配置实现动态代理的一种方式

几个概念

+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">Aspect：切面，即一个横跨多个核心逻辑的功能，或者称之为系统关注点；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">Joinpoint：连接点，即定义在应用程序流程的何处插入切面的执行；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">Pointcut：切入点，即一组连接点的集合；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">Advice：增强，指特定连接点上执行的动作；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">Introduction：引介，指为一个已有的Java对象动态地增加新的接口；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">Weaving：织入，指将切面整合到程序的执行流程中；</font>
+ <font style="color:rgb(31, 41, 55);background-color:#FBDE28;">Interceptor：拦截器，是一种实现增强的方式；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">Target Object：目标对象，即真正执行业务的核心逻辑对象；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">AOP Proxy：AOP代理，是客户端持有的增强后的对象引用。</font>

依赖

```plain
<dependencies>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.4</version>
    </dependency>
</dependencies>
```

#### SpringAPI实现
定义切入函数

```plain
package com.musical.log;

import org.springframework.aop.MethodBeforeAdvice;

import java.lang.reflect.Method;

public class log implements MethodBeforeAdvice {

    //method 要执行的目标对象方法
    //orgs 参数
    //target:目标对象
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println(target.getClass().getName()+"的"+method.getName()+"被执行了");
    }
}
```



applicationContext实现配置

```plain
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">
<!--    注册bean-->
    <bean id="userService" class="com.musical.service.UserServiceImpl"/>
    <bean id="log" class="com.musical.log.log" />
<!--    aop 配置 导入约束-->
    <aop:config>
<!--        切入点-->
<!--        expring-->
        <aop:pointcut id="pointcut" expression="execution(* com.musical.service.UserServiceImpl.*(..))"/>
<!--        执行环绕增强-->
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>


    </aop:config>

</beans>
```



#### 注解实现
<font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">定义一个</font>`<font style="color:rgb(75, 85, 99);background-color:rgb(229, 231, 235);">LoggingAspect</font>`<font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">：</font>

```java
@Aspect
@Component
public class LoggingAspect {
    // 在执行UserService的每个方法前执行:
    @Before("execution(public * com.itranswarp.learnjava.service.UserService.*(..))")
    public void doAccessCheck() {
        System.err.println("[Before] do access check...");
    }

    // 在执行MailService的每个方法前后执行:
    @Around("execution(public * com.itranswarp.learnjava.service.MailService.*(..))")
    public Object doLogging(ProceedingJoinPoint pjp) throws Throwable {
        System.err.println("[Around] start " + pjp.getSignature());
        Object retVal = pjp.proceed();
        System.err.println("[Around] done " + pjp.getSignature());
        return retVal;
    }
}
```

`<font style="color:rgb(75, 85, 99);background-color:rgb(229, 231, 235);">@Before</font>`<font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">注解，后面的字符串是告诉AspectJ应该在何处执行该方法，这里写的意思是：执行</font>`<font style="color:rgb(75, 85, 99);background-color:rgb(229, 231, 235);">UserService</font>`<font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">的每个</font>`<font style="color:rgb(75, 85, 99);background-color:rgb(229, 231, 235);">public</font>`<font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">方法前执行</font>`<font style="color:rgb(75, 85, 99);background-color:rgb(229, 231, 235);">doAccessCheck()</font>`<font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">代码。</font>

```java
public UserServiceAopProxy extends UserService {
    private UserService target;
    private LoggingAspect aspect;

    public UserServiceAopProxy(UserService target, LoggingAspect aspect) {
        this.target = target;
        this.aspect = aspect;
    }

    public User login(String email, String password) {
        // 先执行Aspect的代码:
        aspect.doAccessCheck();
        // 再执行UserService的逻辑:
        return target.login(email, password);
    }

    public User register(String email, String password, String name) {
        aspect.doAccessCheck();
        return target.register(email, password, name);
    }

    ...
}
```

### <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">拦截器类型</font>
<font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">顾名思义，拦截器有以下类型：</font>

+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">@Before：这种拦截器先执行拦截代码，再执行目标代码。如果拦截器抛异常，那么目标代码就不执行了；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">@After：这种拦截器先执行目标代码，再执行拦截器代码。无论目标代码是否抛异常，拦截器代码都会执行；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">@AfterReturning：和@After不同的是，只有当目标代码正常返回时，才执行拦截器代码；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">@AfterThrowing：和@After不同的是，只有当目标代码抛出了异常时，才执行拦截器代码；</font>
+ <font style="color:rgb(31, 41, 55);background-color:rgb(249, 250, 251);">@Around：能完全控制目标代码是否执行，并可以在执行前后、抛异常后执行任意拦截代码，可以说是包含了上面所有功能。</font>



