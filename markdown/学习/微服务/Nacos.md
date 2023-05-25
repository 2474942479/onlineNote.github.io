# 超硬核，Nacos实现原理详细讲解

https://mp.weixin.qq.com/s/dxi9kbsTnirpx7wMh7lG4Q

## **Nacos架构**



![图片](../../markdown/图片/640-20210916180312135)

- Provider APP：服务提供者
- Consumer APP：服务消费者
- Name Server：通过VIP（Virtual IP）或DNS的方式实现Nacos高可用集群的服务路由
- Nacos Server：Nacos服务提供者，里面包含的Open API是功能访问入口，Conig Service、Naming Service 是Nacos提供的配置服务、命名服务模块。Consitency Protocol是一致性协议，用来实现Nacos集群节点的数据同步，这里使用的是Raft算法（Etcd、Redis哨兵选举）
- Nacos Console：控制台

## **注册中心的原理**

- 服务实例在启动时注册到服务注册表，并在关闭时注销
- 服务消费者查询服务注册表，获得可用实例
- 服务注册中心需要调用服务实例的健康检查API来验证它是否能够处理请求

![图片](../../markdown/图片/640-20210916180312270)

## **SpringCloud完成注册的时机**

在Spring-Cloud-Common包中有一个类`org.springframework.cloud. client.serviceregistry .ServiceRegistry` ,它是Spring Cloud提供的服务注册的标准。集成到Spring Cloud中实现服务注册的组件,都会实现该接口。

![图片](../../markdown/图片/640-20210916180312169)

该接口有一个实现类是NacoServiceRegistry。

**SpringCloud集成Nacos的实现过程：**

在`spring-clou-commons`包的`META-INF/spring.factories`中包含自动装配的配置信息如下：

![图片](../../markdown/图片/640-20210916180312334)

其中`AutoServiceRegistrationAutoConfiguration`就是服务注册相关的配置类：

![图片](../../markdown/图片/640-20210916180312260)

在`AutoServiceRegistrationAutoConfiguration`配置类中,可以看到注入了一个`AutoServiceRegistration`实例,该类的关系图如下所示。

![图片](../../markdown/图片/640-20210916180312769)

可以看出, `AbstractAutoServiceRegistration`抽象类实现了该接口,并且最重要的是`NacosAutoServiceRegistration`继承了`AbstractAutoServiceRegistration`。

看到EventListener我们就应该知道，Nacos是通过Spring的事件机制继承到SpringCloud中去的。

`AbstractAutoServiceRegistration`实现了onApplicationEvent抽象方法,并且监听`WebServerInitializedEvent`事件(当Webserver初始化完成之后) , 调用`this.bind ( event )`方法。

![图片](../../markdown/图片/640-20210916180312216)

最终会调用`NacosServiceREgistry.register()`方法进行服务注册。

![图片](../../markdown/图片/640-20210916180312275)

![图片](../../markdown/图片/640-20210916180312285)

## **NacosServiceRegistry的实现**

在`NacosServiceRegistry.registry`方法中,调用了Nacos Client SDK中的`namingService.registerInstance`完成服务的注册。

![图片](../../markdown/图片/640-20210916180312361)

跟踪NacosNamingService的`registerInstance()`方法：

![图片](../../markdown/图片/640-20210916180312443)

- 通过`beatReactor.addBeatInfo()`创建心跳信息实现健康检测, Nacos Server必须要确保注册的服务实例是健康的,而心跳检测就是服务健康检测的手段。
- `serverProxy.registerService()`实现服务注册

**心跳机制：**

![图片](../../markdown/图片/640-20210916180312348)

从上述代码看,所谓心跳机制就是客户端通过schedule定时向服务端发送一个数据包 ,然后启动-个线程不断检测服务端的回应,如果在设定时间内没有收到服务端的回应,则认为服务器出现了故障。Nacos服务端会根据客户端的心跳包不断更新服务的状态。另外推荐：Java进阶视频资源

**注册原理：**

Nacos提供了SDK和Open API两种形式来实现服务注册。

Open API：

![图片](../../markdown/图片/640-20210916180312455)

SDK：

