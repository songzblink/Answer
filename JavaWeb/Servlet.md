# 1、Servlet接口

Servlet是运行在Web服务器的一个小型Java程序，它接受和响应来自Web客户端的请求（通常是通过HTTP协议）。

实现Servlet接口可以有两种方式：

- generic servlet，继承于javax.servlet.GenericServlet；
- HTTP servlet，继承于javax.servlet.http.HttpServlet。

Servlet接口定义了如下方法：

- init【生命周期方法】

  ```java
  /**
   * config 一个ServletConfig类，包含servlet的配置和初始化参数
   */
  void init(ServletConfig config)
  throws ServletException
  ```

  该方法实例化Servlet对象后**只执行一次**，由servlet容器调用，以向servlet指示该servlet正在投入使用。

- getServletConfig

  ```java
  ServletConfig getServletConfig()
  ```

  返回一个ServletConfig类，该类包含这个servlet的初始化参数和启动配置参数，这里返回的ServletConfig对象是传递给init()方法的对象。

- service【生命周期方法】

  ```java
  /**
   * req 客户端请求
   * res servlet响应
   * ServletException 如果发生异常，从而干扰Servlet的正常运行
   * IOException 如果输入或者输出异常发生
   */
  void service(ServletRequest req, ServletResponse res)
  throws ServletException, IOException
  ```

  由servlet容器调用，以允许servlet响应请求。

- getServletInfo

  ```java
  String getServletInfo()
  ```

  返回关于servlet的信息，例如作者、版本和版权等。

- destroy【生命周期方法】

  ```java
  void destroy()
  ```

  由servlet容器调用，以指示servlet该servlet正在退出服务。此方法仅被调用一次，仅在servlet的service方法中的所有线程都已退出或经过超时时间后，才调用此方法。

# 2、GenericServlet类

该抽象类的定义如下：

```java
public abstract class GenericServlet
extends Object
implements Servlet, ServletConfig, Serializable
```

GenericServlet让写一个servlet更加的简单，它提供了生命周期方法：init()方法、destroy()方法以及ServletConfig接口的一些方法的简单版本。GenericServlet可以被一个servlet直接继承，尽管扩展特定于协议的子类（例如HttpServlet）更为常见。

如果你想要的写一个generic servlet，你只需要覆写抽象方法service()即可。

下面列出几个GenericServlet类的方法：

- #### getInitParameter

  ```java
  public String getInitParameter(String name)
  ```

  此方法从servlet的ServletConfig类中获取名为name的参数的值，如果参数不存在则返回null。

- #### getInitParameterNames

  ```java
  public Enumeration<String> getInitParameterNames()
  ```

  此方法获取以Enumeration的方式从ServletConfig类中获取servlet所有初始化参数的名称，如果没有初始化参数则返回空的Enumeration。

- #### getServletConfig

  ```java
  public ServletConfig getServletConfig()
  ```

  返回初始化当前servlet的ServletConfig类（其包含配置信息）。

- #### getServletContext

  ```java
  public ServletContext getServletContext()
  ```

  返回对该servlet正在其中运行的ServletContext的引用。

# 3、HttpServlet类

该抽象类定义如下：

```java
public abstract class HttpServlet
extends GenericServlet
```

该抽象类的子类能够创建一个适用于网站的HTTP servlet，HttpServlet的子类必须**至少覆写一个**方法，通常是下面的方法中的一个：

- doGet()，为了servlet支持HTTP GET请求；
- doPost()，为了servlet支持HTTP POST请求；
- doPut()，为了servlet支持HTTP PUT请求；
- doDelete()，为了servlet支持HTTP DELETE请求；
- init()和destroy()，为了servlet支持管理在Servlet生命周期内保留的资源；
- getServletInfo()，为了servlet支持提供关于自身的信息。

此类中没有必要再去覆写service()方法，通过将标准HTTP请求分派到每种HTTP请求类型的处理程序方法（上面列出的doXXX方法）来处理它们。

# 4、ServletContext接口

接口定义如下：

```java
public interface ServletContext
```

该接口定义了一个方法集合，用于servlet和它所属的servlet容器间进行通信，例如获取一个MIME类型的文件，调度请求或者是写一个日志文件。

context，上下文。每个JVM虚拟机的每个Web应用都有一个上下文。

服务器会为每个应用创建一个ServletContext对象，一个应用只有一个ServletContext对象：

- ServletContext对象的创建是在服务器启动时完成的；
- ServletContext对象的销毁是在服务器关闭时完成的。

ServletContext对象的作用是在整个Web应用的动态资源之间共享数据，例如在AServlet中向ServletContext对象中保存一个值，然后BServlet中就可以获取这个值，这就是共享数据了。

## 4.1、如何获取ServletContext

ServletContext对象包含在ServletConfig对象中，该对象在初始化Servlet时由Web服务器提供。

有两种方法可以获取ServletContext：

- ServletConfig类的getServletContext()方法；

- GenericServlet类有getServletContext()方法。

**在Servlet中获取ServletContext对象：**

```java
public class MyServlet implements Servlet {
    public void init(SetvletConfig config) {
        ServletContext context = config.getServletContext();
    }
}
```

**在GenericServlet或HttpServlet中获取ServletContext对象：**

```java
public class MyServlet extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response) {
        ServletContext context = this.getServletContext();
    }
}
```

## 4.2、存取数据的功能

ServletContext是JavaWeb的四大域对象之一：

