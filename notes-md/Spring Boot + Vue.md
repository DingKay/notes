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

// 略

使用IntelliJ IDEA 创建Maven工程

// 略

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

![](../images/Spring Boot + Vue/mvn启动springboot.png)

#### 1.3.2 直接运行main方法

直接在IDE中运行App类的main方法；

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

虽然@SpringBootApplication包含了@Configuration注解，但是我们可以创建一个新类来配置Bean，这样便于配置的管理，这个类加上@Configuration注解即可

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
person:
  my:
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

#### 4.1.2 自定义转换器

常见的*JSON*处理器除了*jackson-databind*外，还有*Gson*和*fastjson*

1. 使用*Gson*

*Gson*是Google的一个开源*JSON*解析框架，使用*Gson*首先需要除去默认的*jackson-databind*依赖，然后加入*Gson*依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
    </dependency>
</dependencies>
```

由于*Spring Boot*中默认提供了*Gson*的自动转换类*GsonHttpMessageConvertersConfiguration*，因此*Gson*的依赖添加成功后，可以像使用*jackson-databind*那样直接使用*Gson*，但是在*Gson*进行转换时，如果想对日期类型进行格式化，那么还需要自定义*HttpMessageConverter*，自定义*HttpMessageConverter*可以通过如下方式。

​	首先看*GsonHttpMessageConvertersConfiguration*中的一段源码；

```java
@Bean
@ConditionalOnMissingBean
public GsonHttpMessageConverter gsonHttpMessageConverter(Gson gson) {
    GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
    converter.setGson(gson);
    return converter;
}
```

@ConditionalOnMissingBean注解表示当项目中没有提供*GsonHttpMessageConverter*时才会使用默认的*GsonHttpMessageConverter*，所以只需要提供一个*GsonHttpMessageConverter*即可；

```java
@Configuration
public class GsonConfig {
    @Bean
    GsonHttpMessageConverter gsonHttpMessageConverter() {
        GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
        GsonBuilder gsonBuilder = new GsonBuilder();
        gsonBuilder.setDateFormat("yyyy-MM-dd");
        gsonBuilder.excludeFieldsWithModifiers(Modifier.PROTECTED);
        Gson gson = gsonBuilder.create();
        converter.setGson(gson);
        return converter;
    }
}
```

代码解释:

* @Configuration/@Bean 提供了一个*GsonHttpMessageConverter*的实例
* 设置Gson解析日期的格式
* 设置Gson解析时修饰符为Protected的字段被过滤
* 创建Gson对象放入GsonHttpMessageConverter的实例中并返回converter

此时，将Book类中我们需要隐藏返回的字段*price*的修饰符修改为*Protected*

```java
public class Book {
    private String name;
    private String author;
    protected Float price;
    private Date publicationDate;
    // ...
}
```

2. 使用*fastjson*

fastjson是阿里巴巴开源的JSON解析框架，是目前JSON解析速度最快的开源框架。框架也可以集成到*Spring Boot*中，不同于*Gson*，fastjson集成后并不能立马使用，需要开发者提供相应的*HttpMessageConverter*后才能使用。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.47</version>
    </dependency>
</dependencies>
```

配置fastjson的HttpMessageConverter

```java
@Configuration
public class MyFastJsonConfig {
    @Bean
    FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        FastJsonConfig config = new FastJsonConfig();
        config.setDateFormat("yyyy-MM-dd");
        config.setCharset(StandardCharsets.UTF_8);
        config.setSerializerFeatures(
                SerializerFeature.WriteClassName,
                SerializerFeature.WriteMapNullValue,
                SerializerFeature.PrettyFormat,
                SerializerFeature.WriteNullListAsEmpty,
                SerializerFeature.WriteNullStringAsEmpty
        );
        converter.setFastJsonConfig(config);
        return converter;
    }
}
```

代码解释：

* 自定义MyFastJsonConfig，完成对*FastJsonHttpMessageConverterBean*的提供
* *config.setSerializerFeatures*配置了JSON解析过程的一些细节，例如日期格式、数据编码、是否在生成的JSON中输出类名、是否输出value为null的数据、生成的JSON格式化、空集合输出`[]`而非null、空字符串输出`""`而非null等基础配置

