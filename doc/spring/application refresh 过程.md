# spring 启动过程
## prepareRefresh
###1.初始化标记位
###2.初始化占位符标记
###3.验证必要性数据不为空

## obtainFreshBeanFactory
###1.刷新BeanFactory
#####1.清空已有的BeanFactory
#####2.设置BeanFactory的个性化定制,主要是能否允许循环引用和允许加载不同的BeanDefinition（加载不同的BeanDefinition是干嘛用的）
#####3.加载BeanDefinition 待细分
###2.获取一个当前context配备的BeanFactory，必须实现ConfigurableListableBeanFactory

## prepareBeanFactory
###1.设置类加载器
###2.设置支持通配符的解析器？StandardBeanExpressionResolver
###3.自定义编辑加载器？ ResourceEditorRegistrar
###4.配置bean的上下文回调
###5.设置忽略的加载bean
###6.加载特殊bean
###7.注册提早初始化bean的事件发送器
###8.如果存在一个叫loadTimeWeaver的bean,则使用loadTimeWeaver 相关的解析器
###9.设置默认的环境bean

## postProcessBeanFactory
### 允许上下文的子类中bean工厂进行处理

## invokeBeanFactoryPostProcessors
### 调用在上下文中注册为bean的工厂处理器。

## registerBeanPostProcessors
### 注册bean处理器

## initMessageSource
### 初始化信息源

## initApplicationEventMulticaster
### 初始化多路广播

## onRefresh
### 对子的上下文中的特殊Bean进行初始化

## registerListeners
### 注册监听器

## finishBeanFactoryInitialization
###1.类型转换Service
###2.注册一个默认的bean处理器
###3.初始化LoadTimeWeaverAware 类型的bean??
###4.停止使用临时的类加载器
###5.冻结相应配合
###6.开始实例化剩下的bean






