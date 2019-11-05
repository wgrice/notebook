## 什么是 Servlet

Servlet（Server Applet），全称 Java Servlet，未有中文译文。是用Java编写的服务器端程序。其主要功能在于交互式地浏览和修改数据，生成动态 Web 内容。狭义的Servlet是指Java语言实现的一个接口，广义的 Servlet 是指任何实现了这个 Servlet 接口的类，一般情况下，人们将 Servlet 理解为后者。

Servlet 运行于支持Java的应用服务器中。从实现上讲，Servlet 可以响应任何类型的请求，但绝大多数情况下Servlet 只用来扩展基于 HTTP 协议的 Web 服务器。

## Servlet 的工作模式

- 客户端发送请求至服务器；
- 服务器启动并调用 Servlet，Servlet 根据客户端请求生成响应内容并将其传给服务器；
- 服务器将响应返回客户端；

## Servlet API 概览

Servlet API 包含以下4个Java包：

1. javax.servlet   其中包含定义 Servlet 和 Servlet 容器之间契约的类和接口。

2. javax.servlet.http   其中包含定义 HTTP Servlet 和 Servlet 容器之间的关系。

3. javax.servlet.annotation   其中包含标注 Servlet、Filter、Listener 的标注。它还为被标注元件定义元数据。

4. javax.servlet.descriptor   其中包含提供程序化登录Web应用程序的配置信息的类型。

## Servlet 的主要类型

![](C:\wg\project\git\notebook\image\servlet\20180512170931739.png)

## Servlet 的使用方法

Servlet 技术的核心是 Servlet，==它是所有Servlet类必须直接或者间接实现的一个接口==。在编写实现 Servlet 的Servlet 类时，直接实现它。在扩展实现这个这个接口的类时，间接实现它。

## Servlet 的工作原理

Servlet 接口定义了 Servlet 与 Servlet 容器之间的契约。这个契约是：Servlet 容器将 Servlet 类载入内存，并产生 Servlet 实例和调用它具体的方法。但是要注意的是，在一个应用程序中，每种 Servlet 类型只能有一个实例。

用户请求致使 Servlet 容器调用 Servlet 的 Service() 方法，并传入一个 ServletRequest 对象和一个 ServletResponse 对象。ServletRequest 对象和 ServletResponse 对象都是由 Servlet 容器（例如 TomCat ）封装好的，并不需要程序员去实现，程序员可以直接使用这两个对象。

ServletRequest 中封装了当前的 Http 请求，因此，开发人员不必解析和操作原始的 Http 数据。ServletResponse 表示当前用户的 Http 响应，程序员只需直接操作 ServletResponse 对象就能把响应轻松的发回给用户。

对于每一个应用程序，Servlet 容器还会创建一个 ServletContext 对象。这个对象中封装了上下文（应用程序）的环境详情。每个应用程序只有一个 ServletContext。每个 Servlet 对象也都有一个封装 Servlet 配置的 ServletConfig 对象。

## Servlet 接口中定义的方法
让我们首先来看一看Servlet接口中定义了哪些方法吧。

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;
 
    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();
    
    void destroy();
} 
```

## Servlet 的生命周期
其中 `init()`、`service()`、`destroy()` 是 Servlet 生命周期的方法。代表了 Servlet 从“出生”到“工作”再到“死亡 ”的过程。Servlet 容器（例如 TomCat ）会根据下面的规则来调用这三个方法：

1. `init()` 当Servlet第一次被请求时，Servlet 容器就会开始调用这个方法来初始化一个 Servlet 对象出来，但是这个方法在后续请求中不会在被 Servlet 容器调用，就像人只能“出生”一次一样。我们可以利用`init()` 方法来执行相应的初始化工作。调用这个方法时，Servlet 容器会传入一个 ServletConfig 对象进来从而对 Servlet 对象进行初始化。

2. `service()` 每当请求 Servlet 时，Servlet 容器就会调用这个方法。就像人一样，需要不停的接受老板的指令并且“工作”。第一次请求时，Servlet 容器会先调用 `init( )` 方法初始化一个 Servlet 对象出来，然后会调用它的 `service( )` 方法进行工作，但在后续的请求中，Servlet 容器只会调用 `service()` 方法了。

3. `destory()` 当要销毁 Servlet 时，Servlet 容器就会调用这个方法，就如人一样，到时期了就得死亡。在卸载应用程序或者关闭 Servlet 容器时，就会发生这种情况，一般在这个方法中会写一些清除代码。

首先，我们来编写一个简单的Servlet来验证一下它的生命周期：

```java
public class MyFirstServlrt implements Servlet {

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("Servlet正在初始化");
    }
    
    @Override
    public ServletConfig getServletConfig() {
        return null;
    }
    
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        //专门向客服端提供响应的方法
        System.out.println("Servlet正在提供服务");
    
    }
    
    @Override
    public String getServletInfo() {
        return null;
    }
    
    @Override
    public void destroy() {
        System.out.println("Servlet正在销毁");
    }

}   
```


然后在xml中配置正确的映射关系，在浏览器中访问 Servlet，第一次访问时，控制台输出了如下信息：

![](C:\wg\project\git\notebook\image\servlet\20180512164825188.png)

然后，我们在浏览器中刷新3遍：

控制台输出的信息变成了下面这样：

![](C:\wg\project\git\notebook\image\servlet\20180512164922863.png)

接下来，我们关闭Servlet容器：

![](C:\wg\project\git\notebook\image\servlet\20180512165004507.png)

控制台输出了 Servlet 的销毁信息，这就是一个 Servlet 的完整生命周期。

## Servlet  的其它两个方法
- `getServletInfo()` 这个方法会返回 Servlet 的一段描述，可以返回一段字符串。
- `getServletConfig()` 这个方法会返回由 Servlet 容器传给 `init()` 方法的 ServletConfig 对象。

## ServletRequset 接口
Servlet 容器对于接受到的每一个 Http 请求，都会创建一个 ServletRequest 对象，并把这个对象传递给 Servlet 的 `Sevice() ` 方法。其中，ServletRequest 对象内封装了关于这个请求的许多详细信息。

让我们来看一看 ServletRequest 接口的部分内容：

```java
public interface ServletRequest {

