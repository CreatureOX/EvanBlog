# Spring Cloud整体架构
## 介绍：
基于Spring Boot实现的云应用开发工具，用于管理微服务，关注全局。
包括Spring Cloud Netflix、Config配置中心，Bus事件、消息总线负责传递消息等。

## 优点：

约定优于配置
开箱即用、快速启动
适用于各种环境
轻量级的组件
组件支持丰富，功能齐全
## 其中Netflix可细分为：

### Eureka服务中心
用于定位注册发现服务，基于Rest实现，实现云端负载均衡和中间层服务器的故障转移
              Eureka Client：负责将这个服务的信息注册到 Eureka Server 中；反之亦可。
              Eureka Server：注册中心，里面有一个注册表，保存了各个服务所在的机器和端口号。
![title](https://i.loli.net/2019/06/03/5cf4be939b53e92423.png)
主要包括三种角色：
        Register Service: 服务注册中心，它是一个Eureka Server，提供服务注册和发现功能。
        Provider Service：服务提供者，Eureka client ,提供服务。
        Consumer Service ：服务消费者，Eureka client，消费服务。

### Feign 
动态代理机制。Feign Client在底层根据注解和指定的服务连接等。
              @FeignClient针对接口创建一个动态代理,根据@RequestMapping动态构造请求服务请求URL地址，后续操作。

### Ribbon负载均衡
每次请求时选择一台机器，均匀的把请求分发到各个机器上。依次轮询。
              从Eureka Client获取服务注册表，Round Robin轮询，发起请求。
![title](https://i.loli.net/2019/06/03/5cf4bed97efde45510.jpg)
### Hystrix熔断器
负责容错管理，隔离不同服务，避免服务雪崩
              创建微服务专用的线程池，熔断挂掉的微服务直接返回不走请求，服务降级：根据记录手动操作恢复故障数据库。

Hystrix仪表盘主要用于监控Hystrix的实时运行状态，通过它观察Hystrix的各项指标信息，从而快速发现系统中存在的问题并解决。 
![title](https://i.loli.net/2019/06/03/5cf4befced61593419.png)              
### Zuul边缘服务工具
微服务网关，负责网络路由。提供动态路由，监控，弹性，安全等的边缘服务。
              请求走网关，根据请求特征转发给后端各个服务而不用了解服务名，在服务众多时保证高效。

### Turbine
是聚合服务器发送事件流数据的一个工具，用来监控集群下hystrix的metrics情况。
    聚合系统的所有集群的健康状态，就是把多个/hystrix.stream，全部聚合在/turbine.stream中，通过Hystrix Dashboard中可视化查看。查看单个实例的健康情况，可以直接通过Hystrix提供的hystrix.stream就可以实现（把对应的   URL地址放入Hystrix Dashboard中查看状态），对于一个系统的所有集群的健康状态，了解系统健康状态的最宏观有用的方式可使用turbine.

  Spring Cloud的整体组件架构如下图所示：
![title](https://i.loli.net/2019/06/03/5cf4bf19def0834616.png)
## 各组件配置使用运行流程：
1. 请求统一通过API网关（Zuul）来访问内部服务.
2. 网关接收到请求后，从注册中心（Eureka）获取可用服务
3. 由Ribbon进行均衡负载后，分发到后端具体实例
4. 微服务间通过Feign进行通信处理业务
5. Hystrix负责处理服务超时熔断
6. Turbine监控服务间的调用和熔断相关指标
