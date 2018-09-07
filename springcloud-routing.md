参考：
https://spring.io/guides/gs/routing-and-filtering/

导入gs-routing-and-filtering\initial

两个项目：
gateway和book

一.book
1.添加文件book/src/main/java/hello/BookApplication.java
@RestController
@SpringBootApplication
public class BookApplication {

  @RequestMapping(value = "/available")
  public String available() {
    return "Spring in Action";
  }

  @RequestMapping(value = "/checked-out")
  public String checkedOut() {
    return "Spring Boot in Action";
  }

  public static void main(String[] args) {
    SpringApplication.run(BookApplication.class, args);
  }
}
2.添加文件book/src/main/resources/application.properties
  spring.application.name=book
  server.port=8090
  
二.gateway
1. 添加filter gateway/src/main/java/hello/filters/pre/SimpleFilter.java
public class SimpleFilter extends ZuulFilter {

  private static Logger log = LoggerFactory.getLogger(SimpleFilter.class);

  @Override
  public String filterType() {
    return "pre";//filter 类型
  }

  @Override
  public int filterOrder() {
    return 1;//如果多个filter，该filter执行的顺序
  }

  @Override
  public boolean shouldFilter() {
    return true;//包含确定何时执行此过滤器的逻辑（将始终执行此特定过滤器）。
  }

  @Override
  public Object run() {
    RequestContext ctx = RequestContext.getCurrentContext();
    HttpServletRequest request = ctx.getRequest();

    log.info(String.format("%s request to %s", request.getMethod(), request.getRequestURL().toString()));

    return null;
  }
}
说明：
Now let’s see how we can filter requests through our proxy service. Zuul has four standard filter types:
pre filters：请求路由之前执行
route filters：处理请求的实际路由
post filters：请求被路由之后执行
error filters：处理请求的时候如果出错了执行

2.添加文件gateway/src/main/java/hello/GatewayApplication.java
@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {

  public static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
  }

  @Bean
  public SimpleFilter simpleFilter() {
    return new SimpleFilter();
  }
}
3. 添加配置文件gateway/src/main/resources/application.properties
  zuul.routes.books.url=http://localhost:8090
  ribbon.eureka.enabled=false
  server.port=8080
  
  启动后，就可以通过localhost:8080/books/available访问到book localhost:8090.
  filter后台有日志：
  2018-09-07 16:14:35.931  INFO 24852 --- [nio-8080-exec-5] hello.SimpleFilter : GET request to http://localhost:8080/books/availa
ble