    int getContentLength();//返回请求主体的字节数

    String getContentType();//返回主体的MIME类型

    String getParameter(String var1);//返回请求参数的值

} 
```


其中，getParameter 是在 ServletRequest 中最常用的方法，可用于获取查询字符串的值。

## ServletResponse 接口
javax.servlet.ServletResponse 接口表示一个 Servlet 响应，在调用 Servlet 的 `Service( )` 方法前，Servlet 容器会先创建一个 ServletResponse 对象，并把它作为第二个参数传给 `Service( )` 方法。ServletResponse 隐藏了向浏览器发送响应的复杂过程。

让我们也来看看ServletResponse内部定义了哪些方法：

```java
public interface ServletResponse {

    String getCharacterEncoding();

    String getContentType();
    
    ServletOutputStream getOutputStream() throws IOException;
    
    PrintWriter getWriter() throws IOException;
    
    void setCharacterEncoding(String var1);
    
    void setContentLength(int var1);
    
    void setContentType(String var1);
    
    void setBufferSize(int var1);
    
    int getBufferSize();
    
    void flushBuffer() throws IOException;
    
    void resetBuffer();
    
    boolean isCommitted();
    
    void reset();
    
    void setLocale(Locale var1);
    
    Locale getLocale();

}
```


其中的 `getWriter()` 方法，它返回了一个可以向客户端发送文本的的 Java.io.PrintWriter 对象。==默认情况下，PrintWriter 对象使用 ISO-8859-1 编码（该编码在输入中文时会发生乱码）==。

在向客户端发送响应时，大多数都是使用该对象向客户端发送HTML。

还有一个方法也可以用来向浏览器发送数据，它就是 `getOutputStream()`，从名字就可以看出这是一个二进制流对象，因此这个方法是用来发送二进制数据的。

在发送任何HTML之前，应该先调用 `setContentType()` 方法，设置响应的内容类型，并将 ==text/html== 作为一个参数传入，这是在告诉浏览器响应的内容类型为 HTML，需要以 HTML 的方法解释响应内容而不是普通的文本，或者也可以加上 ==charset=UTF-8== 改变响应的编码方式以防止发生中文乱码现象。

## ServletConfig 接口

当 Servlet 容器初始化 Servlet 时，Servlet容器会给 Servlet 的 `init( )` 方式传入一个 ServletConfig 对象。

其中几个方法如下：

```java
public interface ServletConfig {
    
    /**
     * 返回servlet实例名称：
     * 该名称可以通过服务器管理提供，在web应用程序部署描述符中指定；
     * 或者对于未命名的servlet实例，默认将是servlet的类名。
     * @return	servlet实例名称
     */
    public String getServletName();
 
    /**
     * 返回对象{@link ServletContext}：servlet上下文
     * @return	{@link ServletContext}对象，与容器交互
     * @see ServletContext
     */
    public ServletContext getServletContext();
    
    /**
     * Gets the value of the initialization parameter with the given name.
     * 获取初始参数值
     * @param name 初始参数名
     * @return 若不存在参数名，返回null
     */
    public String getInitParameter(String name);
 
    /**
     * 返回servlet的所有初始参数
     * @return Enumeration<String>
     */
    public Enumeration<String> getInitParameterNames();
 
}
```

## ServletContext对象
ServletContext 对象表示 Servlet 应用程序。每个 Web 应用程序都只有一个 ServletContext 对象。在将一个应用程序同时部署到多个容器的分布式环境中，每台 Java 虚拟机上的 Web 应用都会有一个 ServletContext 对象。

通过在 ServletConfig 中调用 `getServletContext()`方法，也可以获得 ServletContext 对象。

那么为什么要存在一个 ServletContext 对象呢？存在肯定是有它的道理，因为有了 ServletContext 对象，就可以共享从应用程序中的所有资料处访问到的信息，并且可以动态注册 Web 对象。前者将对象保存在 ServletContext 中的一个内部 Map 中。保存在 ServletContext 中的对象被称作属性。

ServletContext 中的下列方法负责处理属性：

```java
Object getAttribute(String var1);

