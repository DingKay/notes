# Spring Boot + Vue

* 为了使开发者能够快速上手Spring，利用Spring框架快速搭建Java EE项目，Spring Boot应运而生。Spring Boot中对一些常用的第三方库提供了默认的自动化配置方案，使得开发着只需要很少的Spring配置就能运行一个完整的Java EE应用。

**Spring Boot 主要有如下优势**：

提供一个快速的Spring 项目搭建渠道

开箱即用，很少的Spring配置就能运行一个Java EE项目

提供了生产级的服务监控方案

内嵌服务器，可以快速部署

提供了一系列非功能性的通用配置

纯Java配置，没有代码生成，也不需要XML配置

## 1.0 创建Maven工程

### 1.1 使用Maven命令创建

本地（win10）安装Maven，并将`bin`目录添加至`path`环境变量

```dos
mvn archetype:generate -DgroupId=com.dk -DartifactId=chapter01 -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

命令解释

` * -DgroupId 组织Id（项目包名）`  

` * -DartifactId artifactId 项目名称或者模块名称`

` * -DarchetypeArtifactId 项目骨架`

` * -DinteractiveMode 是否使用交互模式`

打开指令，输入上述命令

![](../images/Spring Boot + Vue/mvn创建springboot工程.png)

使用Eclipse创建Maven工程

略

使用IntelliJ IDEA 创建Maven工程

略

### 1.2 项目构建

#### 1.2.1 添加依赖

首先添加`spring-boot-starter-parent`做为*parent*

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
</parent>
```

添加Web的starter

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

#### 1.2.2 编写启动类

```java
@EnableAutoConfiguration
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

* @EnableAutoConfiguration 注解表示开启自动化配置，由于项目中添加了 `spring-boot-starter-web` 依赖，因此开启了自动化配置之后会自动进行 *spring* 和 *spring MVC* 的配置
* 此Java项目的*main*中，通过*SpringApplication* 中的*run*方法启动项目，第一个参数*App.class* 告诉*Spring*哪个是主要组件，第二个参数是运行时输入的其他参数。

接下来创建*Controller*

```java
@RestController
public class HelloController {

    @GetMapping("hi")
    public String hello() {
        return "Hello, Spring Boot!";
    }
}
```

* @RestController 注解声明了这个controller是一个restFul接口
* @GetMapping 注解声明了一个请求的uri

此时运行项目，并不能访问` http://localhost:8080/hi `原因是Spring Boot并没有生成这个Controller实例；

需要配置包扫描才能将*HelloController* 注册到 *Spring MVC*的容器中

```java
@EnableAutoConfiguration
@ComponentScan()
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

此时如果仍然无法访问，则需要检查启动类 *App* 与 *HelloController* 的包层级是否错误

![](../images/Spring Boot + Vue/启动类与HelloController层级错误.png)

此时需要手动修改扫包注解中的 *value* 值

```java
@ComponentScan(value = "com.dk.controller")
```

如若不修改 *value* 值，则默认扫描启动类 *App*包的路径子路径（包括当前路径）

启动类的两个注解可以用 *SpringBootApplication* 替换

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

### 1.3 项目启动

启动项目有三种不同的方式

#### 1.3.1 使用Maven启动

可以直接使用如下命令启动：

```powershell
mvn spring-boot:run
```

#### 1.3.2 直接运行main方法

直接在IDE中运行App类的main方法；

![](../images/Spring Boot + Vue/mvn启动springboot.png)

#### 1.3.3 打包启动

Spring Boot 也可以直接打成Jar包运行，首先需要添加一个*plugin*到 *pom.xml*

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

然后运行`mvn package` 打包命令

打包完成后，在项目 *target* 目录下生成了一个*Jar*文件，通过*java -jar*命令直接启动；

## 2.0 Spring Boot 基础配置

### 2.1 不使用spring-boot-starter-parent

*spring-boot-starter-parent* 主要提供了以下默认配置

* Java版本默认使用1.8
* 编码格式默认使用UTF-8
* 提供*Dependency Management* 进行项目依赖的版本管理
* 默认的资源过滤与插件配置

使用*Dependency Management*

```xml
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.0.4.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
```

编码格式

```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>
```

添加一个*plugin*

```xml
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
      <source>1.8</source>
      <target>1.8</target>
    </configuration>
  </plugin>
```

### 2.2 @SpringBootApplication

@SpringBootApplication是一个组合注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
	// ...
}
```

这个注解由三个注解组成

