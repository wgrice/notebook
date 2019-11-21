# Servlet 容器

Tomcat 或者 Jetty 就是一个“HTTP 服务器 + Servlet 容器”，我们也叫它们 Web 容器。

![](C:\wg\project\git\notebook\image\web应用结构.png)

Spring 框架就是对 Servlet 的封装，Spring 应用本身就是一个 Servlet，而 Servlet 容器是管理和运行 Servlet 的。

![](C:\wg\project\git\notebook\image\Serlvlet容器.png)

Servlet 接口和 Servlet 容器这一整套规范叫作 Servlet 规范。Tomcat 和 Jetty 都按照 Servlet 规范的要求实现了 Servlet 容器。

## Servlet 容器工作流程

当客户请求某个资源时，HTTP 服务器会用一个 ServletRequest 对象把客户的请求信息封装起来，然后调用 Servlet 容器的 service 方法，Servlet 容器拿到请求后，根据请求的 URL 和 Servlet 的映射关系，找到相应的 Servlet，如果 Servlet 还没有被加载，就用反射机制创建这个 Servlet，并调用 Servlet 的 init 方法来完成初始化，接着调用 Servlet 的 service 方法来处理请求，把 ServletResponse 对象返回给 HTTP 服务器，HTTP 服务器会把响应发送给客户端。

Servlet 规范提供了两种扩展机制：Filter和Listener。

- Filter 是干预过程的，它是过程的一部分，是基于过程行为的。
- Listener 是基于状态的，任何行为改变同一个状态，触发的事件是一致。