![](../images/spring boot + vue/fastjson中文乱码.png)

现在还需要配置一下响应编码，否则返回的JSON中文会乱码，在*application.properties*中添加如下配置

```yaml
spring:
  http:
    encoding:
      force-response: true
```

![](../images/spring boot + vue/fastjson中文乱码-解决.png)

对于*FastJsonHttpMessageConverter*的配置，除了上面这种方式之外，还有另外一种方式。

在*Spring Boot*项目中，当开发者引入*spring-boot-starter-web*以来后，该依赖又依赖了*spring-boot-autoconfigure*，在这个自动化配置中，有个*WebMvcAutoConfiguration*类提供了对*Spring MVC*最基本的配置，如果某一项自动化配置不满足开发需求，开发者可以针对该项自定义配置，只需实现*WebMvcConfigurer*接口即可（在Spring 5.0 之前是通过继承*WebMvcConfigurerAdapter*类来实现的）

```java
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        FastJsonConfig config = new FastJsonConfig();
        config.setDateFormat("yyyy-MM-dd");
        config.setCharset(StandardCharsets.UTF_8);
        config.setSerializerFeatures(
                SerializerFeature.PrettyFormat,
                SerializerFeature.WriteNullListAsEmpty,
                SerializerFeature.WriteMapNullValue,
                SerializerFeature.WriteNullStringAsEmpty
        );
        converter.setFastJsonConfig(config);
        converters.add(converter);
    }
}
```

代码解释：

* 自定义*MyWebMvcConfig*类并实现*WebMvcConfigurer*接口中的*configureMessageConverters*方法
* 将自定义的*FastJsonHttpConverter*加入*converters*中

`注意：如果使用了Gson。也可以采用这种方式配置，但是不推荐，因为项目中没有GsonHttpMessageConverter时，Spring Boot会自己提供一个GsonHttpMessageConverter，此时重写configureMessageConverters方法，参数converters中已经有GsonHttpMessageConverter的实例了，需要替换已有的GsonHttpMessageConverter实例，操作比较麻烦，所以对于Gson推荐直接提供GsonHttpMessageConverter `

### 4.2 静态资源访问

在*Spring MVC*中，对于静态资源都需要开发者手动配置静态资源过滤，*Spring Boot*中对此提供了自动化配置，简化静态资源过滤配置。

#### 4.2.1 默认策略

*Spring Boot*中对于Spring MVC的自动化配置都在*WebMvcAutoConfiguration*类中，因此对于默认的静态资源过滤策略可以从这个类中一窥究竟。

在*WebMvcAutoConfiguration*类中有一个静态内部类*WebMvcAutoConfigurationAdapter*，实现了4.1节提到了*WebMvcAutoConfigurer*接口，该接口中有一个方法*addResourceHandlers*是用来配置静态资源过滤的。方法在*WebMvcAutoConfigurationAdapter*类中得到了实现，部分核心代码如下：

```java
String staticPathPattern = this.mvcProperties.getStaticPathPattern();
       if (!registry.hasMappingForPattern(staticPathPattern)) {
       this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
}
```

Spring Boot 在这里进行了默认静态资源过滤配置，其中*staticPathPattern*默认定义在*WebMvcProperties*中，定义内容如下：

```java
this.staticPathPattern = "/**";
```

*this.resourceProperties.getStaticLocations()*获取到默认静态资源位置定义在*ResourceProperties*中

```java
this.staticLocations = CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
```

getStaticLocations方法中，对这四个静态资源位置进行了扩充

```java
static String[] getResourceLocations(String[] staticLocations) {
    String[] locations = new String[staticLocations.length + WebMvcAutoConfiguration.SERVLET_LOCATIONS.length];
    System.arraycopy(staticLocations, 0, locations, 0, staticLocations.length);
    System.arraycopy(WebMvcAutoConfiguration.SERVLET_LOCATIONS, 0, locations, staticLocations.length, WebMvcAutoConfiguration.SERVLET_LOCATIONS.length);
    return locations;
}
```