1. @SpringBootConfiguration

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}
```

原来就是一个@Configuration，所以@SpringBootApplication的功能就是表明这是一个配置类，可以在这个类配置Bean，这个类所扮演的角色有点类似于Spring中的applicationContext.xml文件的角色

2. @EnableAutoConfiguration表示开启自动化配置，SpringBoot中的自动化配置是*非侵入式*的，在任意时刻，都可以自定义配置代替自动化配置中的某一个配置
3. @ComponentScan包扫描，默认扫描当前类所在的包路径

虽然@SpringBootApplication包含了@Configuration注解，但是我们可以创建一个新类来配置Bean，这样便于配置的管理，这个类加上@ComponentScan注解即可

项目中的@ComponentScan注解除了会扫描@Service、@Repository、@Component、@Controller和@RestController等之外，也会扫描@Configuration注解的类

### 2.3 定制banner

在resource路径下创建*banner.txt*文件，即可定制启动banner

想要关闭*banner*也是可以的，修改启动类的*main*方法

```java
SpringApplicationBuilder builder = new SpringApplicationBuilder(App.class);
builder.bannerMode(Banner.Mode.OFF).run(args);
```

### 2.4 Web容器配置

#### 2.4.1 Tomcat配置

1. 常规配置

在Spring Boot 项目中，可以内置*Tomcat*、*Jetty* 、*Undertow* 、*Netty*等容器，当添加了*spring-boot-starter-web*依赖后，会默认使用*Tomcat*做为Web容器，对*Tomcat*做进一步配置，创建*application.properties*

```properties
## tomcat端口号
server.port=8080
## 错误页跳转路径
server.error.path=/error
## session失效时间，不加单位默认是秒
## 且由于Tomcat配置失效时间为分钟，因此这里如果是秒，则被转换成一个不超过所配置秒数的最大分钟数
server.servlet.session.timeout=30m
## 项目路径，不配置默认为 /
server.servlet.context-path=/chapter04
## uri请求编码
server.tomcat.uri-encoding=UTF-8
## tomcat最大线程数
server.tomcat.max-threads=20
## tomcat存放运行日志和临时文件的基本路径，若不配置，则默认使用系统的临时目录
server.tomcat.basedir=/home/kay/tmp
```

2. HTTPS配置

由于HTTPS具有良好的安全性，在开发中得到广泛的应用，在JDK中提供了一个Java数字证书管理工具*keytool*，在*\jdk\bin*目录下，通过这个工具可以自己生成一个数字证书

```powershell
keytool -genkey -alias kay -keyalg RSA -keysize 2048 -keystore kay.p12 -validity 365
```

命令解释

* -genkey 创建一个新密钥
* -alias keystore的别名
* -keyalg 表示加密算法RSA，一种非对称加密算法
* -keysize 密钥长度
* -keystore 生成的密钥存放位置
* -validity 密钥有时间，单位：天

在*application.properties*文件中新增配置

```xml
## 证书别名
server.ssl.key-alias=kay
## 证书文件名
server.ssl.key-store=classpath:kay.p12
## 证书密码
server.ssl.key-store-password=980217
```

此时如果以*HTTP*请求访问就会访问失败，是因为*Spring Boot*不支持同时在配置中启动*HTTP*和*HTTPS*

![](../images/spring boot + vue/开启ssl后使用http访问.png)

配置请求重定向

```java
    @Bean
    public TomcatServletWebServerFactory webServerFactory() {
        TomcatServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint constraint = new SecurityConstraint();
                constraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                constraint.addCollection(collection);
                context.addConstraint(constraint);
            }
        };
        webServerFactory.addAdditionalTomcatConnectors(createTomcatConnector());
        return webServerFactory;
    }

    private Connector createTomcatConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(80);
        connector.setSecure(false);
        connector.setRedirectPort(8080);
        return connector;
    }
