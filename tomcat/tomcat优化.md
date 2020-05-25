### tomcat内存优化



### tomcat并发优化

```xml
 <Connector port="8090" protocol="org.apache.coyote.http11.Http11NioProtocol"
			   maxThreads="600"	
			   maxIdleTime="500"
			   acceptCount="1000"
               connectionTimeout="20000"
               redirectPort="8443" />
```



#### 参数

maxThreads最大线程数

acceptCount

最大等待数，超过拒绝连接connect reject

#### I/O三种模式

bio

nio

apr



### 压测

#### 异常



```
Thread Name: tomcat-performance Thread Group 1-184
Sample Start: 2020-04-30 15:00:25 CST
Load time: 29
Connect Time: 29
Latency: 0
Size in bytes: 2531
Sent bytes:0
Headers size in bytes: 0
Body size in bytes: 2531
Sample Count: 1
Error Count: 1
Data type ("text"|"bin"|""): text
Response code: Non HTTP response code: java.net.BindException
Response message: Non HTTP response message: Address already in use: connect

HTTPSampleResult fields:
ContentType: 
DataEncoding: null
```

linux 提供给TCP/IP链接的端口为 1024（ windows为 1024-5000）个，并且要四分钟来循环回收它们，就导致我们在短时间内跑大量的请求时将端口占满了，导致如上报错。

解决办法（在jmeter所在服务器操作）：

1.cmd中输入regedit命令打开注册表；

2.在 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters右键Parameters；

3.添加一个新的DWORD，名字为MaxUserPort；

4.然后双击MaxUserPort，输入数值数据为65534，基数选择十进制；

5.完成以上操作，务必重启机器，问题解决

#### ab压测