其中SERVLET_LOCATIONS = new String[]{"/"};

可以看到，Spring Boot 默认会过滤所有的静态资源，而静态资源的位置一共有5个

分别是：

* classpath:/META-INF/resources/
* classpath:/resources/
* classpath:/static/
* classpath:/public/
* /

也就是说，可以将静态资源放到这5个位置中的任意一个。注意，按照定义的顺序，5个静态资源位置的优先级依次降低，但是一般情况下，Spring Boot 不需要*webapp*目录，所以第五个`/`可以暂不考虑；

在一个新创建的Spring Boot项目中，添加了*spring-boot-starter-web*依赖之后，在*resources*目录下分别创建四个目录，四个目录中放入同名的静态资源；

![](../images/spring boot + vue/默认静态资源优先级.png)

#### 4.2.2 自定义策略

如果默认的静态资源过滤策略不能满足开发需求，也可以自定义静态资源过滤策略，自定义静态资源过滤策略有以下两种方式

1. 在配置文件中定义

在*application.properties*文件中直接定义过滤规则和静态资源位置

```yaml
spring:
  resources:
    static-locations: classpath:/static/
  mvc:
    static-path-pattern: /static/**
```

过滤规则为`/static/**`，静态资源位置为`classpath:/static/`

2. Java编码定义

实现*WebMvcConfigurer*接口，再重写该接口的*addResourceHandlers*方法

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/pic/**")
            .addResourceLocations("classpath:/photo/");
}
```

### 4.3 文件上传

Spring MVC对文件上传做了简化，在Spring Boot 中对此做了更进一步的简化，文件上传更为方便；

Java中的文件上传一共涉及两个组件，一个是*CommonsMultipartResolver*另一个*StandardServletMultipartResolver*，其中*CommonsMultipartResolver*使用*commons-fileupload*来处理*multipart*请求，而*StandardServletMultipartResolver*则是基于*Servlet 3.0*来处理*multipart*请求的。

因此，若使用*StandardServletMultipartResolver*，则不需要添加额外的*jar*包，*Tomcat 7.0*开始就支持*Servlet 3.0*了，而*Spring Boot 2.0.4* 内嵌的*Tomcat*为*Tomcat 8.5.32*，因此可以直接使用*StandardServletMultipartResolver*

```java
@Bean(
    name = {"multipartResolver"}
)
@ConditionalOnMissingBean({MultipartResolver.class})
public StandardServletMultipartResolver multipartResolver() {
    StandardServletMultipartResolver multipartResolver = new StandardServletMultipartResolver();
    multipartResolver.setResolveLazily(this.multipartProperties.isResolveLazily());
    return multipartResolver;
}
```

根据这里的配置可以看出，如果开发者如果没有提供*MultipartResolver*，那么默认采用的*MultipartResolver*就是*StandardServletMultipartResolver*。因此，在Spring Boot中上传文件甚至可以做到零配置；

#### 4.3.1 单文件上传

首先创建一个Spring Boot项目，并添加*spring-boot-starter-web*依赖

然后在*resources*目录下的*static*目录中创建一个*upload.html*文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>upload</title>
</head>
<body>
    <form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="uploadFile" value="请选择文件">
    <input type="submit" value="上传">
    </form>
</body>
</html>
```

这是一个很简单的文件上传页面，上传接口是*/upload*，注意请求方法是*POST*，enctype是*multipart/from-data*

创建文件上传处理接口

```java
@RestController
public class UploadController {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");

    @PostMapping("/upload")
    public String upload(MultipartFile uploadFile, HttpServletRequest request) {
        String realPath = request.getSession().getServletContext().getRealPath("/uploadFile/");
        String format = sdf.format(new Date());
        File folder = new File(realPath + format);
        if (!folder.exists()) {
            folder.mkdirs();
        }
        String oldName = uploadFile.getOriginalFilename();
        String newName = UUID.randomUUID().toString() +
                Objects.requireNonNull(oldName).substring(oldName.lastIndexOf("."), oldName.length());
        try {
            uploadFile.transferTo(new File(folder, newName));
            return request.getScheme() + "://" + request.getServerName() + ":" +
                    request.getServerPort() + "/uploadFile/" + format + "/" + newName;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "上传失败！";
    }
}
```

