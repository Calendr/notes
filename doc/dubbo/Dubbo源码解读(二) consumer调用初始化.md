#spring bean 的生成
在spring中Bean是核心领域模型,所以外部扩展都是将自身扩展为一个Bean的方式来适配spring,在dubbo中所有的ConsumerBean都是一个ReferenceBean.
#####ReferenceBean
![](https://ws1.sinaimg.cn/large/005uZLbtly1flqofq1ketj30wm0gkt9w.jpg)
ReferenceBean 除了实现spring bean的基本接口实现以外,还继承了ReferenceConfig这一系列dubbo配置类.在获取相关中bean, ReferenceBean起到了一个适配器的作用,将原本的getObject()转为
ReferenceConfig的get()

#####ReferenceConfig
get()
```
    public synchronized T get() {
        if (destroyed) {
            throw new IllegalStateException("Already destroyed!");
        }
        if (ref == null) {
            init();
        }
        return ref;
    }
```
在这个方法中,能看到get()方法还会执行一个init()方法，主要用于创建代理的前置基础配置准备,由于方法过长,用伪码表示

```
init(){
    //获取具体的接口类型
    setInterfaceClass(interfaceName)
    //dubbo 配置是从大到小,下层配置会覆盖上层配置,例如application < module < consumer < methods.
    //配置基础信息
    config()
    //创建提供给spring bean 使用的代理
    createProxy()
} 
```
创建代理的方法如下
```
createProxy{
     //指向本地,或者使用直连方式,那么可以通过直接引用获取相应的
       invoker,反之使用注册中心获取url
     if(!(isJvmRefer || hasUrl())){
        //拼接一个注册的url,包含consumer信息
        url = AbstractInterfaceConfig.loadRegistries()
     } 
     //向注册中心注册信息
     invoker = protocol.refer(interfaceClass, url)
     //创建代理,dubbo默认使用JavassistProxyFactory,一个字节码类型的代理
     getProxy(invoker, interfaceClass)
}
```

protocol.refer 分为两步,第一步.将consumer信息注册到注册中心,订阅相应的提供者列表(Directory).第二步,,将相应的调用使用ProtocolFilterWrapper.buildInvokerChain进行装饰,以此来添加各种过滤器,占用端口

RegistryProtocol.refer  

```
 public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
       
       //获取相应的注册中心
        Registry registry = registryFactory.getRegistry(url);
        
        //注册相应的Consumer，订阅跟consumer相关的Provider Directory,并把相应的Directory 放入到 cluster
        return doRefer(cluster, registry, type, url);
    }
```