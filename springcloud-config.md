# springcloud-config
参考：
https://spring.io/guides/gs/centralized-configuration/

下载
git clone https://github.com/spring-guides/gs-centralized-configuration.git

eclipse maven 导入gs-centralized-configuration\initial

一、服务端
0、读取GitHub上配置文件
添加配置文件：a-bootiful-client.properties
  xing=wei
  ming=jinhua
  sheng=shandong
  shi=heze caoxian
  message=hello,liu

1、创建java文件：
configuration-service/src/main/java/hello/ConfigServiceApplication.java

@EnableConfigServer
@SpringBootApplication
public class ConfigServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServiceApplication.class, args);
    }
}
2、配置文件
src/main/resources下添加配置文件application.properties
  #配置服务访问端口
  server.port=8888
  #a-bootiful-client.properties所在目录
  spring.cloud.config.server.git.uri=https://github.com/berace/springcloud-config
  
  此时启动服务端，可以通过地址访问到配置：http://localhost:8888/a-bootiful-client.properties
  
  二、客户端
  1、添加配置文件
  configuration-client/src/main/resources/bootstrap.properties
    spring.application.name=a-bootiful-client
    # N.B. this is the default:
    spring.cloud.config.uri=http://localhost:8888
  为了能动态刷新配置，添加配置文件
  configuration-client/src/main/resources/application.properties
    management.endpoints.web.exposure.include=*
    
   2、java文件configuration-client/src/main/java/hello/ConfigClientApplication.java
   
   By default, the configuration values are read on the client’s startup, and not again. You can force a bean to refresh its configuration - to pull updated values from the Config Server - by annotating the MessageRestController with the Spring Cloud Config @RefreshScope and then by triggering a refresh event.
   
   @SpringBootApplication
public class ConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }
}

@RefreshScope
@RestController
class MessageRestController {

    @Value("${message:Hello default}")
    private String message;
   
    @Value("${xing}")
    private String xing;
    @Value("${ming}")
    private String ming;
    @Value("${sheng}")
    private String sheng;
    @Value("${shi}")
    private String shi;

    @RequestMapping("/message")
    String getMessage() {
        return this.message;
    }
    @RequestMapping("/detail")
    String getDetail() {
    	return this.sheng+" "+this.shi + "," + this.xing+" "+this.ming;
    }
}
  
启动客户端，通过地址访问http://localhost:8080/detail或者http://localhost:8080/message
即可返回配置的value。

动态修改a-bootiful-client.properties，不需要任何重启。
需要手动刷新：http://localhost:8080/actuator/refresh（post方式访问）

此时再去访问http://localhost:8080/detail即可看到最新的配置