Enumeration<String> getAttributeNames();

void setAttribute(String var1, Object var2);

void removeAttribute(String var1);
```

## GenericServlet 抽象类 
 前面我们编写 Servlet 一直是通过实现 Servlet 接口来编写的，但是使用这种方法，则必须要实现 Servlet 接口中定义的所有的方法，即使有一些方法中没有任何东西也要去实现，并且还需要自己手动的维护 ServletConfig 这个对象的引用。因此，这样去实现 Servlet 是比较麻烦的。

```java
void init(ServletConfig var1) throws ServletException;
```

幸好，GenericServlet 抽象类的出现很好的解决了这个问题。本着尽可能使代码简洁的原则，GenericServlet 实现了 Servlet 和 ServletConfig 接口，下面是 GenericServlet 抽象类的具体代码：

```java
public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
    private static final String LSTRING_FILE = "javax.servlet.LocalStrings";
    private static ResourceBundle lStrings = ResourceBundle.getBundle("javax.servlet.LocalStrings");
    private transient ServletConfig config;

    public GenericServlet() {
    }
    
    public void destroy() {
    }
    
    public String getInitParameter(String name) {
        ServletConfig sc = this.getServletConfig();
        if (sc == null) {
            throw new IllegalStateException(lStrings.getString("err.servlet_config_not_initialized"));
        } else {
            return sc.getInitParameter(name);
        }
    }
    
    public Enumeration<String> getInitParameterNames() {
        ServletConfig sc = this.getServletConfig();
        if (sc == null) {
            throw new IllegalStateException(lStrings.getString("err.servlet_config_not_initialized"));
        } else {
            return sc.getInitParameterNames();
        }
    }
    
    public ServletConfig getServletConfig() {
        return this.config;
    }
    
    public ServletContext getServletContext() {
        ServletConfig sc = this.getServletConfig();
        if (sc == null) {
            throw new IllegalStateException(lStrings.getString("err.servlet_config_not_initialized"));
        } else {
            return sc.getServletContext();
        }
    }
    
    public String getServletInfo() {
        return "";
    }
    
    public void init(ServletConfig config) throws ServletException {
        this.config = config;
        this.init();
    }
    
    public void init() throws ServletException {
    }
    
    public void log(String msg) {
        this.getServletContext().log(this.getServletName() + ": " + msg);
    }
    
    public void log(String message, Throwable t) {
        this.getServletContext().log(this.getServletName() + ": " + message, t);
    }
    
    public abstract void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
    
    public String getServletName() {
        ServletConfig sc = this.getServletConfig();
        if (sc == null) {
            throw new IllegalStateException(lStrings.getString("err.servlet_config_not_initialized"));
        } else {
            return sc.getServletName();
        }
    }

}

