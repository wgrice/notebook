# Tomcat 架构

## Tomcat 顶层架构

Tomcat的顶层结构图如下。

![](C:\wg\project\git\notebook\image\Tomcat顶层架构.png)

Tomcat 中最顶层的容器是 Server，代表着整个服务器，从上图中可以看出，一个 Server 可以包含至少一个 Service，用于具体提供服务。

Tomcat 要实现 2 个核心功能：

- 处理 Socket 连接，负责网络字节流与 Request 和 Response 对象的转化。
- 加载和管理 Servlet，以及具体处理 Request 请求。

因此 Tomcat 设计了两个核心组件连接器（Connector）和容器（Container）来分别做这两件事情。连接器负责对外交流，容器负责内部处理。他们的作用如下：

1. Connector 用于处理连接相关的事情，并提供 Socket 与 Request 和 Response 相关的转化; 
2. Container 用于封装和管理 Servlet，以及具体处理 Request 请求；

一个 Tomcat 中只有一个 Server，一个 Server 可以包含多个 Service，一个 Service 只有一个 Container，但是可以有多个 Connectors，这是因为一个服务可以有多个连接，如同时提供 Http 和 Https 链接，也可以提供向相同协议不同端口的连接,示意图如下：

![](C:\wg\project\git\notebook\image\Tomcat连接示意图.png)

多个 Connector 和一个 Container 就形成了一个 Service，有了 Service 就可以对外提供服务了，但是 Service 还要一个生存的环境，必须要有人能够给她生命、掌握其生死大权，那就非 Server 莫属了！所以整个 Tomcat 的生命周期由 Server 控制。

### 连接器 Connector

连接器需要完成 3 个高内聚的功能：

- 网络通信。
- 应用层协议解析。
- Tomcat Request/Response 与 ServletRequest/ServletResponse 的转化。

因此 Tomcat 的设计者设计了 3 个组件来实现这 3 个功能，分别是 EndPoint、Processor 和 Adapter。

Endpoint 和 Processor 放在一起抽象成了 ProtocolHandler 组件，连接器用 ProtocolHandler 来处理网络连接和应用层协议。

![](C:\wg\project\git\notebook\image\Tomcat连接器-1574095248864.png)

EndPoint 是一个接口，它的抽象实现类 AbstractEndpoint 里面定义了两个内部类：Acceptor 和 SocketProcessor。其中 Acceptor 用于监听 Socket 连接请求。SocketProcessor 用于处理接收到的 Socket 请求。

EndPoint 接收到 Socket 连接后，生成一个 SocketProcessor 任务提交到线程池去处理，SocketProcessor 的 Run 方法会调用 Processor 组件去解析应用层协议，Processor 通过解析生成 Request 对象后，会调用 Adapter 的 Service 方法。

![](C:\wg\project\git\notebook\image\Tomcat连接示意图2.png)

### 容器 Container

Tomcat 设计了 4 种容器，分别是 Engine、Host、Context 和 Wrapper。这 4 种容器不是平行关系，而是父子关系。

![](C:\wg\project\git\notebook\image\Tomcat容器示意图1.png)

4个子容器的作用分别是：

1. Engine：引擎，用来管理多个站点，一个 Service 最多只能有一个 Engine； 
2. Host：代表一个站点，也可以叫虚拟主机，通过配置 Host 就可以添加站点；
3. Context：代表一个 Web 应用程序，对应着平时开发的一套程序，或者一个 WEB-INF 目录以及下面的 web.xml 文件；
4. Wrapper：每一 Wrapper 封装着一个 Servlet；

一个 Service 最多只能有一个 Engine，一个 Engine 管理多个 Host，一个 Host 可以部署多个 Context（Web应用），一个 Context 可能会有多个 Wrapper（Servlet）。

![](C:\wg\project\git\notebook\image\Tomcat容器示意图2.png)

每一个容器都有一个 Pipeline 对象。Container 处理请求是使用 Pipeline-Value 管道来处理的！

Pipeline-Value 是责任链模式，责任链模式是指在一个请求处理的过程中有很多处理者依次对请求进行处理，每个处理者负责做自己相应的处理，处理完之后将处理后的请求返回，再让下一个处理着继续处理。

但是，Pipeline-Value 使用的责任链模式和普通的责任链模式有些不同。区别主要有以下两点：

1. 每个 Pipeline 都有特定的 Value，而且是在管道的最后一个执行，这个 Value 叫做 BaseValue，BaseValue 是不可删除的；
2. 在上层容器的管道的 BaseValue 中会调用下层容器的管道。

我们知道 Container 包含四个子容器，而这四个子容器对应的 BaseValue 分别在：StandardEngineValue、StandardHostValue、StandardContextValue、StandardWrapperValue，对应图中最后一个 basic。

![](C:\wg\project\git\notebook\image\Tomcat容器示意图3.png)

1. Connector 在接收到请求后会首先调用最顶层容器的 Pipeline 来处理，这里的最顶层容器的 Pipeline 就是EnginePipeline（Engine的管道）；

2. 在 Engine 的管道中依次会执行 EngineValue1、EngineValue2 等等，最后会执行 StandardEngineValue，在StandardEngineValue 中会调用 Host 管道，然后再依次执行 Host 的 HostValue1、HostValue2 等，最后在执行 StandardHostValue，然后再依次调用 Contex 的管道和 Wrapper 的管道，最后执行到 StandardWrapperValue；

3. 当执行到 StandardWrapperValue 的时候，会在 StandardWrapperValue 中创建 FilterChain，并调用其 doFilter 方法来处理请求，这个 FilterChain 包含着我们配置的与请求相匹配的 Filter 和 Servlet，其 doFilter 方法会依次调用所有的 Filter 的 doFilter方法和 Servlet 的 service 方法，这样请求就得到了处理；

4. 当所有的 Pipeline-Value 都执行完之后，并且处理完了具体的请求，这个时候就可以将返回的结果交给 Connector 了，Connector 在通过 Socket 的方式将结果返回给客户端。

## Tomcat 启动顺序

![](C:\wg\project\git\notebook\image\Tomcat启动顺序.png)