上传完毕后，通过返回的路径可以直接访问图片；

因为Spring Boot默认资源路径支持`/`，因此这里的图片可以直接访问到

也可以对图片上传的细节进行配置

```yaml
spring:
  servlet:
    multipart:
      enabled: true
      file-size-threshold: 0
      location: /home/kay/file
      max-file-size: 1MB
      max-request-size: 10MB
      resolve-lazily: false
```

* 是否开启文件上传支持，默认为true
* 文件写入磁盘的阈值，默认为0
* 上传文件的临时保存位置
* 单个文件上传大小限制，默认为1MB
* 多个文件上传的总大小，默认为10MB
* 文件是否延迟解析，默认为false

#### 4.3.2 多文件上传

多文件上传与单文件上传基本一致，首先修改HTML文件

```html
...
<input type="file" name="uploadFiles" value="请选择文件" multiple>
...
```

修改控制器

```java
@PostMapping("/uploads")
public String uploads(MultipartFile[] uploadFiles, HttpServletRequest request) {
    for (MultipartFile uploadFile : uploadFiles) {
    ...
    }
    ...
}
```

### 4.4 @ControllerAdvice

顾名思义，@ControllerAdvice就是@Controller的增强版，@ControllerAdvice主要用来处理全局数据，一般搭配@ExceptionHandler、@ModeAttribute以及@InitBinder使用

#### 4.4.1 全局异常处理

@ControllerAdvice最常见的使用场景就是全局异常处理，在4.3节介绍了文件上传大小的限制，如果用户上传的文件超过了限制大小，就会抛出异常，此时可以通过@ControllerAdvice结合@ExceptionHandler定义全局异常捕获机制

```java
@ControllerAdvice
public class CustomExceptionHandler {
    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public void uploadException(MaxUploadSizeExceededException e, HttpServletResponse response) throws Exception{
        response.setContentType("text/html; charset=UTF-8");
        PrintWriter out = response.getWriter();
        out.write("上传文件大小超出限制");
        out.flush();
        out.close();
    }
}
```

只需在系统中定义*CustomExceptionHandler*类，然后添加到*@ControllerAdvice*注解即可。当系统启动时，该类就会被扫描到Spring容器中，然后定义*uploadException*方法，在该方法上添加了@ExceptionHandler注解，其中定义的*MaxUploadSizeExceededException.class*表明该方法用来处理这个类型的异常，如果想该方法处理所有类型的异常，只需要将*MaxUploadSizeExceededException.class*改成*Exception.class*即可。

方法的参数可以有异常实例、HTTPServletResponse以及HttpServletRequest、Model等，返回值可以是一段JSON、一个ModelAndView、一个逻辑视图名等

如果返回参数是一个*ModelAndView*，假设使用的页面模板为*Thymeleaf*

Thymeleaf配置

```yaml
spring:
  thymeleaf:
    suffix: .html
    check-template-location: true
    prefix: classpath:/templates/
    encoding: UTF-8
    enabled: true
    cache: false
```

CustomExceptionHandler

```java
@ControllerAdvice
public class CustomExceptionHandler {
    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public ModelAndView uploadException(MaxUploadSizeExceededException e, HttpServletResponse response) throws IOException {
        ModelAndView view = new ModelAndView();
        view.setViewName("error");
        view.addObject("msg", "上传文件大小超出限制！");
        return view;
    }
}
```

Thymeleaf模板

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>upload error</title>
</head>
<body>
    <h1 th:text = "${msg}"></h1>