```

其中，GenericServlet 抽象类相比于直接实现 Servlet 接口，有以下几个好处：

1. 为 Servlet 接口中的所有方法提供了默认的实现，则程序员需要什么就直接改什么，不再需要把所有的方法都自己实现了。

2. 提供方法，包围 ServletConfig 对象中的方法。

3. 将 `init()`方法中的 ServletConfig 参数赋给了一个内部的 ServletConfig 引用从而来保存 ServletConfig 对象，不需要程序员自己去维护 ServletConfig 了。

```java
public void init(ServletConfig config) throws ServletException {
    this.config = config;
    this.init();
}
```

但是，我们发现在 GenericServlet 抽象类中还存在着另一个没有任何参数的 `Init()` 方法：

```java
public void init() throws ServletException {
}
```

设计者的初衷到底是为了什么呢？在第一个带参数的 `init()` 方法中就已经把 ServletConfig 对象传入并且通过引用保存好了，完成了 Servlet 的初始化过程，那么为什么后面还要加上一个不带任何参数的 `init()` 方法呢？这不是多此一举吗？

当然不是多此一举了，存在必然有存在它的道理。我们知道，抽象类是无法直接产生实例的，需要另一个类去继承这个抽象类，那么就会发生方法覆盖的问题，如果在类中覆盖了 GenericServlet 抽象类的 `init()` 方法，那么程序员就必须手动的去维护 ServletConfig 对象了，还得调用 `super.init(servletConfig)`方法去调用父类 GenericServlet 的初始化方法来保存 ServletConfig 对象，这样会给程序员带来很大的麻烦。

GenericServlet 提供的第二个不带参数的 `init() ` 方法，就是为了解决上述问题的。

这个不带参数的 `init()` 方法，是在 ServletConfig 对象被赋给 ServletConfig 引用后，由第一个带参数的 `init(ServletConfig servletconfig)` 方法调用的，那么这意味着，当程序员如果需要覆盖这个GenericServlet 的初始化方法，则只需要覆盖那个不带参数的 `init( )` 方法就好了，此时，servletConfig 对象仍然有 GenericServlet 保存着。

说了这么多，通过扩展 GenericServlet 抽象类，就不需要覆盖没有计划改变的方法。因此，代码将会变得更加的简洁，程序员的工作也会减少很多。

然而，虽然 GenricServlet 是对 Servlet 一个很好的加强，但是也不经常用，因为他不像 HttpServlet 那么高级。HttpServlet 才是主角，在现实的应用程序中被广泛使用。那么我们接下来就看看传说中的 HttpServlet 到底厉害在哪里吧。

## javax.servlet.http 包内容

之所以所 HttpServlet 要比 GenericServlet 强大，其实也是有道理的。HttpServlet 是由 GenericServlet 抽象类扩展而来的，HttpServlet 抽象类的声明如下所示：

```java
public abstract class HttpServlet extends GenericServlet implements Serializable 
```

HttpServlet 之所以运用广泛的另一个原因是现在大部分的应用程序都要与 HTTP 结合起来使用。这意味着我们可以利用 HTTP 的特性完成更多更强大的任务。Javax.servlet.http 包是 Servlet API 中的第二个包，其中包含了用于编写 Servlet 应用程序的类和接口。Javax.servlet.http 中的许多类型都覆盖了 Javax.servlet 中的类型。

 ![](C:\wg\project\git\notebook\image\servlet\20180513104757248.png)

## HttpServlet抽象类

HttpServlet 抽象类是继承于 GenericServlet 抽象类而来的。使用 HttpServlet 抽象类时，还需要借助分别代表 Servlet 请求和 Servlet 响应的 HttpServletRequest 和 HttpServletResponse 对象。

HttpServletRequest 接口扩展于 javax.servlet.ServletRequest 接口，HttpServletResponse 接口扩展于 javax.servlet.servletResponse 接口。

```java
public interface HttpServletRequest extends ServletRequest
public interface HttpServletResponse extends ServletResponse
```

HttpServlet 抽象类覆盖了 GenericServlet 抽象类中的 `service()` 方法，并且添加了一个自己独有的 `Service(HttpServletRequest request, HttpServletResponse)` 方法。

让我们来具体的看一看 HttpServlet 抽象类是如何实现自己的 service 方法吧：

首先来看 GenericServlet 抽象类中是如何定义 service 方法的：

```java
public abstract void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
```

我们看到是一个抽象方法，也就是 HttpServlet 要自己去实现这个 service 方法，我们在看看 HttpServlet 是怎么覆盖这个 service方法的：

```java
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
    HttpServletRequest request;
    HttpServletResponse response;
    try {
        request = (HttpServletRequest)req;
        response = (HttpServletResponse)res;
    } catch (ClassCastException var6) {
        throw new ServletException("non-HTTP request or response");
    }

	this.service(request, response);

}
```

我们发现，HttpServlet 中的 service 方法把接收到的 ServletRequsest 类型的对象转换成了 HttpServletRequest 类型的对象，把 ServletResponse 类型的对象转换成了 HttpServletResponse 类型的对象。之所以能够这样强制的转换，是因为在调用 Servlet 的 service 方法时，Servlet 容器总会传入一个 HttpServletRequest 对象和 HttpServletResponse 对象，预备使用 HTTP 。因此，转换类型当然不会出错了。

转换之后，service方法把两个转换后的对象传入了另一个service方法，那么我们再来看看这个方法是如何实现的：

```java
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    String method = req.getMethod();
    long lastModified;
    if (method.equals("GET")) {
        lastModified = this.getLastModified(req);
        if (lastModified == -1L) {
            this.doGet(req, resp);
        } else {
            long ifModifiedSince = req.getDateHeader("If-Modified-Since");
            if (ifModifiedSince < lastModified) {
                this.maybeSetLastModified(resp, lastModified);
                this.doGet(req, resp);
            } else {
                resp.setStatus(304);
            }
        }
    } else if (method.equals("HEAD")) {
        lastModified = this.getLastModified(req);
        this.maybeSetLastModified(resp, lastModified);
        this.doHead(req, resp);
    } else if (method.equals("POST")) {
        this.doPost(req, resp);
    } else if (method.equals("PUT")) {
        this.doPut(req, resp);
    } else if (method.equals("DELETE")) {
        this.doDelete(req, resp);
    } else if (method.equals("OPTIONS")) {
        this.doOptions(req, resp);
    } else if (method.equals("TRACE")) {
        this.doTrace(req, resp);
    } else {
        String errMsg = lStrings.getString("http.method_not_implemented");
        Object[] errArgs = new Object[]{method};
        errMsg = MessageFormat.format(errMsg, errArgs);
        resp.sendError(501, errMsg);
    }

}
```

我们发现，这个 service 方法的参数是 HttpServletRequest 对象和 HttpServletResponse 对象，刚好接收了上一个 service 方法传过来的两个对象。

 接下来我们再看看 service 方法是如何工作的，我们会发现在service方法中还是没有任何的服务逻辑，但是却在解析HttpServletRequest 中的方法参数，并调用以下方法之一：doGet、doPost、doHead、doPut、doTrace、doOptions 和 doDelete 。这 7 种方法中，每一种方法都表示一个 Http 方法。doGet 和 doPost 是最常用的。所以，如果我们需要实现具体的服务逻辑，不再需要覆盖 service 方法了，只需要覆盖 doGet 或者 doPost 就好了。

总之，HttpServlet 有两个特性是 GenericServlet 所不具备的：

1. 不用覆盖service方法，而是覆盖 doGet 或者 doPost 方法。在少数情况，还会覆盖其他的 5 个方法。

2. 使用的是 HttpServletRequest 和 HttpServletResponse 对象。

## HttpServletRequest接口









HttpServletRequest 表示 Http 环境中的 Servlet 请求。它扩展于 javax.servlet.ServletRequest 接口，并添加了几个方法。

```java
//返回请求上下文的请求URI部分
String getContextPath();
//返回一个cookie对象数组
Cookie[] getCookies();
//返回指定HTTP标题的值
String getHeader(String var1);
//返回请求URL中的查询字符串
String getQueryString();
//返回与这个请求相关的会话对象
HttpSession getSession();
```

## HttpServletRequest 内封装的请求

因为 Request 代表请求，所以我们可以通过该对象分别获得 HTTP 请求的请求行，请求头和请求体。

![](C:\wg\project\git\notebook\image\servlet\20180513130638615.png)

### 通过 request 获得请求行


假设查询字符串为：==username=zhangsan&password=123==

获得客户端的请求方式：`String getMethod()`

获得请求的资源：

```java
String getRequestURI()

