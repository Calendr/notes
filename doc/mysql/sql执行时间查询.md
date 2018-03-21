设置profiling       

SET profiling=1;


执行sql SELECT * FROM table

查看运行结果

SHOW profiles;

接着用  
SHOW profile FOR QUERY queryId

最后关闭 set profiling=0