- PageContext；
- ServletRequest；
- HttpSession；
- ServletContext；

所有域对象都有存取数据的功能，因为域对象内部有一个Map，用来存储数据，下面是ServletContext对象用来操作数据的方法：

- #### setAttribute

  ```java
  void setAttribute(String name, Object object)
  ```

  在此ServletContext中把一个对象和一个属性名绑定。如果属性名已经存在绑定的对象，将会替换成新的对象。

- #### getAttribute

  ```java
  Object getAttribute(String name)
  ```

  从当前ServletContext中返回指定name的属性值，如果没有这个名称的属性则返回null。

- #### removeAttribute

  ```java
  void removeAttribute(String name)
  ```

  从当前ServletContext中删除指定name的属性值。

- #### getAttributeNames

  ```java
  Enumeration<String> getAttributeNames()
  ```

  返回当前ServletContext中的所有属性名称。

## 4.3、获取应用初始化参数

Servlet可以获取它的初始化参数，但这些参数是局部的参数，即一个Servlet只能获取它自己的初始化参数，不能获取别人的。

如果想要配置公共的初始化参数，为所有的Servlet所使用，这需要利用ServletContext来实现。

假设web.xml中有如下配置：

```xml
<web-app ...>
...
	<context-param>
    	<param-name>paramName1</param-name>
        <param-value>paramValue1</param-value>
    </context-param>
 	<context-param>
    	<param-name>paramName2</param-name>
        <param-value>paramValue2</param-value>
    </context-param>   
</web-app>
```

那么我们可以在HTTPServlet中利用如下代码实现：

```java
ServletContext context = this.getServletContext(); // 获取ServletContext对象
String value1 = context.getInitParameter("paramName1"); // 通过参数名获取参数值
String value2 = context.getInitParameter("paramName2");
System.out.println(value1 + ", " + value2);

Enumeration names = context.getInitParameterNames(); // 获取所有应用初始化参数名称
while(names.hasMoreElements()) {
    System.out.println(names.nextElement());
}
```

## 4.4、获取资源相关的方法

### 4.4.1、获取真实路径

利用getRealPath()方法可以实现，定义如下

```java
String getRealPath(String path)
```

获取与给定虚拟路径相对应的真实路径。

假设当前目录结构如下：

```
WebRoot
	|- META-INF
	|- WEB-INF
		-lib
			-b.txt
	|- a.txt
```

**获取a.txt的真实路径:**

```java
String realPath = servletContext.getRealPath("/a.txt")
```

realPath的值为a.txt文件的绝对路径：F:/tomcat6/webapps\hello\a.txt；

**获取b.txt的真实路径：**

```java
String realPath = servletContext.getRealPath("/WEB-INF/b.txt")
```

### 4.4.2、获取资源流

把资源以输入流的方式获取，利用getResourceAsStream()方法可以实现，定义如下：

```java
InputStream getResourceAsStream(String path)
```

返回位于指定路径处的资源作为InputStream对象。

**获取a.txt资源流：**

```java
InputStream in = servletContext.getResourceAsStream("/a.txt")；
```

**获取b.txt资源流：**

```java
InputStream in = servletContext.getResourceAsStream("/WEB-INF/b.txt")；
```

### 4.4.3、获取指定目录下所有资源路径

利用getResourcePaths()方法可以实现，定义如下：

```java
Set<String> getResourcePaths(String path)
```

返回类似于目录的列表，列出了Web应用程序中资源的所有路径，这些路径的最长子路径与提供的path参数匹配。

比如获取/WEB-INF下的所有资源路径

```java
Set set = servletContext.getResourcePaths("/WEB-INF")；
System.out.println(set)；
```

# 5、一文读懂Servlet

一句话来说，Servlet是运行在Web服务器的一个小型Java程序，它接受和响应来自Web客户端的请求（通常是通过HTTP协议）。

Servlet的实现可以是通过如下方式：

- 实现javax.servlet.Servlet接口；
- 继承javax.servlet.GenericServlet类；
- 继承javax.servlet.http.HttpServlet类。

我们通常的做法是会去继承HttpServlet类来完成Servlert的实现。它包含有3个生命周期方法，分别是：

```java
void init(ServletConfig config)
throws ServletException

void service(ServletRequest req, ServletResponse res)
throws ServletException, IOException
    
void destroy()
```

**init()方法：**该方法实例化Servlet对象后**只执行一次**，由servlet容器调用，以向servlet指示该servlet正在投入使用。

**service()方法：**由servlet容器调用，会被调用多次，以允许servlet响应请求。

**destroy()方法：**由servlet容器调用，以指示servlet该servlet正在退出服务。此方法仅被调用一次，仅在servlet的service方法中的所有线程都已退出或经过超时时间后，才调用此方法。



在HttpServlet抽象类的子类能够创建一个适用于网站的HTTP servlet，HttpServlet的子类必须**至少覆写一个**方法，通常是下面的方法中的一个：

- doGet()，为了servlet支持HTTP GET请求；
- doPost()，为了servlet支持HTTP POST请求；
- doPut()，为了servlet支持HTTP PUT请求；
- doDelete()，为了servlet支持HTTP DELETE请求；
- init()和destroy()，为了servlet支持管理在Servlet生命周期内保留的资源；
- getServletInfo()，为了servlet支持提供关于自身的信息。

此类中没有必要再去覆写service()方法，通过将标准HTTP请求分派到每种HTTP请求类型的处理程序方法（上面列出的doXXX方法）来处理它们。

【待续】