```

再次使用*HTTP*访问则可以重定向至*HTTPS*

#### 2.4.2 Jetty配置

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

#### 2.4.3 Undertow配置

````xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
````

### 2.5 Properties配置

application.properties的配置可以放在项目中的四个目录下；

* 项目根目录下的*config*文件夹中    ## 优先级为1
* 项目根目录下                             ## 优先级为2
* *classpath*下的*config*文件夹中     ## 优先级为3
* *classpath*目录下                        ## 优先级为4 

如果这四个位置都有*application.properties*文件，那么会按照优先级查找配置信息，并加载到*Spring Environment*中

如果使用了*application.yml*做为配置文件，那么优先级与其一致

默认情况下按顺序依次查找*application.properties*文件并加载，如果不想使用*application.properties*做为配置文件名，也可以自定义。例如，在*resources*目录下创建一个配置文件*app.properties*然后将项目打成*jar*包，打包成功后，使用如下命令运行；

`java -jar xxx.1.0-SNAPSHOT.jar --spring.config.name=app`

在运行时指定了文件名，使用*spring.config.location*可以指定配置文件所在的目录（注：需要以*/*结束）

`java -jar xxx.1.0-SNAPSHOT.jar --spring.config.name=app -spring.config-location=classpath:/`

### 2.6 类型安全配置属性

无论是*properties*配置还是*YAML*配置，最终都会被加载到*Spring Environment* 中，Spring提供了*@Value*注解以及*EnvironmentAware*接口来将*Spring Environment*中的数据注入到属性上，*Spring Boot*对此进一部提出了类型安全配置属性（Type-safe Configuration Properties) 这样即使在数据量非常庞大的情况下，也可以更加方便地将配置文件中的数据注入到Bean中

```yaml
book:
  name: 三国演义
  author: 罗贯中
  price: 30
```

将这段配置注入到Bean中

```java
@Component
@ConfigurationProperties(prefix = "book")
public class Book {
    private String name;
    private String author;
    private float price;
    // Getter&Setter
}
```

代码解释

* @ConfiguirationProperties中的*prefix*属性描述了要加载的配置文件中的前缀
* 如果配置文件是*YAML*文件，那么可以将数据注入一个集合中
* Spring Boot采用了一种宽松的规则来进行属性绑定，如果Bean中的属性名为authorName，那么配置文件中的属性可以是*bootk.author_name、book.authorName*或者*book.AUTHORNAME*

`使用@ConfigurationProperties注解时，IDEA提示：Spring Boot Configuration Annotation Processor not configured`

![](../images/spring boot + vue/IDEA提示错误1.png)

解决：![](../images/spring boot + vue/IDEA提示错误1-解决.png)

添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

### 2.7 YAML配置

#### 2.7.1 常规配置

*YAML*是*JSON*的超集，简洁而强大。是一种专门用来书写配置文件的语言，可以替代*application.properties*，在创建一个*Spring Boot*项目时，引入的*spring-boot-starter-web*依赖间接的引入了*snakeyaml*依赖，*snakeyaml*会实现对*YAML*配置的解析，利用缩进表示层级关系，并且大小写敏感。

```yaml
server:
  port: 443
  tomcat:
    max-threads: 5
    uri-encoding: UTF-8
    basedir: /home/kay/tmp/
  error:
    path: /error
  servlet:
    context-path: /
    session:
      timeout: 5m
  ssl:
    key-alias: kay
    key-store-password: xxxxx
    key-store: classpath:kay.p12
```

#### 2.7.2 复杂配置

*YAML*不仅可以配置常规属性，也可以配置复杂属性。

```yaml
my:
  address:
    name: DingKay
    address: Nanjing
    favorites:
      - 玩游戏
      - 睡觉
      - 看书
```

注入的实体

```java
@Component
@ConfigurationProperties("person.my")
public class Person {
    private String name;
    private String address;
    private List<String> favorites;
    // Getter&Setter
}
```

注入的结果

![](../images/spring boot + vue/person-my.png)

还支持更复杂的配置，即集合中也可以是一个对象

```yaml
school:
  name:
  students:
    - name: 小一
      age: 12
    - name: 王二
      age: 13
    - name: 丁一
      age: 23
```

注入的实体

```java
@Configuration
@EnableConfigurationProperties(School.class)
@ConfigurationProperties("school")
public class School {
    private String name;
    private List<Student> students;
    // Getter&Setter
}
```

注入的结果

![](../images/spring boot + vue/yaml-school.png)

在*Spring Boot*中使用*YAML*虽然方便，但是*YAML*也有一些缺陷，例如无法使用*@PropertySource*注解加载*YAML*文件，如果项目中有这种需求，还是需要使用*Properties*格式的配置文件。

Tips:当*properties*和*yaml*文件同时存在时，如*application.yml* *application.properties*两个文件同时存在，此时的优先级为：`application.properties > application.yaml`

### 2.8 Profile

频繁的切换开发环境，测试环境以及生产环境，这个时候大量的配置需要更改，例如数据库配置、redis配置、mongodb配置、jms配置等。Spring提供了@Profile注解，Spring Boot则更进一步提供了更加简洁的解决方案，Spring Boot中约定的不同环境下配置文件名称规则为*application-{profile}.properties / yml*，profile占位符表示当前环境的名称，具体配置如下

1. **创建配置文件**

首先在resource目录下创建两个配置文件：*application-dev.yml和*application-prod.yml*，分别表示开发环境中的配置和生产环境中的配置。其中，*application-dev.yml文件的内容如下

```properties
server.port=8080
```

*application-prod.properties*文件的内容如下

```properties
server.port=80
```

2. **配置*application.yml***

```yaml
spring:
  profiles:
    active: dev