StringBuffer getRequestURL()
// web应用的名称
String getContextPath()
// get提交url地址后的参数字符串
String getQueryString()
```

###  通过 request 获得请求头

```java
long getDateHeader(String name)

String getHeader(String name)

Enumeration getHeaderNames()

Enumeration getHeaders(String name)

int getIntHeader(String name)
```

referer 头的作用：执行该此访问的的来源，做防盗链。

### 通过request获得请求体

请求体中的内容是通过post提交的请求参数，格式是：

==username=zhangsan&password=123&hobby=football&hobby=basketball==

| key      | value    |
| -------- | -------- |
| username | zhangsan |
| password | 123      |

以上面参数为例，通过一下方法获得请求参数：

```java
String getParameter(String name)

String[] getParameterValues(String name)

Enumeration getParameterNames()

Map<String,String[]> getParameterMap()
```

   注意：**get请求方式的请求参数 上述的方法一样可以获得**。

### Request乱码问题的解决方法







在前面我们讲过，在 service 中使用的编码解码方式默认为：ISO-8859-1 编码，但此编码并不支持中文，因此会出现乱码问题，所以我们需要手动修改编码方式为 UTF-8 编码，才能解决中文乱码问题，下面是发生乱码的具体细节：

 ![](C:\wg\project\git\notebook\image\servlet\20180513195355316.png)

>  解决post提交方式的乱码：
>
> ​	request.setCharacterEncoding("UTF-8");
>
>  解决get提交的方式的乱码：
>
> ​	parameter = newString(parameter.getbytes("iso8859-1"),"utf-8");

## HttpServletResponse 接口

在 Servlet API 中，定义了一个 HttpServletResponse 接口，它继承自 ServletResponse 接口，专门用来封装 HTTP 响应消息。    由于 HTTP 请求消息分为状态行，响应消息头，响应消息体三部分，因此，在 HttpServletResponse 接口中定义了向客户端发送响应状态码，响应消息头，响应消息体的方法。

### HttpServletResponse 内封装的响应

 ![](C:\wg\project\git\notebook\image\servlet\20180513203525684.png)

### 通过 Response 设置响应

```java
// 给这个响应添加一个 cookie
void addCookie(Cookie var1);
// 给这个请求添加一个响应头
void addHeader(String var1, String var2);
// 发送一条响应码，讲浏览器跳转到指定的位置
void sendRedirect(String var1) throws IOException;
//设置响应行的状态码
void setStatus(int var1);

void addHeader(String name, String value)

void addIntHeader(String name, int value)

void addDateHeader(String name, long date)

void setHeader(String name, String value)

void setDateHeader(String name, long date)

void setIntHeader(String name, int value)
```

其中，add 表示添加，而 set 表示设置。

```java
PrintWriter getWriter()
```

获得字符流，通过字符流的 `write(String s)` 方法可以将字符串设置到 response 缓冲区中，随后 Tomcat 会将 response 缓冲区中的内容组装成 Http 响应返回给浏览器端。

```java
ServletOutputStream getOutputStream()
```

获得字节流，通过该字节流的 `write(byte[] bytes)` 可以向 response 缓冲区中写入字节，再由 Tomcat 服务器将字节内容组成Http响应返回给浏览器。

注意：**虽然 response 对象的 `getOutSream()` 和 `getWriter()` 方法都可以发送响应消息体，但是他们之间相互排斥，不可以同时使用，否则会发生异常**。

### Response 的乱码问题


原因：response 缓冲区的默认编码是 iso8859-1 ，此码表中没有中文。所以需要更改 response 的编码方式：

![](C:\wg\project\git\notebook\image\servlet\20180513205620262.png)

通过更改 response 的编码方式为 UTF-8 ，任然无法解决乱码问题，因为发送端服务端虽然改变了编码方式为 UTF-8，但是接收端浏览器端仍然使用 GB2312 编码方式解码，还是无法还原正常的中文，因此还需要告知浏览器端使用 UTF-8 编码去解码。

上面通过调用两个方式分别改变服务端对于 Response 的编码方式以及浏览器的解码方式为同样的 UTF-8 编码来解决编码方式不一样发生乱码的问题。

![](C:\wg\project\git\notebook\image\servlet\20180513210927639.png)

`response.setContentType("text/html;charset=UTF-8")` 这个方法包含了上面的两个方法的调用，因此在实际的开发中，只需要调用一个 `response.setContentType("text/html;charset=UTF-8")` 方法即可。

![](C:\wg\project\git\notebook\image\servlet\20180513204459256.png)

## Response的工作流程

![](C:\wg\project\git\notebook\image\servlet\20180513204718428.png)

## Servlet的工作流程

![](C:\wg\project\git\notebook\image\servlet\20180513204808318.png)

## 编写第一个Servlet

首先，我们来写一个简单的用户名，密码的登录界面的html文件：

```html
<form action="/form" method="get">
```


该 html 文件在最后点击提交按钮时，把表单所有数据通过 Get 方式发送到 /form 虚拟路径下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/form" method="get">
    <span>用户名</span><input type="text" name="username"><br>
    <span>密码</span><input type="password" name="password"><br>
    <input type="submit" name="submit">
</form>
</body>
</html>
```

