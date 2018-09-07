
参考：
https://spring.io/guides/gs/client-side-load-balancing/

下载代码：
git clone https://github.com/spring-guides/gs-client-side-load-balancing.git

代码导入eclipse：
gs-client-side-load-balancing\initial

两个项目：say-hello和user

一.服务端
1.添加代码say-hello/src/main/java/hello/SayHelloApplication.java
@RestController
@SpringBootApplication
public class SayHelloApplication {

  private static Logger log = LoggerFactory.getLogger(SayHelloApplication.class);

  @RequestMapping(value = "/greeting")
  public String greet() {
    log.info("Access /greeting");

    List<String> greetings = Arrays.asList("Hi there", "Greetings", "Salutations");
    Random rand = new Random();

    int randomNum = rand.nextInt(greetings.size());
    return greetings.get(randomNum);
  }

  @RequestMapping(value = "/")
  public String home() {
    log.info("Access /");
    return "Hi!";
  }

  public static void main(String[] args) {
    SpringApplication.run(SayHelloApplication.class, args);
  }
}
2.添加配置文件application.properties
  server.port=8090
  spring.application.name=say-hello
二.服务端
1.添加代码user/src/main/java/hello/UserApplication.java
@SpringBootApplication
@RestController
public class UserApplication {

  @Bean
  RestTemplate restTemplate(){
    return new RestTemplate();
  }

  @Autowired
  RestTemplate restTemplate;

  @RequestMapping("/hi")
  public String hi(@RequestParam(value="name", defaultValue="Artaban") String name) {
    String greeting = this.restTemplate.getForObject("http://localhost:8090/greeting", String.class);
    return String.format("%s, %s!", greeting, name);
  }

  public static void main(String[] args) {
    SpringApplication.run(UserApplication.class, args);
  }
}
2.添加配置文件application.properties
  spring.application.name=user
  server.port=8888
  
 启动两个服务，访问：
 $ curl http://localhost:8888/hi
  Greetings, Artaban!

$ curl http://localhost:8888/hi?name=Orontes
  Salutations, Orontes!
  
三.由单点到负载均衡，配置Ribbon
1. 修改客户端配置user/src/main/resources/application.properties，添加如下配置：
  say-hello.ribbon.eureka.enabled=false
  say-hello.ribbon.listOfServers=localhost:8090,localhost:9092,localhost:9999
  say-hello.ribbon.ServerListRefreshInterval=15000
2. 客户端添加文件user/src/main/java/hello/SayHelloConfiguration.java
  public class SayHelloConfiguration {

  @Autowired
  IClientConfig ribbonClientConfig;

  @Bean
  public IPing ribbonPing(IClientConfig config) {
    return new PingUrl();
  }

  @Bean
  public IRule ribbonRule(IClientConfig config) {
    return new AvailabilityFilteringRule();
  }
}
3. 修改客户端代码user/src/main/java/hello/UserApplication.java
@SpringBootApplication
@RestController
@RibbonClient(name = "say-hello", configuration = SayHelloConfiguration.class)
public class UserApplication {

  @LoadBalanced
  @Bean
  RestTemplate restTemplate(){
    return new RestTemplate();
  }

  @Autowired
  RestTemplate restTemplate;

  @RequestMapping("/hi")
  public String hi(@RequestParam(value="name", defaultValue="Artaban") String name) {
    String greeting = this.restTemplate.getForObject("http://say-hello/greeting", String.class);
    return String.format("%s, %s!", greeting, name);
  }

  public static void main(String[] args) {
    SpringApplication.run(UserApplication.class, args);
  }
}
4. 把服务端 say-hello 打成可执行的jar
$ mav clean package
5. 按照user/src/main/resources/application.properties中配置的端口启动say-hello
  $ java -jar say-hello-0.0.1-SNAPSHOT.jar --server.port=8090
  $ java -jar say-hello-0.0.1-SNAPSHOT.jar --server.port=9092
  $ java -jar say-hello-0.0.1-SNAPSHOT.jar --server.port=9999
  
 6. 启动user访问localhost:8888/hi
 
从服务的控制台可以看到Ribbon每一个一段时间（user/src/main/resources/application.properties中配置的）就会ping一次服务：
2018-09-07 11:02:09.247  INFO 27604 --- [nio-8090-exec-8] hello.SayHelloApplication                : Access /
2018-09-07 11:02:22.390  INFO 27604 --- [nio-8090-exec-5] hello.SayHelloApplication                : Access /

2018-09-07 11:02:09.269  INFO 26272 --- [nio-9999-exec-5] hello.SayHelloApplication                : Access /
2018-09-07 11:02:22.419  INFO 26272 --- [nio-9999-exec-7] hello.SayHelloApplication                : Access /

2018-09-07 11:02:09.258  INFO 19320 --- [nio-9092-exec-7] hello.SayHelloApplication                : Access /
2018-09-07 11:02:22.409  INFO 19320 --- [nio-9092-exec-9] hello.SayHelloApplication                : Access /