</body>
</html>
```

最后效果

![](../images/spring boot + vue/自定义全局异常处理类返回Thymeleaf视图效果.png)

#### 4.4.2 添加全局数据

@ControllerAdvice是一个全局数据处理组件。因此，也可以在@ControllerAdvice中配置全局数据，使用@ModelAttribute注解进行配置

```java
@ControllerAdvice
public class GlobalConfig {
    @ModelAttribute(value = "info")
    public Map<String, String> userInfo() {
        HashMap<String, String> userInfo = new HashMap<>();
        userInfo.put("userName", "罗贯中");
        userInfo.put("gender", "男");
        return userInfo;
    }
}
```

* 在全局配置中添加userInfo方法，返回一个map，该方法有一个注解@ModelAttribute，其中的value属性表示这条返回数据的key，而方法的返回值是返回数据的value。
* 此时在任意请求的Controller中，通过方法参数中的Model都可以获取info的数据

Controller示例代码如下

```java
@RestController
public class HelloController {

    @GetMapping("hello")
    public ModelAndView hello(Model model) {
        Map<String, Object> map = model.asMap();
        Set<String> keySet = map.keySet();
        Iterator<String> iterator = keySet.iterator();
        while (iterator.hasNext()) {
            String key = iterator.next();
            Object value = map.get(key);
            System.out.println(key + " >>>>> " + value);
        }
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addAllObjects(map);
        modelAndView.setViewName("hello");
        return modelAndView;
    }
}
```

Thymeleaf模板

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>Hello</title>
</head>
<body>
    <h1>hello!</h1>
    <div th:text="${info.userName}"></div>
    <div th:text="${info.gender}"></div>
</body>
</html>
```

#### 4.4.3 请求参数预处理

@ControllerAdvice结合@InitBinder还能实现请求参数预处理，即将表单中的数据绑定到实体类上时进行一些额外处理。

例如有两个实体类Book和Author

```java
@Data
public class Author {
    private String name;
    private int age;
}
@Data
public class Book {
    private String name;
    private String author;
}
```

在Controller上需要接收两个实体类的数据，Controller中的方法定义如下

```java
@RestController
public class BookController {
    @GetMapping("book")
    public ModelAndView book(Book book, Author author) {
        System.out.println(book.toString() + " >>>>> " + author.toString());
        ModelAndView view = new ModelAndView();
        view.addObject(book);
        view.addObject(author);
        view.setViewName("book");
        return view;
    }
}
```

此时参数传递时，两个实体中的`name`属性会混淆，@ControllerAdvice结合@InitBinder可以顺利解决该问题

首先给Controller中方法的参数添加@ModelAttribute注解

```
@RestController
public class BookController {
    @GetMapping("book")
    public ModelAndView book(@ModelAttribute("a") Book book, @ModelAttribute("b") Author author) {
        System.out.println(book.toString() + " >>>>> " + author.toString());
        ModelAndView view = new ModelAndView();
        view.addObject(book);
        view.addObject(author);
        view.setViewName("book");
        return view;
    }
}
```

然后配置@ControllerAdvice

```java
@ControllerAdvice
	// ...
    @InitBinder("a")
    public void init(WebDataBinder binder) {
        binder.setFieldDefaultPrefix("a.");
    }

    @InitBinder("b")
    public void init2(WebDataBinder binder) {
        binder.setFieldDefaultPrefix("b.");
    }
```

* 创建两个方法，第一个 `@InitBinder("a")`表示该方法是处理 `@ModelAttribute("a")`对应的参数的，第二个 `@InitBinder("b")`处理 `@ModelAttribute("b")`对应的参数
* 在每个方法中给相应的 `Field`设置一个前缀，然后在浏览器中请求 book?a.author=罗贯中&a.name=三国演义&b.name=dk&b.age=22 即可成功的区分出`name`属性
* 在*WebDataDinder*对象中，还可以设置允许的字段、禁止的字段、必填的字段以及验证器等。

### 4.5 自定义错误页

上一节中介绍了*Spring Boot*中的全局异常处理，在处理异常时，开发者可以根据实际情况，返回不同的页面，但是这种异常处理方式一般用来处理应用级别的异常。有一些容器级别的错误就处理不了，例如*Filter*中抛出异常，使用@ControllerAdvice定义的全局异常处理机制就无法处理。

因此，Spring Boot中，默认情况下，如果用户在发起请求时发生了404/500错误，Spring Boot会有一个默认的页面展示给用户。

