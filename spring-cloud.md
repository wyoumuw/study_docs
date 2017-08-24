# spring-cloud
## actuator
import  maven :spring-boot-starter-actuator
### introduction
监控端点情况
其中各个端点的敏感信息如下,敏感的信息要加 **management.security.enabled=false**  才能访问

| ID	| 描述	| 敏感（Sensitive）|
| ------------- |-------------| :-----:|
|autoconfig	|显示一个auto-configuration的报告，该报告展示所有auto-configuration候选者及它们被应用或未被应用的原因	|true|
|beans	|显示一个应用中所有Spring Beans的完整列表	|true|
|configprops	|显示一个所有@ConfigurationProperties的整理列表	|true|
|dump	|执行一个线程转储	|true|
|env	|暴露来自Spring　ConfigurableEnvironment的属性	|true|
|health	|展示应用的健康信息（当使用一个未认证连接访问时显示一个简单的’status’，使用认证连接访问则显示全部信息详情）	|false|
|info	|显示任意的应用信息	|false|
|metrics	|展示当前应用的’指标’信息	|true|
|mappings	|显示一个所有@RequestMapping路径的整理列表	|true|
|shutdown	|允许应用以优雅的方式关闭（默认情况下不启用）	|true|
|trace	|显示trace信息（默认为最新的一些HTTP请求）	|true|
#### 应用配置类
用于检测应用配置属性（静态数据）
* /autoconfig:查看自动配置
* /beans:容器中的bean
* /configprops:配置属性
* /env:包括环境变量、jvm属性、应用的配置属性、命令行中的参数
* /info:显示在application.properties下定义的info前缀的属性(自定义属性)
#### 度量指标类
用于检测应用实际运行中的动态数据
* /metrics:显示应用指标，还可以注入CounterService进自己的controller里使用increment(属性名)来添加属性进metrics里并增加计数器，当然GaugeService也可以注入。并且可以使用/metrics/mem.free(可以查mem.*来找组)来进行细粒度查询

      {
          "mem": 总内存,
          "mem.free": 可用内存,
          "processors": 处理器内核数量,
          "instance.uptime": 运行时间,
          "uptime": 运行时间,
          "systemload.average": 负载均衡
          "heap.committed": 192000,#heap.* 堆内存使用情况
          "heap.init": 126976,
          "heap.used": 107023,
          "heap": 1804288,
          "nonheap.committed": 40640,#nonheap.* 非堆内存使用情况
          "nonheap.init": 2496,
          "nonheap.used": 39669,
          "nonheap": 0,
          "threads.peak": 25,#threads.* 线程使用情况
          "threads.daemon": 21,
          "threads.totalStarted": 27,
          "threads": 23,
          "classes": 5682,#classes.* 类加载情况
          "classes.loaded": 5682,
          "classes.unloaded": 0,
          "gc.ps_scavenge.count": 回收次数,#gc.* gc情况
          "gc.ps_scavenge.time": 回收耗时,
          "gc.ps_marksweep.count": 标记-清除算法的次数,
          "gc.ps_marksweep.time": 标记-清除算法的耗时,
          "httpsessions.max": 最大会话数,#httpsessions。* 应用服务器容器回话使用情况，该指标仅在嵌入式容器才会提供
          "httpsessions.active": 激活中的回话,
          "gauge.response.metrics":5 ,#guage.* http请求性能 例如guage.response.metrices就是我请求/metrics的耗时为多少毫秒
          "counter.status.200.metrics": 2 #counter.* http请求计数 例如counter.status.200.metrics就是我请求/metrics返回200的次数
      }

* /health:获取应用的健康指标。检测器是实现了HealthIndicator接口实现，可以自定义这个接口并放入容器内springboot会自动检测。
      
      @Component
      public class MyHealthIndicator implements HealthIndicator{
      	@Override
      	public Health health() {
      		int errorCode=dealing();
      		if(errorCode==0){
      			return Health.down().withDetail("youmu error",errorCode).build();
      		}
      		return Health.up().withDetail("youmustatus",errorCode).build();
      	}
      
      	private int dealing(){
      		if(new Date().getTime()%2==0){
      			return new Random().nextInt();
      		}
      		return 0;
      	}
      }

* /dump:显示线程信息
* /trace:追踪最后100条http请求

#### 控制类
此类需要开启才能被使用
* [post]/shutdown:需要配置  **endpoints.shutdown.enabled=true** 才能使用，会关闭该端点