```

这个表示使用*application-dev.yml*文件启动项目，若将dev改成prod，则表示使用*application-prod.yml*启动项目，项目启动成功后，就可以通过相应的端口号访问了。

3. **在代码中配置**

对于第二部在*application.yml*中添加的配置，我们也可以在代码中添加配置来完成，在启动类的*main*方法上添加代码，可以替换第二步的配置

```java
SpringApplicationBuilder builder = new SpringApplicationBuilder(Application.class);
        builder.application().setAdditionalProfiles("dev");
        builder.run(args);
```

4. **在项目启动时配置**

对于第二步和第三步提到的两种配置方式，也可以在项目打成*jar*包后启动时，在命令行动态指令当前环境，示例如下

```powershell
java -jar xxxx.1.0-SNAPSHOT.jar --spring.profiles.active=prod
```

对于以上三种方式，若同时开启三种配置*Profile*配置，优先级如下 4 > 2 > 3

启动参数*--spring.profiles.active* 优先级最高，其次是修改*application.yml*文件，然后是在*main*方法中添加代码

## 3.0 Spring Boot 整合视图层

在目前的企业级开发中，前后端分离是趋势，但是视图层技术还占有一席之地，Spring Boot对视图层技术提供了很好的支持，官方推荐使用的模板引擎是*Thymeleaf*，不过像*FreeMarker*也支持，并不推荐使用*JSP*

### 3.1 整合Thymeleaf

*Thymeleaf*是新一代Java模板引擎，支持HTML原型，既可以让前端工程师在浏览器中直接打开查看样式，也可以让后端工程师结合真实数据查看显示效果，同时，Spring Boot提供了*Thymeleaf*自动化配置解决方案，因此在Spring Boot中使用Thymeleaf非常方便，整合步骤如下

1. **创建工程，添加依赖**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
</dependencies>
```

2. **配置Thymeleaf**