访问一下我们刚才写的这个简单的登录界面：

![](C:\wg\project\git\notebook\image\servlet\20180514163530760.png)

![](C:\wg\project\git\notebook\image\servlet\20180514163550574.png)

接下来，我们就开始写一个 Servlet 用来接收处理表单发送过来的请求，这个 Servlet 的名称就叫做 FormServlet：

```java
public class FormServlet extends HttpServlet {
    private static final long serialVersionUID = -4186928407001085733L;
 
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 
    }
 
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
         //设置响应的编码格式为UTF-8编码，否则发生中文乱码现象
        response.setContentType("text/html;charset=UTF-8");
        //1.获得请求方式
        String method = request.getMethod();
        //2.获得请求的资源相关的内容
        String requestURI = request.getRequestURI();//获得请求URI
        StringBuffer requestURL = request.getRequestURL();
        String webName = request.getContextPath();//获得应用路径（应用名称）
        String querryString = request.getQueryString();//获得查询字符串
 
        response.getWriter().write("<h1>下面是获得的字符串</h1>");
        response.getWriter().write("<h1>method(HTTP方法):<h1>");
        response.getWriter().write("<h1>"+method+"</h1><br>");
        response.getWriter().write("<h1>requestURi(请求URI）:</h1>");
        response.getWriter().write("<h1>" + requestURI + "</h1><br>");
        response.getWriter().write("<h1>webname(应用名称):</h1>");
        response.getWriter().write("<h1>" + webName + "</h1><br>");
        response.getWriter().write("<h1>querrystring(查询字符串):</h1>");
        response.getWriter().write("<h1>" + querryString + "</h1>");
 
 
    }
}
```

该 Servlet 的作用是，接收 form 登录表单发送过来的 HTTP 请求，并解析出请求中封装的一些参数，然后在回写到 response 响应当中去，最后在浏览器端显示。

最后一步，我们在 XML 中配置好这个 Servlet 的映射关系：

```xml
<servlet>
    <servlet-name>FormServlet</servlet-name>
    <servlet-class>com.javaee.util.FormServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>FormServlet</servlet-name>
    <url-pattern>/form</url-pattern>
</servlet-mapping>
```

 接下来，启动 tomcat，在浏览器中输入登录表单的地址：

填入用户名为：root，密码为：123,最后点击提交：

![](C:\wg\project\git\notebook\image\servlet\20180514171147398.png)

提交之后，表单数据将会发送到相应的 Servlet 进行处理，此时，浏览器的地址变成如下所示：

![](C:\wg\project\git\notebook\image\servlet\20180514171352799.png)

我们会发现，在地址栏中，多了后面的 ==?username=root&password=123&提交=提交== 字符串，这其实就是我们开始填写的参数，以 Get 的方法发送过去，所以查询字符串会直接加在链接后面，如果采用的是 Post 方式则不会出现在链接中，因此，登录表单为了安全性大多采用 Post 方式提交。

我们来看看 Servlet 给我们返回了什么东西：

![](C:\wg\project\git\notebook\image\servlet\20180514171653913.png)

正如我们在 Servlet 中写的那样，Servlet 把 HTTP 请求中的部分参数给解析出来了。

因此，可以再翻到上面的 Servlet 重新去理解一遍Servlet的工作原理，可能会有更清晰的认识。

## Servlet 的局限性

我们已经看到，Servlet 如果需要给客户端返回数据，比如像下面这样的一个 HTML 文件：

 ![](C:\wg\project\git\notebook\image\servlet\20180514172209114.png)

Servlet 内部需要这样写输出语句：

