问题:ie8下,导入文件,服务端报错,前端接受不到一直在等待的问题.  
原因:ie8下传输文件流,前端是做成表单提交的方式,内嵌了一个iframe,response的状态码必须为200,不然iframe得不到返回数据.
解决方案:  
&emsp;&emsp;1.自己手动catch方法中异常,抛出一个正常的字符串
&emsp;&emsp;2.实现一个额外的filter,在filter中将返回的response status 改为200.  
&emsp;&emsp;3.添加一个额外的拦截器,将返回的response status 改为200.  
&emsp;&emsp;4.修改nginx配置,对特定的url将返回状态改为200  
&emsp;&emsp;5.重写SpringMVC的异常处理机制

方案分析:因为业务中已经存在一个异常处理器,会把返回的错误信息经过拦截,并且将拦截的response status设为相应的请求异常.所以对错误信息的返回,需要对原有异常处理器进行复用.因此对于方案1,在扩展性和复用性的上都不可取.原有实现如下

```
    @ExceptionHandler(Exception.class)
    @ResponseBody
 @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public String processControllerError(Exception ex) {
        
        return "服务异常";
    }
```

这导致了方案2,3将状态改为200后,不生效,因为在SpringMVC下,调用方式是 filter - interceptor - servlet - interceptor - filter,并且response的相关状态在一次请求中只会被设置一次,经过异常处理器之后,这个状态码就不会修改.而对于nginx来说,每次新添功能都要在nginx相关配置里面进行修改,各个环境之间需要过多重复劳动.因此最后选取了方案四,方案四对原先的异常处理进行了稍微重写,改动代码如下


```
    // controller抛出未知异常全局捕获
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public String processControllerError(Exception ex, HttpServletResponse response, HandlerMethod handlerMethod) {

        forIe8Import(HttpStatus.INTERNAL_SERVER_ERROR, response, handlerMethod);
        return "服务器异常";
    }

    private void forIe8Import(HttpStatus httpStatus, HttpServletResponse response, HandlerMethod handlerMethod){
        if (handlerMethod.getMethodAnnotation(ForIE8Import.class) != null){
            response.setStatus(HttpStatus.OK.value());
        } else {
            response.setStatus(httpStatus.value());
        }
    }
```
相应的使用方法只用加一个注解,就可实现返回错误的修改,减少重复代码.