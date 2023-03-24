
服务提供者consumer,项目启动时报出警告，因为此时只有一个dubbo的bean：

```java
WARN 14736 --- [main] b.f.a.ServiceAnnotationBeanPostProcessor :  [DUBBO] No Spring Bean annotating Dubbo's @Service was found under package[xxx], dubbo version: 2.7.1, current host:....
```

消费者provider启动或调用时报错：

```java
 Error creating bean with name 'xxx': Injection of @org.apache.dubbo.config.annotation.Reference dependencies is failed; nested exception is org.apache.dubbo.rpc.RpcException: Fail to create remoting client for service(.......
```

或：

```java
org.apache.dubbo.rpc.RpcException: Failed to invoke remote method: xxx, provider: xxxx, cause: message can not send, because channel is closed .
```

原因：

```java
import org.apache.dubbo.config.annotation.Service
```
与
```java
import org.springframework.stereotype.Service 
```

是同名的两个@Service注解，使用时要注意。使用dubbo提供的注解才能实现rpc调用。


参考资料：[https://dubbo.apache.org/zh/docs3-v2/java-sdk/quick-start/spring-boot/](https://dubbo.apache.org/zh/docs3-v2/java-sdk/quick-start/spring-boot/)
git:
[https://github.com/apache/dubbo-samples/tree/master/dubbo-samples-spring-boot](https://github.com/apache/dubbo-samples/tree/master/dubbo-samples-spring-boot)