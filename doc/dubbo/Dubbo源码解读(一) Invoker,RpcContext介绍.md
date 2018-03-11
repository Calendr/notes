

##Invoker
dubbo的核心领域模型,是各个功能调用之间的载体,除了底层的channel和最上层的proxy以外,都是用Invoker进行信息的传输.
#####提供方法如下:

![](https://ws1.sinaimg.cn/large/005uZLbtly1flpy8wk1hvj30c906sjrt.jpg)

#####Invoker作为领域模型,跟其相关联的组件如下

![](https://ws1.sinaimg.cn/large/005uZLbtly1flpzmxfaiwj30xs0gc75u.jpg)

1. Result: Invoker返回值,就是调用Provider之后,获得的最终结果  
* ProxyFactory: 将dubbo内部的调用封装成一个spring bean 的代理 类,最终在spring中通过 InvokerInvocationHandler 来调用相应的bean  
* Directory: 服务提供者列表,缓存了一份注册中心的提供者列表  
* LoadBalance: 负载均衡,根据相应算法,获取唯一一个Provider  
* Filter: 装饰者,可以给Invoker添加其他额外功能,例如监控统计,白名单等功能  
* Invocation: 调用者,包含一个Invoker，一次具体的调用,类似会话 
* Protocol: 协议层，用于在Channel上面,对信息进行转换和反转  
* Router: 路由规则,筛选出符合其路由规则的Invoker  
* Exporter: Provider向注册中心提供服务时使用  
* Cluster: 会筛选出符合调用信息的Invoker列表    
由以上的功能可见,dubbo中的Invoker,就是Spring中的Bean一样,是各个操作的载体,在后续的dubbo consumer 和 provider中还会详细介绍相应的方法调用

##RpcContext
RpcContext作为上下文,如果一个服务A调用了B,B再调用了C,那么RpcContext中只会记载相应的B调用了C,并且上下文会记录其他额外的信息,例如提供给监控中心的统计,超时调用的警告,全链路压测等功能的信息记录,结合Filter使用,不会有入侵性代码