## eureka [uriker]
### 杂言
* 一台机器一个服务只能注册一个，他是用eureka.instance.instanceId来区分,他默认的命名规则为${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance—id:$ {server.port}}
* mvc的Interceptor无法拦截actuator
* 端点

### 配置
        org.springframework.cloud.netflix.eureka.server.EurekaServerConfigBean//eureka服务配置类对应eureka.server.*
        org.springframework.cloud.netflix.eureka.EurekaClientConfigBean //服务注册配置类对应eureka.client.*
        org.springframework.cloud.netflix.eureka.EurekalnstanceConfigBean//对应eureka.instance.*
        
        
* eureka.client.registry-fetch-interval-seconds:客户端重新拉取服务清单的间隔
* eureka.instance.lease-expiration-duration-in-seconds:服务实例租约过期时间
* eureka.instance.lease-renewal-interval-in-seconds:服务实例重新续约间隔
* eureka.client.service-url.defaultZone:服务注册中心地址(多个用,分隔) 示例=> http://localhost:10001/eureka/
* eureka.instance.hostname:服务实例地址，注册中心会ping这个地址去查询是否能通，默认用域名，要使用ip需要开启
* eureka.instance.prefer-ip-address:开启ip访问
* eureka.server.enable-self-preservation:是否开启自我保护



注入下面信息可以获取当前client信息

            @Autowired
            private DiscoveryClient discoveryClient;
            @Autowired
            private Registration registration;
            usage:List<ServiceInstance> serviceInstances=discoveryClient.getInstances(registration.getServiceId());
            
#### 注意:配置集群的时候必须appname相同，并且设置hostname，这个名字要ping的通。
#### 错误:EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
该状态持续很久，访问该服务也返回错误，但在注册中心界面，该服务却一直存在，且为UP状态，并且在大约十分钟后，出现一行红色大字：EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
原因：自我保护机制。Eureka Server在运行期间，会统计心跳失败的比例在15分钟之内是否低于85%，如果出现低于的情况（在单机调试的时候很容易满足，实际在生产环境上通常是由于网络不稳定导致），Eureka Server会将当前的实例注册信息保护起来，同时提示这个警告。
解决方法：
添加如下配置，关闭自我保护
Eureka server application.yml
eureka:
server:
enableSelfPreservation: false  
Service application.yml
eureka:
instance:
leaseRenewalIntervalInSeconds: 1
leaseExpirationDurationInSeconds: 2

##RIBBON
###retry
* spring.cloud.loadbalancer.retry.enabled=true  :开启重试机制
* hystrix.command.default.execution.isolation.thread.timeoutinMilliseconds = 10000 :断路器的超时时间需要大于Ribbon的超时时间， 不然不会触发重试。
* hello-service.ribbon.ConnectTimeout = 250 :请求连接的超时时间。
* hello-service.ribbon.ReadTimeout = 1000 :请求处理的超时时间。
* hello-service.ribbon.OkToRetryOnAllOperations = true :对所有操作请求都进行重试
* hello-service.ribbon.MaxAutoRetriesNextServer = 2 :切换实例的重试次数
* hello-service.ribbon.MaxAutoRetries = 1 :对当前实例的重试次数。
* ribbon.eureka.enabled=false :是否开启ribbon对服务的自理
* {serviceId}.ribbon.listOfServers=xxxx,xxxx :手动设置某个服务的地址，


## Hystrix
做断路器，服务降级--异常处理，请求缓存，请求合并
commandProperties:com.netflix.hystrix.HystrixCommandProperties.可以参考https://yq.aliyun.com/articles/61510


##Feign
           org.springframework.cloud.netflix.feign.FeignClientsConfiguration

##Stream
spring.cloud.stream.bindings.input.destination:可以用来动态指定队列的名字（kafka的topic或者rabbit的exchange）

##### ps:问题小计
1. @ConfigurationProperties
2. Actuator中env加密;参考69页

        //可以修改端点路径
        endpoints.info.path=/appinfo
        endpoints.health.path=/checkHealth

3. actuator端点路径的配置---》配置management.context-path

         //此时要修改eureka的配置
        eureka.instance.statusPageUrlPath=${management.context-path}/info
        eureka.instance.healthCheckUrlPath=${management.context-path}/health

4. 为什么EUREKA注册中心会向自己注册
5. 服务注册中心的安全配置:http://<username>:<password>@localhost:1111/eureka/
6. hystrix的分组，线程隔离
7. 监测restTemplate超时时间
8. discoveryClient