```java
PrintWriter writer = response.getWriter();
writer.write("<!DOCTYPE html>\n" +
        "<html>\n" +
        "\t<head>\n" +
        "\t\t<meta charset=\"UTF-8\">\n" +
        "\t\t<title>标题标签</title>\n" +
        "\t</head>\n" +
        "\t<body>\n" +
        "\t\t<!--标题标签-->\n" +
        "\t\t<h1>公司简介</h1><br />\n" +
        "\t\t<h2>公司简介</h2><br />\n" +
        "\t\t<h3>公司简介</h3><br />\n" +
        "\t\t<h4>公司简介</h4><br />\n" +
        "\t\t\n" +
        "\t\t<!--加入一条水平线-->\n" +
        "\t\t<hr />\n" +
        "\t\t\n" +
        "\t\t<h5>公司简介</h5><br />\n" +
        "\t\t<h6>公司简介</h7><br />\n" +
        "\t\t<h100>公司简介</h100>\n" +
        "\t</body>\n" +
        "</html>\n");
————————————————
版权声明：本文为CSDN博主「刘扬俊」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_19782019/article/details/80292110
```

即一行一行的把 HTML 语句给用 Writer 输出，早期简单的网页还能应付得住，但是随着互联网的不断发展，网站的内容和功能越来越强大，一个普通的 HTML 文件可能就达到好几百行，如果在采用使用 Servlet 去一行一行的输出 HTML 代码的话，将会非常的繁琐并且浪费大量的时间，且在当时，出现了 PHP 这种可以内嵌到 HTML 文件的动态语言，使得制作动态网页变得异常的简单和轻松，因此大量的程序员转上了PHP语言的道路，JAVA 的份额急剧减小，当时 JAVA 的开发者 Sun 公司为了解决这个问题，也开发出了自己的动态网页生成技术，使得同样可以在 HTML 文件里内嵌 JAVA 代码，这就是现在的 JSP 技术，关于 JSP 技术的具体内容，我们将留到下一节进行讲解。

## ServletContextListener（Servlet全局监听器）

首先要说明的是，ServletContextListener 是一个接口，我们随便写一个类，只要这个类实现了 ServletContextListener 接口，那么这个类就实现了==监听 ServletContext==的功能。那么，这个神奇的接口是如何定义的呢？我们来看一下这个接口的内部情况：

```java
package javax.servlet;

import java.util.EventListener;

public interface ServletContextListener extends EventListener {
    
    void contextInitialized(ServletContextEvent var1);

	void contextDestroyed(ServletContextEvent var1);

}
```

我们发现，在这个接口中只声明了两个方法，分别是 `void contextInitialized(ServletContextEvent var1)` 和 `void contextDestroyed(ServletContextEvent var1) `方法，所以，我们很容易的就能猜测到，ServletContext 的生命只有两种，分别是：

1. ServletContext 初始化（应用start时） ---> Servle t容器调用 `void contextInitialized(ServletContextEvent var1)`

2. ServletContext 销毁（应用stop时） ---> Servlet 容器调用 `void contextDestroyed(ServletContextEvent var1)`

因此，我们大概能够猜到 ServletContextListener 的工作机制了，当应用启动时，ServletContext 进行初始化，然后 Servlet 容器会自动调用正在监听 ServletContext 的 ServletContextListener 的 `void contextInitialized(ServletContextEvent var1)` 方法，并向其传入一个 ServletContextEvent 对象。当应用停止时，ServletContext 被销毁，此时 Servlet 容器也会自动地调用正在监听 ServletContext 的S ervletContextListener 的 `void contextDestroyed(ServletContextEvent var1)` 方法。

为了验证我们的猜测，我们来随便写一个类，并且实现 ServletContextListener 接口，即实现监听 ServletContext 的功能：

```java
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("ServletContextListener.contextInitialized方法被调用");
    }
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("ServletContextListener.contextDestroyed方法被调用");
    }
}
```

然后，在 web.xml 中注册我们自己写的这个 MyListener:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

<listener>
    <listener-class>MyListener</listener-class>
</listener>

