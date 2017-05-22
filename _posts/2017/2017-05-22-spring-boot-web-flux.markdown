---
layout: "post"
title: "spring-boot -web-flux"
date: "2017-05-22 10:03"
---
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Spring5 :使用 spring webflux 开发 Reactive 应用](#spring5-使用-spring-webflux-开发-reactive-应用)
	- [1. 创建一个Spring boot项目](#1-创建一个spring-boot项目)
	- [2. 创建一个Handler来处理http](#2-创建一个handler来处理http)
	- [3. 创建一个Routes 类来定义应用的路由](#3-创建一个routes-类来定义应用的路由)
	- [4. 创建一个HttpeServerConfig 类来创建一个httpServer类](#4-创建一个httpeserverconfig-类来创建一个httpserver类)
	- [5. 创建一个Spring boot main 类来启动应用程序](#5-创建一个spring-boot-main-类来启动应用程序)
	- [6. 创建一个Testing  类来测试应用程序](#6-创建一个testing-类来测试应用程序)

<!-- /TOC -->

# Spring5 :使用 spring webflux 开发 Reactive 应用

```
 Spring5 -spring web-flux 有是一个新的非阻塞函数式的web框架.我们基于此可以构建异步的 非阻塞的 事件驱动的服务,并且扩展性也非常好

从非阻塞风格的代码迁移到函数式非阻塞响应式风格的代码中的业务逻辑可以作为异步的函数调用.可以采用以下方式:
1) 使用java8 方法引用
2) 使用lambda表达式.
由于线程是非阻塞的,处理能力被最大化使用.
spring5 仍处于一个里程碑的发布版本.
```


## 1. 创建一个Spring boot项目
1.创建项目

使用 Spring initializer 创建一个SPring boot 项目.将如下依赖加入pom.xml中
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```
SPring-boot-starter-webflux包中带了Spring-webflux//netty.其他项目需要自行添加.

2.建立一个简单的用户数据表和从list中获取到的user数据的DTO类.

  - 这仅仅是虚拟的数据bean,同时也可以从其他数据源类似于 RDBMS,MongoDB 或者RestClient 加载数据.
  - MongoDB 有一个响应式的客户端驱动
  - restClinet客户端也不会导致任何阻塞
  - 例外
    - 如今的jdbc 驱动天生不是影响式的,任何调用jdbc的驱动都仍然是线程阻塞的

```
public class User {
    public User(){}
    public User(Long id, String user) {
        this.id = id;
        this.user = user;
    }
    private Long id;
    private String user;
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getUser() { return user; }
    public void setUser(String user) { this.user = user; }

}

```

3.创建一个user操作类


```
@Repository
public class UserRepository {
    private final List<User> users = Arrays.asList(new User(1L, "User1"), new User(2L, "User2"));
    public Mono<User> getUserById(String id) {
        return Mono.justOrEmpty(users.stream().filter(user -> {
            return user.getId().equals(Long.valueOf(id));
        }).findFirst().orElse(null));
    }
    public Flux<User> getUsers() {
        return Flux.fromIterable(users);
    }

}
```


Mono和Flux 是由目标反应器提供的响应类型.Spring还提供其他响应流的实现,例如RXJave.Mono和Flux 是Reactive streams 的发布者实现.Mono是0或者任意单个值的发布,Flux是0到任意值的发布.他们和RXJava
中的Flowable和Observable类似.他们代替流向这些订阅者发布消息.

getUserById()返回一个Mono<User>对象,这个对象不论何时都会返回0~1个用户对象,GetUsers()返回一连串变动的用户对象,不论何时都包含0~n个用户对象.

相比命令式编程风格,我们并不返回可用前前阻塞线程的User/List<User>对象,而是只返回一个流的引用,流可以在后面访问User/List<User>.

## 2. 创建一个Handler来处理http

```
@Service
public class UserHandler {
@Autowired
private UserRepository userRepository;

public Mono<ServerResponse> handleGetUsers(ServerRequest request) {
return ServerResponse.ok().body(userRepository.getUsers(), User.class);
}

public Mono<ServerResponse> handleGetUserById(ServerRequest request) {
return userRepository.getUserById(request.pathVariable("id"))
.flatMap(user -> ServerResponse.ok().body(Mono.just(user), User.class))
.switchIfEmpty(ServerResponse.notFound().build());
}
}
```

- A Handler类,就如spring 容器中的写了业务逻辑的服务bean
  - ServerResponse 是一个服务实体,在spring的容器中
    - 包括响应数据
    - 响应状态码
    - 头信息等等
    - 包含默认有用的方法来支持不同类型的响应
      - notFound()
      - ok()
      - accepted()
      - create()
      - 等
 - UserHandler 有不同的方法返回Mono<serverResponse>
   -  UserRepository.getUsers() returns a Flux<User>; and ServerResponse.ok().body(UserRepository.getUsers(), User.class) 转换 this Flux<User> into Mono<ServerResponse> 任何时候他们将流的引用提交到ServerResponse
   - UserRepository.getUserById()  returns a Mono<User> and ServerResponse.ok().body(Mono.just(user), User.class) transforms this Mono<User> into Mono<ServerResponse>, which indicates a stream that emits ServerResponse whenever available.
   - ServerResponse.notFound().build()  returns a Mono<ServerResponse> which indicate a stream that emits 404 ServerResponse when no user is found with the given pathVariable.
 - 在命令式编程风格中,数据接收前线程会一直阻塞,这样使得其吓尿和功能在数据到来前无法运行.而响应式编程中,我们定义一个获取数据的流,然后定义一个在数据到来后的回调函数操作.这样不会使线程阻塞,在数据被返回时,可用线程就用于执行.

## 3. 创建一个Routes 类来定义应用的路由

```
import static org.springframework.web.reactive.function.server.RequestPredicates.GET;
import static org.springframework.web.reactive.function.server.RequestPredicates.accept;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;
@Configuration
public class Routes {
    private UserHandler userHandler;

    public Routes(UserHandler userHandler) {
           this.userHandler = userHandler;
    }

    @Bean
    public RouterFunction<?> routerFunction() {
      return route(GET("/api/user").and(accept(MediaType.APPLICATION_JSON)), userHandler::handleGetUsers)
        .and(route(GET("/api/user/{id}").and(accept(MediaType.APPLICATION_JSON)), userHandler::handleGetUserById));
  }
}
```
  RouterFunction 就如spring web中的@RequestMapping.在spring5 中RouterFunction定义路由.
  - RouterFunction helper 类有有用的方法
    - route
  - RequestPredicates 有许多有用的方法 来定义和构建RouterFunction
    - GET
    - post
    - path
    - querParam
    - accept
    - header
    - contentType
  - 当httpRequest接收到之后,每一条路由路径都会与处理器中的一个具体的方法匹配
  - Spring5 也同样支持@RequestMapping tpye of Controller
    - ```
      @GetMapping("/user") public Mono<ServerResponse> handleGetUsers() {}
    ```
    - RouterFunction provides the DSL 类型的路由方法 (与此同时,混合类型的路由目前spring还不支持)



## 4. 创建一个HttpeServerConfig 类来创建一个httpServer类
```
import org.springframework.http.server.reactive.HttpHandler;
import org.springframework.http.server.reactive.ReactorHttpHandlerAdapter;
import reactor.ipc.netty.http.server.HttpServer;
@Configuration
public class HttpServerConfig {
  @Autowired
  private Environment environment;

  @Bean
  public HttpServer httpServer(RouterFunction<?> routerFunction) {
    HttpHandler httpHandler = RouterFunctions.toHttpHandler(routerFunction);
    ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(httpHandler);
    HttpServer server = HttpServer.create("localhost", Integer.valueOf(environment.getProperty("server.port")));
    server.newHandler(adapter);
    return server;
  }
}
```
- 这创建了一个netty httpServer 在应用程序定义的配置文件端口号上.Spring同样支持类似于
  tomcat/undertow.由于netty是异步的基于事件驱动的天性,他就是一个很好的响应式应用.Tomcat 使用Java Nio 去实现servlet 规范.Netty是一个优化过的异步的/事件驱动的非阻塞IO的NIO的实现.

- tomcat服务也呢给你使用如下的代码构建:
  - ```
  Tomcat tomcatServer = new Tomcat();
  tomcatServer.setHostname("localhost");
  tomcatServer.setPort(Integer.valueOf(environment.getProperty("server.port")));
  Context rootContext = tomcatServer.addContext("", System.getProperty("java.io.tmpdir"));
  ServletHttpHandlerAdapter servlet = new ServletHttpHandlerAdapter(httpHandler);
  Tomcat.addServlet(rootContext, "httpHandlerServlet", servlet);
  rootContext.addServletMapping("/", "httpHandlerServlet");
  tomcatServer.start();
  ```

## 5. 创建一个Spring boot main 类来启动应用程序
```
@SpringBootApplication
public class Spring5ReactiveApplication {
public static void main(String[] args) throws IOException {
SpringApplication.run(Spring5ReactiveApplication.class, args);
}
}
```
## 6. 创建一个Testing  类来测试应用程序
- Http testing 测试工具
  - postman/或者curl来进行测试,
- 集成化测试

```
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
public class UserTest {
@Autowired
private WebTestClient webTestClient;

@Test
public void test() throws IOException {
FluxExchangeResult<User> result = webTestClient.get().uri("/api/user").accept(MediaType.APPLICATION_JSON)
.exchange().returnResult(User.class);
assert result.getStatus().value() == 200;
List<User> users = result.getResponseBody().collectList().block();
assert users.size() == 2;
assert users.iterator().next().getUser().equals("User1");
}

@Test
public void test1() throws IOException {
User user = webTestClient.get().uri("/api/user/1")
.accept(MediaType.APPLICATION_JSON).exchange().returnResult(User.class).getResponseBody().blockFirst();
assert user.getId() == 1;
assert user.getUser().equals("User1");
}

@Test
public void test2() throws IOException {
webTestClient.get().uri("/api/user/10").accept(MediaType.APPLICATION_JSON).exchange().expectStatus()
.isNotFound();
}
}
```
 - WebTestClient就像 TestRestTemplate类,在spring boot中有发起rest call并判断其返回值的方法.
 - 在测试配置中,Spring test 创建了一个TestRestTemplate的bean.他也是胰脏鞥呢webClient;就像spring web 的RestTemplate.他就可以进行响应式/非阻塞的测试
```
WebClient.create("http://localhost:9000").get().uri("/api/user/1")
        .accept(MediaType.APPLICATION_JSON).exchange().flatMap(resp -> resp.bodyToMono(User.class)).block();
```
  exchange()返回一个Mono<ClientResponse> 他代表了一个流引用.
  block() 阻塞线程直到Mono返回User/List<User>这是因为测试用例是需要返回值响应的.

  Spring web 是容易开发和调试的.不管桑踹鞥呢Spring5 的响应式 或者Spring Web 命令式服务按照用例场景都是很明智的.在许多场景下,命令式可能工作的更好,但是在一些场景下高可用是关键的业务场景下,响应式非阻塞或许是一个更好的选择.


[参考文档][eba9ca0e]

  1.[1]:https://www.oschina.net/translate/spring-5-reactive-web-services "基于spring5的响应式应用"

  2.[2]:http://www.google.com "Google"

  3.[3]:https://github.com/joqk12345/log "gibhub"

  4.[4]:https://dzone.com/articles/spring-5-reactive-web-services "响应式web服务"

  [数字时代的生存](http://www.cnblogs.com/joqk/)



跳转到[目录](#index)