事实上，Spring Boot在返回错误信息时不一定返回HTML页面，而是根据实际情况返回HTML页面或者一段JSON，若开发者发起Ajax请求，则错误信息是一段JSON。对于开发者而言，这一段HTML或者JSON都可以自由定制。

Spring Boot中的错误默认是由*BasicErrorController*类来处理的，该类中的核心方法主要有两个：

```java
    @RequestMapping(
        produces = {"text/html"}
    )
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        HttpStatus status = this.getStatus(request);
        Map<String, Object> model = Collections.unmodifiableMap(this.getErrorAttributes(request, this.isIncludeStackTrace(request, MediaType.TEXT_HTML)));
        response.setStatus(status.value());
        ModelAndView modelAndView = this.resolveErrorView(request, response, status, model);
        return modelAndView != null ? modelAndView : new ModelAndView("error", model);
    }

    @RequestMapping
    @ResponseBody
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        Map<String, Object> body = this.getErrorAttributes(request, this.isIncludeStackTrace(request, MediaType.ALL));
        HttpStatus status = this.getStatus(request);
        return new ResponseEntity(body, status);
    }
```

其中，erroHtml方法用来返回HTML页面，error用来返回错误JSON，具体返回的是HTML还是JSON则要看请求头的Accept参数。返回JSON的逻辑很简单，不必过多介绍，返回HTML的逻辑稍微有点复杂，在errorHtml中，通过调用*resolveErrorView*方法来获取一个错误视图的*ModelAndView*。而*resolveErrorView*方法的调用最终会来到*DefaultErrorViewResovler*类中。

DefaultErrorViewResolver类是Spring Boot中默认的错误信息视图解析器

```java
    // ..
	static {
        Map<Series, String> views = new EnumMap(Series.class);
        views.put(Series.CLIENT_ERROR, "4xx");
        views.put(Series.SERVER_ERROR, "5xx");
        SERIES_VIEWS = Collections.unmodifiableMap(views);
    }
	// ..
    private ModelAndView resolve(String viewName, Map<String, Object> model) {
        String errorViewName = "error/" + viewName;
        TemplateAvailabilityProvider provider = this.templateAvailabilityProviders.getProvider(errorViewName, this.applicationContext);
        return provider != null ? new ModelAndView(errorViewName, model) : this.resolveResource(errorViewName, model);
    }
	// ..
```

从这一段源码中可以看到，Spring Boot默认是在error目录下查找4xx、5xx的文件作为错误视图，当找不到时会回到errorHtml方法中，然后使用error作为默认的错误页面视图名，如果名为error的视图也找不到，用户就会看到默认的错误提示页。

#### 4.5.1 简单配置

通过上述的介绍，自定义错误页其实很简单，提供4xx和5xx页面即可。如果开发者不需要向用户展示详细的错误信息，那么可以把错误信息定义成静态页面，直接在*resources/static*目录下创建error目录，然后在error目录中创建错误展示页面。错误展示页面的命名规则有两种：一种是4xx.html、5xx.html；另一种是直接使用响应码命名文件，例如404.html、405.html、500.html。第二种命名方式划分得更细，当出错时，不同的错误会展示不同的错误页面。

![](../images/spring boot + vue/自定义静态错误页.png)

访问错误的路径，效果如下：

![](../images/spring boot + vue/Custom404.png)

修改Contoller，提供一个会抛异常的请求；

```java
@GetMapping("500")
public String error500() {
    int i = 1 / 0;
    return "error500";
}
```

访问后，会展示500.html的内容；

![](../images/spring boot + vue/Custom500.png)

