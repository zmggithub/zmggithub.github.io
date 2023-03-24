#### nacos配置中心获取配置内容

##### 一、问题描述

最近要搞分布式配置中心，然后技术选型方面考虑了spring-cloud-config和阿里开源的nacos。
但是在项目代码和nacos完全启动后，发现并没有访问到nacos中的配置项。

##### 二、解决方案

###### 1.检查配置文件

首先要保证项目中的bootstrap.yml配置文件存在，且里面的配置正确。

```yaml
#application.yml
spring:
  profiles:
    active: dev
```

```yaml
#application-dev.yml
spring:
  application:
    name: CommonServer
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.153.190:8848
        namespace: zhumingguang
      config:
        server-addr: 192.168.153.190:8848
        namespace: zhumingguang
        enabled: true  #是否开启注册 默认true开启
        prefix: com.zmg.common  #DataId 名称（默认就是服务名称）
        # file-extension: yml # 此处为配置使用的后缀名默认properties文件，这里指定spring.profiles.active=dev，一定要指定为.yml格式配置文件。
        group: zmgcommon #分组名称 默认DEFAULT_GROUP
```

###### 2.nacos中配置文件名规则

在 nacos中，dataId 的完整格式如下：

```java
${prefix}-${spring.profile.active}.${file-extension}
```

1. prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。
2. spring.profile.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profile.active 为空时，对应的连接符-也将不存在，dataId 的拼接格式变成${prefix}.${file-extension}
3. file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持properties 和yml 类型。

配置如上的话文件名示例：WorkflowServer-dev.yml



##### 三、示例

所以如果以上面我的配置文件为标准写dataId的话，应该为:CommonServer-dev.yml。即： 

![nacos-dev-yaml](C:\Users\zmg\Desktop\private\Nacos\photo\nacos-dev-yaml.png)



##### **四、单机启动**

```yaml
#*************** Config Module Related Configurations ***************#

### If use MySQL as datasource:
spring.datasource.platform=mysql
### Count of DB:
db.num=1
### Connect URL of DB:
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=123456
```

启动命令(standalone代表着单机模式运行，非集群模式): `sh startup.sh -m standalone`



##### 五、集群部署

端口号会占用2个，如8846会占用8846和8847，最好隔一个。

虚拟机中vm参数：-Xms512 -Xmx512m -Xmn256 -XX:MetaspaceSize=64m -XX:MaxMetaspaceSize=128m