![图片](../../markdown/图片/640-20210916180312314)

这两种形式本质都一样，底层都是基于HTTP协议完成请求的。所以注册服务就是发送一个HTTP请求：

![图片](../../markdown/图片/640-20210916180312437)

对于nacos服务端，对外提供的服务接口请求地址为`nacos/v1/ns/instance`，实现代码咋`nacos-naming`模块下的InstanceController类中：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

- 从请求参数汇总获得serviceName（服务名）和namespaceId（命名空间Id）
- 调用registerInstance注册实例

![图片](../../markdown/图片/640-20210916180312426)

- 创建一个控服务（在Nacos控制台“服务列表”中展示的服务信息），实际上是初始化一个serviceMap，它是一个ConcurrentHashMap集合
- getService，从serviceMap中根据namespaceId和serviceName得到一个服务对象
- 调用addInstance添加服务实例

![图片](../../markdown/图片/640-20210916180312579)

![图片](../../markdown/图片/640-20210916180312407)

- 根据namespaceId、serviceName从缓存中获取Service实例
- 如果Service实例为空，则创建并保存到缓存中

![图片](../../markdown/图片/640-20210916180312510)

- 通过`putService()`方法将服务缓存到内存
- `service.init()`建立心跳机制
- `consistencyService.listen`实现数据一致性监听

`service.init ( )`方法的如下图所示，它主要通过定时任务不断检测当前服务下所有实例最后发送心跳包的时间。如果超时,则设置healthy为false表示服务不健康,并且发送服务变更事件。

在这里请大家思考一一个问题,服务实例的最后心跳包更新时间是谁来触发的?实际上前面有讲到, Nacos客户端注册服务的同时也建立了心跳机制。另外推荐：Java进阶视频资源

![图片](../../markdown/图片/640-20210916180312464)

putService方法，它的功能是将Service保存到serviceMap中：

![图片](../../markdown/图片/640-20210916180312537)

继续调用addInstance方法把当前注册的服务实例保存到Service中：

![图片](../../markdown/图片/640-20210916180312482)

总结：

- Nacos客户端通过Open API的形式发送服务注册请求

- Nacos服务端收到请求后，做以下三件事：

- - 构建一个Service对象保存到ConcurrentHashMap集合中
  - 使用定时任务对当前服务下的所有实例建立心跳检测机制
  - 基于数据一致性协议服务数据进行同步

## **服务提供者地址查询**

Open API：

![图片](../../markdown/图片/640-20210916180312466)

SDK：

![图片](../../markdown/图片/640-20210916180312557)

InstanceController中的list方法：

![图片](../../markdown/图片/640-20210916180312653)

- 解析请求参数
- 通过doSrvIPXT返回服务列表数据

![图片](../../markdown/图片/640-20210916180312711)

![图片](../../markdown/图片/640-20210916180312668)

- 根据namespaceId、serviceName获得Service实例
- 从Service实例中基于srvIPs得到所有服务提供者实例
- 遍历组装JSON字符串并返回

## **Nacos服务地址动态感知原理**

可以通过subscribe方法来实现监听，其中serviceName表示服务名、EventListener表示监听到的事件：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueTFNOceCjKWXWuBckIsVSAjiclgrKlwMkia0nlib38MTemSjrzZeh3LoeGLcaEwOsibDlwb0hhZvyCeQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

具体调用方式如下：

![图片](../../markdown/图片/640-20210916180312731)

或者调用selectInstance方法，如果将subscribe属性设置为true，会自动注册监听：

![图片](../../markdown/图片/640-20210916180312605)

![图片](../../markdown/图片/640-20210916180312658)

Nacos客户端中有一个HostReactor类，它的功能是实现服务的动态更新，基本原理是：

- 客户端发起时间订阅后，在HostReactor中有一个UpdateTask线程，每10s发送一次Pull请求，获得服务端最新的地址列表
- 对于服务端，它和服务提供者的实例之间维持了心跳检测，一旦服务提供者出现异常，则会发送一个Push消息给Nacos客户端，也就是服务端消费者
- 服务消费者收到请求之后，使用HostReactor中提供的processServiceJSON解析消息，并更新本地服务地址列表