这种定义都是使用了静态HTML页面，无法向用户展示完整的错误信息，若采用视图模块技术，则可以向用户展示更多的错误信息，如果使用HTML模板，那么先引入模板相关的依赖，这里以Thymeleaf为例，Thymeleaf页面模板默认处于*classpath:/templates/*目录下，因此在该目录下先创建*error*目录，再创建错误展示页

![](../images/spring boot + vue/模板错误页.png)

页面代码如下

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title th:text="${status}"></title>
</head>
<body>
    <div th:text="${timestamp}"></div>
    <div th:text="${status}"></div>
    <div th:text="${error}"></div>
    <div th:text="${message}"></div>
    <div th:text="${path}"></div>
</body>
</html>
```

Spring Boot在这里一共返回了5条错误相关的信息，分别是`timestamp`、`status`、`error`、`message`、`path`。5xx.html页面的内容与4xx.html页面的内容一致。

此时，用户访问一个不存在的地址，4xx.html页面中的内容将被展示出来

![](../images/spring boot + vue/4xx模板页面.png)

访问一个会抛异常的地址

![](../images/spring boot + vue/5xx模板页面.png)

> 若用户定义了多个错误页面，则响应码.html页面的优先级高于4xx.html、5xx.html页面的优先级，即若当前是一个404错误，则优先展示404.html而不是4xx.html；动态页面的优先级高于静态页面，即若resources/templates和resources/static下同时定义了4xx.html，则优先展示resources/templates/4xx.html

#### 4.5.2 复杂配置

上面这种配置还不够灵活，只能定义HTML页面，无法处理JSON的定制，Spring Boot中支持对Error信息的深度定制，接下来将从三个方面介绍深度定制：**自定义Error数据**、**自定义Error视图**以及**完全自定义**

1. 自定义Error数据

自定义Error数据就是对返回的数据进行自定义。Spring Boot返回的Error信息一共有5条，分别是`timestamp`、`status`、`error`、`message`以及`path`。在*BasicErrorController*的errorHtml的方法和error方法中，都是通过*getErrorAttributes*方法获取*Error*信息的。该方法最终会调用到*DefaultErrorAttributes*类的*getErrorAttributes*方法，而*DefaultErrorAttributes*类是在*ErrorMvcAutoConfiguration*中默认提供的。*ErrorMvcAutoConfiguration*类的*errorAttributes*方法源码如下：

```java
@Bean
@ConditionalOnMissingBean(
value = {ErrorAttributes.class},
search = SearchStrategy.CURRENT
)
public DefaultErrorAttributes errorAttributes() {
return new DefaultErrorAttributes(this.serverProperties.getError().isIncludeException());
}
```

从这段源码中可以看出，当系统没有提供ErrorAttributes时才会采用*DefaultErrorAttributes*。

因此，自定义错误提示时，只需要自己提供一个*ErrorAttributes*即可，而*DefaultErrorAttributes*是*ErrorAttributes*的子类，因此只需要继承*DefaultErrorAttributes*即可

```java
@Component
public class MyErrorAttribute extends DefaultErrorAttributes {
    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
        Map<String, Object> errorAttributes = super.getErrorAttributes(webRequest, includeStackTrace);
        errorAttributes.put("customMsg", "出错啦！");
        errorAttributes.remove("error");
        return errorAttributes;
    }
}
```

* 自定义MyErrorAttribute继承自DefaultErrorAttributes，重写DefaultErrorAttributes中的getErrorAttributes方法，MyErrorAttribute添加@Componet注解，该类将被注册到Spring容器中
* 通过super.getErrorAttributes(webRequest, includeStackTrace);获取Spring Boot默认提供的错误信息，然后在此基础上添加Error信息或移除Error信息

此时，当系统抛出异常时，错误信息将被修改，修改4xx.html模板

```html
..
<div th:text="${customMsg}"></div>
..
```

![](../images/spring boot + vue/新增customMsg错误信息字段.png)

使用PostMan查看返回的JSON错误信息

![](../images/spring boot + vue/postman查看错误JSON.png)

2. 自定义Error视图

Error视图是展示给用户的页面，在*BasicErrorController*的errorHtml方法中调用*resolveErrorView*方法获取一个*ModelAndView*实例。resolveErrorView方法是由ErrorViewResolver提供的，通过ErrorMvcAutoConfiguration类的源码可以看到Spring Boot默认采用的ErrorViewResolver是DefaultErrorViewResolver。

```java
@Bean
@ConditionalOnBean({DispatcherServlet.class})
@ConditionalOnMissingBean
public DefaultErrorViewResolver conventionErrorViewResolver() {
return new DefaultErrorViewResolver(this.applicationContext, this.resourceProperties);
}
```

