最近有个工单说审核流程走不下去,经排查日志发现,内部报错,事务没有回滚造成的
伪码如下:  

```
public class Audit {

    @Transactional
    public void audit() {
        throw new RuntimeException("error");
    }

    public void batchAudit() {
        audit();
    }
}
```

在batchAudit中对audit进行调用,其中audit方法加上了@Transactional,但是其报错没有对事务进行回滚.原因是因为@Transactional是基于切面进行事务的添加,而切面的内部是用代理进行操作的,因此batchAudit在内部调用audit方法时,只是经过内部引用,而不走相应的代理类,导致事务没有回滚.  
另外@Transactional还有一些使用注意事项如下:

1. 由于@Transactional是基于切面的,所以只有经过外部调用的方式才能生效,内部应用的方式是不会有事务的
2. @Transactional 不要放在类声明上,事务本质就是对多次写操作而使用的,若查询或者单次操作加上事务,会对程序性能有一定影响
3. @Transactional 默认只回滚抛出受检查异常(继承自RuntimeException的异常)