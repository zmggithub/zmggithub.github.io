#### nacos配置中心

 配置服务接口，可以理解顶级接口Nacos配置中心核心API，包含如下操作nacos方法：

```java
 interface ConfigService{
 
    getConfig
    getConfigAndSignListener
    addListener
    publishConfig
    publishConfig
    publishConfigCas
    publishConfigCas
    removeConfig
    removeListener
    getServerStatus
    shutDown
 }
```

CacheData是监听管理重要的类

```
CacheData 
```