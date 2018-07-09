# RESTful API

现在流行前后端分离，而前后端交互可以应用RESTful接口。

 REST 指的是一组架构[约束条件](https://baike.baidu.com/item/%E7%BA%A6%E6%9D%9F%E6%9D%A1%E4%BB%B6)和原则。满足这些约束条件和原则的应用程序或设计就是 RESTful。

REST被翻译为表述状态转移。表述是指资源的表述，资源本身是不被操作的；REST原则是无状态通信，此处的无状态指服务端不保存客户端状态。而实际的状态分为应用状态和资源状态，服务端通过为客户端提供资源表述及超媒体，以达到应用状态的转移，而“会话”状态则被客户端作为应用状态来跟踪。

CURD的Web数据库架构过于简单，超媒体是应用状态引擎， 要通过链接引导客户端完成资源访问。具有链接的特性被称为连通性。RESTful风格的API，要注重超媒体驱动，可参考Spring Data Rest。

资源须有URI， URI的设计应该遵循可寻址性原则，具有自描述性。 通常来讲，URI不应该使用动作来描述，最好只用来描述资源。在设计URI时，应加入版本号的概念，用于区分不同的表述形式。

示例项目：[https://github.com/spring-guides/gs-rest-hateoas](https://github.com/spring-guides/gs-rest-hateoas) 这是官方提供的超媒体驱动的RESTful Web服务。

参考资料：[http://www.runoob.com/w3cnote/restful-architecture.html](http://www.runoob.com/w3cnote/restful-architecture.html)

