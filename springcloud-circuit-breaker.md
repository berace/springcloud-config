参考：
https://spring.io/guides/gs/circuit-breaker/
下载：
git clone https://github.com/spring-guides/gs-circuit-breaker.git
导入：
gs-circuit-breaker\initial

两个项目：bookstore和reading

一. bookstore
1. 添加文件bookstore/src/main/java/hello/BookstoreApplication.java
@RestController
@SpringBootApplication
public class BookstoreApplication {

  @RequestMapping(value = "/recommended")
  public String readingList(){
    return "Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)";
  }

  public static void main(String[] args) {
    SpringApplication.run(BookstoreApplication.class, args);
  }
}
2.添加配置文件：bookstore/src/main/resources/application.properties
  server.port=8090
  
二.reading
1.添加文件reading/src/main/java/hello/ReadingApplication.java

@RestController
@SpringBootApplication
public class ReadingApplication {

  @RequestMapping("/to-read")
  public String readingList() {
    RestTemplate restTemplate = new RestTemplate();
    URI uri = URI.create("http://localhost:8090/recommended");

    return restTemplate.getForObject(uri, String.class);
  }

  public static void main(String[] args) {
    SpringApplication.run(ReadingApplication.class, args);
  }
}

2.添加配置文件：reading/src/main/resources/application.properties
  server.port=8080
  
我们现在可以在浏览器中访问Reading应用程序上的/ to-read端点，并查看我们的阅读列表。 
然而，由于我们依赖Bookstore应用程序，如果发生任何事情，或者如果Reading无法访问Bookstore，
我们将没有列表，我们的用户将收到一个令人讨厌的HTTP 500错误消息。

三.设置Circuit Breaker
1.添加文件reading/src/main/java/hello/BookService.java
@Service
public class BookService {

  private final RestTemplate restTemplate;

  public BookService(RestTemplate rest) {
    this.restTemplate = rest;
  }

  @HystrixCommand(fallbackMethod = "reliable")
  public String readingList() {
    URI uri = URI.create("http://localhost:8090/recommended");

    return this.restTemplate.getForObject(uri, String.class);
  }

  public String reliable() {
    return "Cloud Native Java (O'Reilly)";
  }
}
2.修改reading/src/main/java/hello/ReadingApplication.java
@EnableCircuitBreaker
@RestController
@SpringBootApplication
public class ReadingApplication {

  @Autowired
  private BookService bookService;

  @Bean
  public RestTemplate rest(RestTemplateBuilder builder) {
    return builder.build();
  }

  @RequestMapping("/to-read")
  public String toRead() {
    return bookService.readingList();
  }

  public static void main(String[] args) {
    SpringApplication.run(ReadingApplication.class, args);
  }
}

Try it out
Run both the Bookstore service and the Reading service, and then open a browser to the Reading service, at localhost:8080/to-read.
You should see the complete recommended reading list:

Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)

Now shut down the Bookstore application. Our list source is gone, but thanks to Hystrix and Spring Cloud Netflix, 
we have a reliable abbreviated list to stand in the gap; you should see:

Cloud Native Java (O'Reilly)