Spring Boot为Thymeleaf提供了自动化配置类*ThymeleafAutoConfiguration*，相关的配置属性在*ThymeleafProperties*类中，ThymeleafProperties部分源码如下

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
    // ...
}
```

由此配置可以看到，默认的模板位置在*classpath：/templates/*，默认的模板配置后缀为*.html*

如果想对默认的*Thymeleaf*配置参数进行自定义配置，那么可以直接在*application.properties / yml*中进行配置，部分常见配置如下 

```properties
# 是否开启缓存，开发时可设置为false，默认为true
spring.thymeleaf.cache=false
# 检查模板是否存在，默认为true
spring.thymeleaf.check-template=true
# 检查模板位置是否存在，默认为true
spring.thymeleaf.check-template-loaction=true
# 模板文件编码
spring.thymeleaf.encoding=UTF-8
# 模板文件位置
spring.thymeleaf.prefix=classpath:/templates/
# Content-Type配置
spring.thymeleaf.servlet.content-type=text/html
# 模板文件后缀
spring.thymeleaf.suffix=.html
```

3. **配置控制器**

```java
@Controller
public class BookController {
    @GetMapping("book")
    public ModelAndView book() {
        List<Book> books = new ArrayList<>();
        Book book1 = new Book();
        book1.setAuthor("罗贯中");
        book1.setName("三国演义");
        book1.setPrice(25);

        Book book2 = new Book();
        book2.setAuthor("曹雪芹");
        book2.setName("红楼梦");
        book2.setPrice(32);

        books.add(book1);
        books.add(book2);

        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("books", books);
        modelAndView.setViewName("books");
        return modelAndView;
    }
```

4. **创建视图**

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Book</title>
</head>
<body>
<table border="1">
    <tr>
    <td>图书名称</td>
    <td>图书作者</td>
    <td>图书售价</td>
    </tr>
    <tr th:each="book:${books}">
    <!--/*@thymesVar id="book" type="com.dk.entity.Book"*/-->
    <td th:text="${book.author}"></td>
    <td th:text="${book.name}"></td>
    <td th:text="${book.price}"></td>
    </tr>
</table>
</body>
</html>
```

5. **运行**

![](../images/spring boot + vue/thymeleaf-book.png)

### 3.2 整合FreeMarker

FreeMarker是一个非常古老的模板引擎，可以用在Web环境或者非Web环境中，与Thymeleaf不同，FreeMarker需要经过解析才能够在浏览器中展示出来。FreeMarker不仅可以用来配置HTML页面模板，也可以作为电子邮件模板，配置文件模板以及源码模板等。Spring Boot对FreeMarker整合也提供了很好的支持

1. **创建项目，添加依赖**

创建Spring Boot项目，添加*spring-boot-starter-web*和*spring-boot-starter-freemarker*依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
    </dependency>
</dependencies>
```

2. **配置FreeMarker**

Spring Boot对FreeMarker也提供了自动化配置类*FreeMarkerAutoConfiguration*， 相关的配置在*FreeMarkerProperties*中

```java
@ConfigurationProperties(
    prefix = "spring.freemarker"
)
public class FreeMarkerProperties extends AbstractTemplateViewResolverProperties {
    public static final String DEFAULT_TEMPLATE_LOADER_PATH = "classpath:/templates/";
    public static final String DEFAULT_PREFIX = "";
    public static final String DEFAULT_SUFFIX = ".ftl";
    private Map<String, String> settings = new HashMap();
    private String[] templateLoaderPath = new String[]{"classpath:/templates/"};
    private boolean preferFileSystemAccess = true;
    // ...
}
```

从默认配置中可以看到，FreeMarker默认模板位置和Thymeleaf相同，都在*classpath：/templates/*中，默认文件后缀为*.ftl*，可以在*application.properties*中对这些默认配置进行修改

```yaml
spring:
  freemarker:
    # HttpServletRequest的属性是否可以覆盖Controller中model的同名项
    allow-request-override: false
    # HttpSession的属性值是否可以覆盖Controller中model的同名项
    allow-session-override: false
    # 是否开启缓存
    cache: false
    # 模板文件编码
    charset: utf-8
    # 是否检查模板位置
    check-template-location: true
    # Content-Type的值
    content-type: text/html
    # 是否将HttpServletRequest中的属性添加到model中
    expose-request-attributes: false
    # 是否将HttpSession中的属性添加到model中
    expose-session-attributes: false
    # 模板文件后缀
    suffix: .ftl
    # 模板文件位置
    template-loader-path: classpath:/templates/
```

3. **配置控制器**

与Thymeleaf相同

4. **创建视图**

按照配置，在resources/templates目录下创建books.ftl文件

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
</head>
<title>图书列表</title>
<body>
<table border="1">
    <tr>
        <td>图书名称</td>
        <td>图书作者</td>
        <td>图书售价</td>
    </tr>
    <#--  books不为空 且有数据  -->
    <#if books ??&& (books?size>0)>
    <#--  遍历集合  -->
    <#list books as book>
    <tr>
        <td>${book.name}</td>
        <td>${book.author}</td>
        <td>${book.price}</td>
    </tr>
    </#list>
    </#if>
</table>
</body>
</html>
```

## 4.0 Spring Boot整合Web开发

### 4.1 返回JSON数据

#### 4.1.1 默认实现

JSON是目前主流的前后端数据传输方式，*Spring MVC*中使用消息转换器*HttpMessageConverter*对*JSON*的转换提供了很好的支持，在Spring Boot中更进一步，对相关配置做了进一步的简化。默认情况下，创建一个Spring Boot项目后，添加*spring-boot-starter-web*依赖，这个依赖中默认加入了*jackson-databind*作为*JSON*处理器，此时不需要添额外的*JSON*处理器就能返回一段*JSON*了

```java
public class Book {
    private String name;
    private String author;
    @JsonIgnore
    private Float price;
    @JsonFormat(pattern = "yyyy-MM-dd")
    private Date publicationDate;
    // getter&Setter
}
```

创建BookController

```java
@Controller
public class BookController {
    @GetMapping("book")
    @ResponseBody
    public Book book() {
        Book book = new Book();
        book.setName("三国演义");
        book.setAuthor("罗贯中");
        book.setPrice(32f);
        book.setPublicationDate(new Date());
        return book;
    }
}
```

当然，如果频繁的用到@ResponseBody注解，那么可以用@RestController组合注解代替@Controller和@ResponseBody

此时访问*https://localhost/book*

![](../images/spring boot + vue/book的json返回.png)

这是Spring Boot自带的处理方式，如果采用这种方式，那么对于字段忽略、日期格式化等常见需求都可以用注解来解决

这是通过Spring中默认提供的*MappingJackson2HttpMessageConverter*来实现的，当然这里也可以根据实际需求自定义*JSON*转换器