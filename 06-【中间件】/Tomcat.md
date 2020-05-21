# 类加载机制
既然Tomcat的类加载机器不同于双亲委派模式，那么它又是一种怎样的模式呢？[官网链接](https://tomcat.apache.org/tomcat-8.5-doc/class-loader-howto.html)有对此进行描述。
另外，我们还是先来看看Tomcat整体的类加载图是怎样的吧~

![image-20191205145542023](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20191205145545-981879.png) 

我们在这张图中看到很多类加载器，除了Jdk自带的类加载器，我们尤其关心Tomcat自身持有的类加载器。仔细一点我们很容易发现：Catalina类加载器和Shared类加载器，他们并不是父子关系，而是兄弟关系。为啥这样设计，我们得分析一下每个类加载器的用途，才能知晓。

- Common类加载器，负责加载Tomcat和Web应用都复用的类
- Catalina类加载器，负责加载Tomcat专用的类，而这些被加载的类在Web应用中将不可见
- Shared类加载器，负责加载Tomcat下所有的Web应用程序都复用的类，而这些被加载的类在Tomcat中将不可见
- WebApp类加载器，负责加载具体的某个Web应用程序所使用到的类，而这些被加载的类在Tomcat和其他的Web应用程序都将不可见
- Jsp类加载器，每个jsp页面一个类加载器，不同的jsp页面有不同的类加载器，方便实现jsp页面的热插拔

`CATALINA_HOME`和`CATALINA_BASE`在tomcat官网上有对他们进行描述，[官网链接](https://tomcat.apache.org/tomcat-8.5-doc/introduction.html#CATALINA_HOME_and_CATALINA_BASE)。简单地说，`CATALINA_HOME`指的是安装目录，`CATALINA_BASE`指的是工作目录。在一台物理机上面，可以只有一个安装目录，但是允许有多个工作目录。

# Servlet
**HttpServlet为什么要实现serializable？**  

javaEE规范要求 当服务器出现意外情况而”down“了 可以将缓存中的数据写入硬盘 当重启的时候可以恢复当时状态,由此可见Servlet实例是需要被存储的，所以建议实现序列化。

# HTTP/AJP协议
Tomcat在`server.xml`中配置了两种连接器。 

`HTTP Connector`  

拥有这个连接器，Tomcat才能成为一个web服务器，但还额外可处理Servlet和jsp。

`AJP Connector`   

AJP连接器可以通过AJP协议和另一个web容器进行交互。

配置:
```xml
<!-- Define a non-SSL/TLS HTTP/1.1 Connector on port 8080 -->
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />

<!-- Define an AJP 1.3 Connector on port 8009 -->
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```
第一个连接器监听8080端口，负责建立HTTP连接。在通过浏览器访问Tomcat服务器的Web应用时，使用的就是这个连接器。

第二个连接器监听8009端口，负责和其他的HTTP服务器建立连接。在把Tomcat与其他HTTP服务器集成时，就需要用到这个连接器。AJP连接器可以通过AJP协议和一个web容器进行交互。

Web客户访问Tomcat服务器的两种方式:
![image-20191205145617744](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20191205145619-431113.png) 