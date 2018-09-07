参考：
https://spring.io/guides/gs/service-registration-and-discovery/

下载：
git clone https://github.com/spring-guides/gs-service-registration-and-discovery.git

导入：gs-service-registration-and-discovery\initial

两个项目：eureka-service和eureka-client

一.eureka-service
1.添加文件：eureka-service/src/main/java/hello/EurekaServiceApplication.java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServiceApplication.class, args);
    }
}
2.添加配置文件：eureka-service/src/main/resources/application.properties
  server.port=8761

  eureka.client.register-with-eureka=false
  eureka.client.fetch-registry=false

  logging.level.com.netflix.eureka=OFF
  logging.level.com.netflix.discovery=OFF
  
二.eureka-client
需要自己添加依赖：
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
1.添加文件：eureka-client/src/main/java/hello/EurekaClientApplication.java
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}

@RestController
class ServiceInstanceRestController {

    @Autowired
    private DiscoveryClient discoveryClient;

    @RequestMapping("/service-instances/{applicationName}")
    public List<ServiceInstance> serviceInstancesByApplicationName(
            @PathVariable String applicationName) {
        return this.discoveryClient.getInstances(applicationName);
    }
}
2.添加配置文件：eureka-client/src/main/resources/bootstrap.properties
  spring.application.name=a-bootiful-client
  
（启动eureka-service 可以通过http://localhost:8761/访问，可以看到注册到eureka的服务：
Instances currently registered with Eureka
Application	AMIs	Availability Zones	Status）

首先启动eureka-service，然后在加载后启动eureka-client，测试端到端的结果。 
eureka-client将花费大约一分钟在注册表中注册，并从注册表中刷新自己的注册实例列表。 所有这些阈值都是可配置的。 
访问浏览器中的eureka-client，http：// localhost：8080 / service-instances / a-bootiful-client。 
在那里，您应该看到响应中反映的eureka-client的ServiceInstance。


