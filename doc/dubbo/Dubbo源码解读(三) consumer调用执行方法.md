在前面Consumer的spring bean中说过,经过代理之后实际调用的是一个代理,当调用一个方法时,首先调用 InvokerInvocationHandler(防止出现调用代理时调用了原生的对象方法)其次调用的是MockClusterInvoker检测调用服务是否需要进行mock数据,接着调用AbstractClusterInvoker

#####AbstractClusterInvoker
```
public Result invoke(final Invocation invocation) throws RpcException {
       
        List<Invoker<T>> invokers = list(invocation);
        LoadBalance loadbalance = getLoadBalance();
        return doInvoke(invocation, invokers, loadbalance);
    }
```
调用方法时,首先获取提供相应服务的服务列表,以及相应的负载均衡算法,在list方法中,会对相应的Directory中进行挑选出符合提供相应服务的列表,再根据相应的Router提供符合路由规则的服务列表.

#####FailoverClusterInvoker
```
public Result invoke(final Invocation invocation) throws RpcException {
       
        for (int i = 0; i < len; i ++){
             selectInvoker();
             try{
                invoker.invoker
             }catch(Exception){
             }   
        }
    }
```
在这个类中,会根据相应的负载均衡算法得到唯一的服务提供方,并且会提供重试机制.  
dubbo默认的ConsumerFilter顺序如下ConsumerContextFilter(设置rpc Context上下文) --> FutureFilter(配置调用是否为同步,回调方法) --> MonitorFilter （监控用,统计数据）

#####AbstractInvoker
在前面的RpcContext中说过,RpcContext只保留一次调用的会话信息,因此在AbstractInvoker.Invoker中会将RpcContext设置的额外信息,保存到Invocation中.并将挑选相应的具体实现类来执行doInvoker,现在以DubboInvoker来说明具体实现

#####DubboInvoker

DubboInvoker.doInvoker 主要用于进行何种方式的通信,DubboInvoker使用netty将相应的Invocation序列化.传输到Provider

###Provider
会有一个线程进行死循环,主要用于接收到的管道信息,在接到信息之后,通过反序列化,然后经过相应的Protocol,将信息改装成相信的调用,最终跟经过一系列filter的调用,获得最终的返回结果.
以下是dubbo缺省方式下,调用个Provider的调用链  
InvokeInvocationHandle --> MockClusterInvoker --> AbstractClusterInvoker(获取相应的服务提供者列表,获取负载平衡算法) --> FailoverClusterInvoker(失败重试,只支持抛出 RpcContextException;使用相应的负载平衡算法挑选出相应的Invoker,进行调用) --> InvokeWrapper --> ListenerInvokerWrapper --> ConsumerContextFilter(设置rpc Context上下文) --> FutureFilter(配置调用是否为同步,回调方法) --> MonitorFilter （监控用,统计数据） --> AbstractInvoker(invoker) --> DubboInvoker (doInvoker) -->
ReferenceCountExchangeClient(request) --> HeaderExchangeClient(request) --> AbstractPeer(send) --> NettyChannel（send,nio）-->  
服务方 :ChannelEventRunnable(监听到 RECEIVEDs 事件) --> DecodeHandler(解码处理) --> HeaderExchangeHandler(处理事件) --> DubboProtocol（将相应的事件信息转换为调用类）--> EchoFilter(调试用) --> ClassLoaderFilter(将当前线程以后的调用设置为invoker接口的类加载器,调用完,设置回来) --> GenericFilter -->
ContextFilter (清空 消费方带来的额外信息) --> TraceFilter --> TimeOutFilter --> MonitorFilter (监控用) --> ExceptionFilter -->AbstractProxyInvoker -->JavassistProxyFactory