</web-app>
```

接下来，让我们启动一下Tomcat，看一看会发生什么吧！控制台打印信息如下：

![](C:\wg\project\git\notebook\image\servlet\20181222161836740.png)


我们发现，当应用启动时，`ServletContextListener.contextInitialized()` 方法被调用了。这其实是 Servlet 容器偷偷干的事情。那么，当我们停止 Tomcat 时，按照猜想，Servlet 容器应该也会偷偷调用 `void contextDestroyed(ServletContextEvent var1)` 方法，来通知 ServletContextListener 监听器：ServletContext 已经被销毁了。那么，事实是不是和我们猜想的一模一样呢？让我们来停止 Tomcat 的运行，看一看控制台的情况吧：

![](C:\wg\project\git\notebook\image\servlet\20181222162311326.png)

我们发现，`void contextDestroyed(ServletContextEvent var1)` 方法确实被 Servlet 容器调用了。因此，我们的猜想得到了证实。

## ServletContextListener在Spring中的应用

Spring 容器是如何借用 ServletContextListener 这个接口来实例化的。

首先让我们再来回顾一下 ServletContext 的概念，ServletContext 翻译成中文叫做 ==Servlet上下文== 或者 ==Servlet全局==，但是这个翻译我认为翻译的实在是有点牵强，也导致了许多的开发者不明白这个变量到底具体代表了什么。其实 ServletContext 就是一个“域对象”，它存在于整个应用中，并在在整个应用中有且仅有 1 份，它表示了当前整个应用的“状态”，你也可以理解为某个时刻的 ServletContext 代表了这个应用在某个时刻的“一张快照”，这张“快照”里面包含了有关应用的许多信息，应用的所有组件都可以从 ServletContext 获取当前应用的状态信息。ServletContex t随着程序的启动而创建，随着程序的停止而销毁。通俗点说，我们可以往这个 ServletContext 域对象中“存东西”，然后也可以在别的地方中“取出来”。

我们知道，Spring容器可以通过：

```java
ApplicationContext ctx=new ClassPathXmlApplicationContext("配置文件的路径"）;
```

显示地实例化一个 Spring IOC 容器。也可以像下面一样，在 web.xml 中注册 Spring IOC 容器：

```java
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<context-param>
	<param-name>contextConfigLocation</param-name>
    <param-value>
        classpath:applicationContext.xml
    </param-value>
</context-param>
```

其中的监听器类 org.springframework.web.context.ContextLoaderListener 实现了ServletContextListener 接口，能够监听 ServletContext 的生命周期中的“初始化”和“销毁”。注意，这个 org.springframework.web.context.ContextLoaderListener 监听器类人家 Spring 团队写的，我们只要拿来用就行了。当然，别忘记导入相关的 Jar 包。（spring-web-4.2.4.RELEASE.jar）

那么，Spring 团队给我们提供的这个监听器类是如何实现：当 ServletContext 初始化后，Spring IOC 容器也能跟着初始化的呢？怀着好奇心，让我们再来看一看 org.springframework.web.context.ContextLoaderListener 的内部实现情况吧。

```java
package org.springframework.web.context;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    public ContextLoaderListener() {
    }
    public ContextLoaderListener(WebApplicationContext context) {
        super(context);
    }
----------------------------重点关注下面这里哦！----------------------------
    public void contextInitialized(ServletContextEvent event) {
        this.initWebApplicationContext(event.getServletContext());
    }
----------------------------重点关注上面这里哦！----------------------------

    public void contextDestroyed(ServletContextEvent event) {
        this.closeWebApplicationContext(event.getServletContext());
        ContextCleanupListener.cleanupAttributes(event.getServletContext());
    }

}
```

我们发现，org.springframework.web.context.ContextLoaderListener 这个类实现了 ServletContextListener 接口中的两个方法，其中，当 ServletContext 初始化后，`public void contextInitialized(ServletContextEvent event)` 方法被调用，接下来执行 `initWebApplicationContext(event.getServletContext())` 方法，但是我们发现这个方法并没有在这个类中声明，因此，我们再看一下其父类中是如何声明的：

```java

public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
 
    if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
        throw new IllegalStateException("Cannot initialize context because there is already a root application context present - check whether you have multiple ContextLoader* definitions in your web.xml!");
    } else {
        Log logger = LogFactory.getLog(ContextLoader.class); 
        servletContext.log("Initializing Spring root WebApplicationContext");
        if (logger.isInfoEnabled()) { 
            logger.info("Root WebApplicationContext: initialization started");
        }
 
        long startTime = System.currentTimeMillis();

        try { 
            if (this.context == null) { 
                this.context = this.createWebApplicationContext(servletContext); 
            }

            if (this.context instanceof ConfigurableWebApplicationContext) {
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)this.context;
                if (!cwac.isActive()) {
                    if (cwac.getParent() == null) 
                        ApplicationContext parent = this.loadParentContext(servletContext); 
                        cwac.setParent(parent); 
                    }  
                    this.configureAndRefreshWebApplicationContext(cwac, servletContext); 
                }
            }
 
            servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
            ClassLoader ccl = Thread.currentThread().getContextClassLoader(); 
            if (ccl == ContextLoader.class.getClassLoader()) { 
                currentContext = this.context; 
            } else if (ccl != null) {
                currentContextPerThread.put(ccl, this.context);
            }
 
            if (logger.isDebugEnabled()) {
                logger.debug("Published root WebApplicationContext as ServletContext attribute with name [" + WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE + "]"); 
            }
 
            if (logger.isInfoEnabled()) {
                long elapsedTime = System.currentTimeMillis() - startTime;
                logger.info("Root WebApplicationContext: initialization completed in " + elapsedTime + " ms");
            }
            return this.context; 
        } catch (RuntimeException var8) { 
            logger.error("Context initialization failed", var8); 
            servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, var8); 
            throw var8;
        } catch (Error var9) { 
            logger.error("Context initialization failed", var9); 
            servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, var9);
            throw var9; 
        }
    }
}
```




分析到这一步，我们发现 Spring 容器在这个方法中被实例化了。接下来，就让我们整理一下整体的思路：

当 Servlet 容器启动时，ServletContext 对象被初始化，然后 Servlet 容器调用 web.xml 中注册的监听器的`public void contextInitialized(ServletContextEvent event)` 方法，而在监听器中，调用了`this.initWebApplicationContext(event.getServletContext())` 方法，在这个方法中实例化了 Spring IOC 容器。即 ApplicationContext 对象。

因此，当 ServletContext 创建时我们可以创建 applicationContext 对象，当 ServletContext 销毁时，我们可以销毁 applicationContext 对象。这样 applicationContext 就和 ServletContext “共生死”了。