# Spring Boot + Vue

* 为了使开发者能够快速上手Spring，利用Spring框架快速搭建Java EE项目，Spring Boot应运而生。Spring Boot中对一些常用的第三方库提供了默认的自动化配置方案，使得开发着只需要很少的Spring配置就能运行一个完整的Java EE应用。

**Spring Boot 主要有如下优势**：

提供一个快速的 Spring 项目搭建渠道

开箱即用，很少的 Spring 配置就能运行一个Java EE项目

提供了生产级的服务监控方案

内嵌服务器，可以快速部署

提供了一系列非功能性的通用配置

纯Java配置，没有代码生成，也不需要XML配置

## 1.0 创建Maven工程

### 1.1 使用Maven命令创建

本地（win10）安装Maven，并将`bin`目录添加至`path`环境变量

```powershell
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

*YAML*是*JSON*的超集，简洁而强大。是一种专门用来书写配置文件的语言，可以替代*application.properties*，在创建一个*Spring Boot*项目时，引入的*spring-boot-starter-web*依赖间接的引入了*snakeyaml*依赖，*snakeyaml*会实现对*YAML*配置的解析，利用缩进表示层级关系，并且**大小写敏感**。

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

```java
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

从这段源码可以看出，如果没有定义ErrorViewResolver，那么默认使用的ErrorViewResolver是DefaultErrorViewResolver，正是在DefaultErrorViewResolver中配置了默认去error目录下寻找4xx.html、5xx.html。因此，开发者想要自定义Error视图，只需要提供自己的ErrorViewResolver即可

```java
@Component
public class MyErrorViewResolver implements ErrorViewResolver {
    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
        ModelAndView modelAndView = new ModelAndView("errorPage");
        modelAndView.addAllObjects(model);
        return modelAndView;
    }
}
```

* 自定义MyErrorViewResolver实现ErrorViewResolver接口并实现接口中的resolveErrorView方法，使用@Component注解将该类注册到Spring容器中
* 在resolveErrorView方法中，最后一个Map参数就是Spring Boot提供的默认5条Error信息（可以按照自定义Error数据的步骤对这5条消息进行修改），在resolveErrorView方法中，返回一个ModelAndView，在其中设置Error视图和Error数据
* 理论上，开发者也可以通过实现ErrorViewResolver接口来实现Error数据的自定义，但是如果只是单纯的想自定义Error数据，还是建议继承DefaultErrorAttribute

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>ErrorPage</title>
</head>
<body>
    <h1 th:text="${customMsg}"></h1>
    <h1 th:text="${timestamp}"></h1>
    <h1 th:text="${status}"></h1>
    <h1 th:text="${error}"></h1>
    <h1 th:text="${message}"></h1>
    <h1 th:text="${path}"></h1>
</body>
</html>
```

在errorPage中，除了展示Spring Boot提供的5条Error信息外，也展示了自定义Error信息，此时，无论请求发生4xx、5xx错误，都会来到errorPage页面

3. 完全自定义

前面提到的两种自定义方式都是对BasicErrorController类中的某个环节进行修补。查看Error自动化配置类*ErrorMvcAutoConfiguration*，可以发现BasicErrorController本身只是一个默认的配置

```java
// ...
@Bean
@ConditionalOnMissingBean(
value = {ErrorController.class},
search = SearchStrategy.CURRENT
)
public BasicErrorController basicErrorController(ErrorAttributes errorAttributes) {
return new BasicErrorController(errorAttributes, this.serverProperties.getError(), this.errorViewResolvers);
}
// ...
```

如果需要更加灵活的对Error视图和数据进行处理，提供自己的ErrorController即可。提供自己的ErrorController有两种方式：

一种是实现ErrorController接口，另一种是直接继承BasicErrorController。由于ErrorController接口只提供待实现的方法，而BasicErrorController已经实现了很多功能，因此这里选择第二种方法；

```java
@Controller
public class MyErrorController extends BasicErrorController {

    @Autowired
    public MyErrorController(ErrorAttributes errorAttributes, ServerProperties serverProperties, List<ErrorViewResolver>
            errorViewResolvers) {
        super(errorAttributes, serverProperties.getError(), errorViewResolvers);
    }

    @Override
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        Map<String, Object> errorAttributes = getErrorAttributes(request, isIncludeStackTrace(request, MediaType.ALL));
        errorAttributes.put("error", "出错啦！");
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(errorAttributes, status);
    }

    @Override
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        HttpStatus status = getStatus(request);
        Map<String, Object> errorAttributes = getErrorAttributes(request, isIncludeStackTrace(request, MediaType.ALL));
        errorAttributes.put("customMsg", "出错啦！");
        ModelAndView modelAndView = new ModelAndView("myErrorPage", errorAttributes, status);
        return modelAndView;
    }
}
```

* z自定义MyErrorController继承自BasicErrorController并添加@Controller注解，将MyErrorController注册到Spring MVC容器中
* 由于BasicErrorController没有无参构造方法，因此在创建BasicErrorController实例时需要传递参数，在MyErrorController的构造方法上添加@AutoWired注解注入所需参数
* 参考BasicErrorController中的实现，重写error和errorHtml方法，对error的视图和数据进行充分的自定义

### 4.6 CORS支持

CORS（Cross-Origin Resource Sharing）是由W3C制定的一种跨域资源共享技术标准，其目的就是为了解决前端的跨域请求。在Java开发中，最常见的前端跨域请求解决方案是JSONP，但是JSONP只支持GET请求，这是一个很大的缺陷，而CORS则支持多种HTTP请求方法，以CORS中的*GET*请求为例，当浏览器发起请求时，请求头中携带了如下信息

```
...
HOST:localhost:8080
Origin:http://localhost:8081
Referer:http://localhost:8081/index.html
...
```

假如服务端支持CORS，则服务端给出的响应信息如下

```
...
Access-Control-Allow-Origin:http://localhost:8081
Content-Length:20
Content-Type:text/plain;charset=UTF-8
...
```

> 注意：响应头中有一个Access-Control-Allow-Origin字段，用来记录可以访问该资源的域。当浏览器收到这样的响应头信息后，提取出Access-Control-Allow-Origin字段中的值，发现该值包含当前页面所在的域，就知道这个跨域是被允许的，因此就不再对前端的跨域请求进行限制，这就是GET请求的整个跨域流程，在这个过程中，前端请求的代码不需要修改，主要是后端进行处理，这个流程主要是针对GET、POST以及HEAD请求，并且没有自定义请求头，如果用户发起一个DELETE请求、PUT请求或者自定义了请求头，流程就会稍微发杂一些。

以DELETE请求为例，当前端发起一个DELETE请求时，这个请求的处理会经过两个步骤。

第一步，发送一个OPTIONS请求

```
...
Access-Control-Request-Method DELETE
Connection keep-alive
Host localhost:8080
Origin http://localhost:8081
...
```

这个请求将向服务端询问是否具备该资源的DELETE权限，服务端会给浏览器一个响应

```
...
HTTP/1.1 200
Access-Control-Allow-Origin: http://localhost:8081
Access-Control-Allow-Methods: DELETE
Access-Control-Max-Age: 1800
Allow: GET, HEAD, POST, PUT, DELETE, OPTIONS, PATCH
Content-Length: 0
...
```

至此一个跨域的DELETE请求完成。

无论是简单请求，还是需要先进行探测的请求，前端的写法都是不变的，额外的处理都是在服务端来完成的。在传统的Java EE开发中，可以通过过滤器统一配置，而Spring Boot中对此则提供了更加简洁的解决方案。

1. 创建Spring Boot工程，添加依赖
2. 创建Controller

```java
@PostMapping("book")
public String addBook(String name) {
return "receive:" + name;
}

@DeleteMapping("/{id}")
public String deleteBookById(@PathVariable Long id) {
return String.valueOf(id);
}
```

3. 配置跨域

跨域有两个地方跨域配置，一个是直接在相应的请求方法上加注解：

```java
@PostMapping("book")
@CrossOrigin(value = "http://localhost:8081", maxAge = 1800, allowedHeaders = "*")
public String addBook(String name) {
return "receive:" + name;
}

@DeleteMapping("/{id}")
@CrossOrigin(value = "http://localhost:8081", maxAge = 1800, allowedHeaders = "*")
public String deleteBookById(@PathVariable Long id) {
return String.valueOf(id);
}
```

代码解释：

* @CrossOrigin中的value表示支持的域，这里表示支持localhost:8081域的请求是支持跨域的
* maxAge表示探测请求的有效期，DELETE、PUT请求或者自定义头请求，在执行过程中会先发送探测请求，探测请求不用每次都发送，可以配置一个有效期，有效期过了之后才会发送探测请求，这个属性默认是1800秒，即30分钟。
* allowedHeaders表示允许的请求头，*表示允许所有的请求头

这种配置方式是一种细粒度的配置，可以控制到每一个方法上，当然，也可以不在每个方法上面加@CrossOrigin注解，而是采用一种全局配置

```java
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/*")
                .allowedHeaders("*")
                .allowedMethods("*")
                .maxAge(1800)
                .allowedOrigins("http://localhost:8081");
    }
}

```

代码解释：

* 全局配置需要自定义类实现WebMvcConfigurer接口，然后实现接口中的 addCorsMappings 方法。
* 在 addCorsMappings 方法中，addMapping表示对哪种格式的请求路径进行跨域处理；allowedHeaders表示允许的请求头，默认允许所有的请求头信息；allowedMethods表示允许的请求方法，默认是GET、POST和HEAD；*表示支持所有的请求方法；maxAge表示探测请求有效期；allowedOrigins表示支持的域；

以上两种配置任选其一即可。

4. 测试

新建一个项目，创建一个index.html文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/jquery.min.js"></script>
</head>
<body>
    <div id="contentDiv"></div>
    <div id="deleteResult"></div>
    <input type="button" value="提交数据" onclick="getData()"/> <br/>
    <input type="button" value="删除数据" onclick="deleteData()"/> <br/>
    <script>
        function deleteData() {
            $.ajax({
                url: 'http://localhost:8080/99',
                type: 'delete',
                success:function (msg) {
                    $("#deleteResult").html(msg);
                }
            })
        }
        function getData() {
            $.ajax({
                url: 'http://localhost:8080/book/',
                type: 'post',
                data: {name:'三国演义'},
                success:function (msg) {
                    $("#contentDiv").html(msg);
                }
            })
        }
    </script>
</body>
</html>
```

两个普通的Ajax都发送了一个跨域请求

将项目的端口修改为8081

```properties
server.port=8081
```

启动项目，点击两个按钮

![](../images/spring boot + vue/页面按钮发送请求cors.png)

### 4.7 配置类与XML配置

Spring Boot推荐使用Java来完成相关的配置工作，在项目中，不建议将所有的配置都放在一个配置类中，可以根据不同的需求提供不同的配置类，例如专门处理Spring Security的配置类、提供Bean的配置类、Spring MVC的相关配置类。这些配置类上都需要添加@Configuration注解，@ComponentScan注解会扫描所有的Spring组件，也包括@Configuration。

Spring Boot中不推荐使用XML配置，尽量使用Java配置代替XML配置，如果需要使用XML配置，只需要在resources目录下提供XML配置文件，然后通过@ImportResource加载配置文件，例如有个Hello类如下

```java
public class Hello {
    public String sayHello(String name) {
        return "hello " + name;
    }
}

```

添加一个beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <beans>
        <bean class="com.dk.bean.Hello" id="hello"></bean>
    </beans>
</beans>
```

创建配置类BeansConfig

```java
@Configuration
@ImportResource("classpath:beans.xml")
public class BeansConfig {
}

```

在Controller中使用

```java
@Autowired
Hello hello;
@GetMapping("bean")
public String bean() {
return hello.sayHello("xdk");
}
```

### 4.8 注册拦截器

Spring MVC中提供了AOP风格的拦截器，拥有更加精细的拦截处理能力，Spring Boot中拦截器的注册更加方便

1. 创建Spring Boot项目，添加web依赖；
2. 创建自定义拦截器实现HandlerInterceptor接口

```java
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor >> preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor >> postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor >> afterCompletion");
    }
}
```

拦截器中的方法将按照 preHandle -> Controller -> postHandle -> afterCompletion的顺序执行。注意，只有preHandle方法返回true时后面的方法才会执行。当拦截器链内存在多个拦截器时，postHandle在拦截器链内的所有拦截器返回成功时才会调用，而afterCompletion只有preHandle返回true才调用，但若拦截器链内的第一个拦截器的preHandle方法返回false，则后面的方法都不会执行。

3. 配置拦截器，定义配置类进行拦截器的配置

```java
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/hello");
    }
}
```

自定义实现WebMvcConfigurer接口，实现接口中的 addInterceptor方法，其中，addPathPatterns表示拦截器路径，excludePathPatterns表示排除的路径

4. 测试，浏览器中访问Controller，打印日志；

```
MyInterceptor >> preHandle
MyInterceptor >> postHandle
MyInterceptor >> afterCompletion
```

### 4.9 启动系统任务

有一些特殊的任务需要在系统启动时执行，例如配置文件加载，数据库初始化等操作，如果没有使用Spring Boot，这些问题可以在Listener中解决。Spring Boot对此提供了两种解决方案： **CommandLineRunner**和 **ApplicationRunner**。两者差异主要体现在参数上不同

#### 4.9.1 CommandLineRunner

Spring Boot项目启动时会遍历所有 **CommandLineRunner**的实现类，并调用其中的run方法。如果整个系统中有多个实现类，那么可以用@Order注解对这些实现类的调用顺序进行排序。

在一个Spring Boot Web项目中添加两个 **CommandLineRunner**

```java
@Component
@Order(1)
public static class CommandLineRunnerOne implements CommandLineRunner {
@Override
public void run(String... args) throws Exception {
System.out.println("RunnerOne >> " + Arrays.toString(args));
}
}

@Component
@Order(2)
public static class CommandLineRunnerTwo implements CommandLineRunner {
@Override
public void run(String... args) throws Exception {
System.out.println("RunnerTwo >> " + Arrays.toString(args));
}
}
```

代码解释：

* @Order注解用来描述CommandLineRunner的执行顺序，数字越小越先执行。
* run方法中是调用的核心逻辑，参数是系统启动时传入的参数，即入口类中main方法的参数（在调用SpringApplication.run方法时被传入Spring Boot项目中）

```
RunnerOne >> main[小丁, 丁一]
RunnerTwo >> main[小丁, 丁一]
```

#### 4.9.2 ApplicationRunner

ApplicationRunner的用法和CommandLineRunner基本一致，区别主要体现在run方法的参数上

```java
@Component
@Order(1)
public static class ApplicationRunnerOne implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        List<String> nonOptionArgs = args.getNonOptionArgs();
        System.out.println("ApplicationRunnerOne-nonOptionArgs = " + nonOptionArgs);
        Set<String> optionNames = args.getOptionNames();
        System.out.println("ApplicationRunnerOne-optionNames = " + optionNames);
        for (String optionName : optionNames) {
            System.out.println("ApplicationRunnerOne-key: " + optionName + "; value: " + args.getOptionValues(optionName));
        }
    }
}

@Component
@Order(2)
public static class ApplicationRunnerTwo implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        List<String> nonOptionArgs = args.getNonOptionArgs();
        System.out.println("ApplicationRunnerTwo-nonOptionArgs = " + nonOptionArgs);
        Set<String> optionNames = args.getOptionNames();
        System.out.println("ApplicationRunnerTwo-optionNames = " + optionNames);
        for (String optionName : optionNames) {
            System.out.println("ApplicationRunnerTwo-key: " + optionName + "; value: " + args.getOptionValues(optionName));
        }
    }
}
```

代码解释：

* @Order注解用来描述CommandLineRunner的执行顺序，数字越小越先执行。
* 不同于CommandLineRunner中run方法的String数组参数，这里run方法的参数是ApplicationArguments对象，如果想从ApplicationArguments对象中获取入口类中main方法接受的参数，调用getNonOptionArgs即可。ApplicationArguments中的getOptionNames方法用来获取项目启动命令行中参数的key，例如本项目打成jar包，运行 java -jar xxx.jar -name=Michael 命令来启动项目，此时getOptionNames方法获取到的就是name，而getOptionValues则是获取相应的value。

```
--name=Michael --age=99 小丁 丁一
```

命令解释：

* --name=Michael --age=99都属于getOptionNames/getOptionValues范畴。
* 后面的 **小丁**、**丁一**可以通过getNonOptionArgs方法获取，获取到的是一个数组，相当于上文提到的运行时配置的ProgramArgument

output：

```
ApplicationRunnerOne-nonOptionArgs = [小丁, 丁一]
ApplicationRunnerOne-optionNames = [name, age]
ApplicationRunnerOne-key: name; value: [Michael]
ApplicationRunnerOne-key: age; value: [99]
ApplicationRunnerTwo-nonOptionArgs = [小丁, 丁一]
ApplicationRunnerTwo-optionNames = [name, age]
ApplicationRunnerTwo-key: name; value: [Michael]
ApplicationRunnerTwo-key: age; value: [99]
```

### 4.10 整合Servlet、Filter和Listener

一般情况下，使用Spring、Spring MVC这些框架后，基本上就告别了Servlet、Filter以及Listener了，但是有时在整合一些第三方框架时，可能还是不得不使用Servlet，比如在整合某报表插件时，就需要使用Servlet。Spring Boot中对于整合这些基本的Web组件也提供了很好的支持。

在项目中添加如下三个组件：

```java
@WebFilter("/*")
public static class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("MyFilter >> init");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("MyFilter >> doFilter");
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
        System.out.println("MyFilter >> destroy");
    }
}

@WebListener
public static class MyListener implements ServletRequestListener {
    @Override
    public void requestDestroyed(ServletRequestEvent servletRequestEvent) {
        System.out.println("MyListener >> requestDestroyed");
    }

    @Override
    public void requestInitialized(ServletRequestEvent servletRequestEvent) {
        System.out.println("MyListener >> requestInitialized");
    }
}

@WebServlet("/my")
public static class MyServlet extends HttpServlet {
    private static final long serialVersionUID = 6477475829358344516L;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("req.getParameter(\"name\") = " + req.getParameter("name"));
    }
}
```

代码解释：

* 这里定义了三个基本组件，分别使用@WebServlet、@WebFilter 和@WebListener三个注解进行标记
* 这里以ServletRequestListener为例，但是对于其他的Listener，例如HttpSessionListener、ServletContextListener等也是支持的。

在项目入口类上添加@ServletComponentScan注解，实现对Servlet、Listener以及Filter的扫描

最后启动项目，查看日志

```
MyFilter >> init
...
...
MyListener >> requestInitialized
MyFilter >> doFilter
req.getParameter("name") = xdk
MyListener >> requestDestroyed
```

### 4.11 路径映射

一般情况下，使用了页面模板后，用户需要访问控制器才能访问页面，有一些页面需要在控制器中加载数据，然后渲染，才能显示出来；还有一些页面在控制器中不需要加载数据，只是完成简单的跳转，对于这类页面，配置路径映射，提高访问速度。

例如有两个Thymeleaf模板页面login.html和index.html，直接在MVC配置中重写addViewControllers方法配置映射关系即可

```java
@Override
public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/login").setViewName("login");
    registry.addViewController("/index").setViewName("index");
}
```

### 4.12 配置AOP

#### 4.12.1 AOP简介

要介绍面向切面编程（Aspect-Oriented Programming，AOP），需要考虑这样一个场景：公司有一个管理系统已经上线，但是运行不稳定，有时快有时慢，为了弄清哪个环节出了问题，开发人员想要监控每个方法的执行时间，再根据这些执行时间判断问题所在，当问题解决后，再把这些监控移除掉。系统目前已经运行，如果手动修改系统成千上万个方法，工作量太大，而且这套监控方法最后还是要移除掉。如果能够在系统中动态添加代码，就能很好的解决这个需求。在系统运行时动态添加代码的方式称为面向切面编程（AOP），在AOP中有一些常见的概念

* Joinpoint（连接点）：类里面可以被增强的方法即为连接点。例如，想修改哪个方法的功能，那么该方法就是一个连接点。
* Pointcut（切入点）：对Joinpoint进行拦截的定义即为切入点。例如，拦截所有以insert开始的方法，这个定义即为切入点。
* Advice（通知）：拦截到Joinpoint之后所要做的事情就是通知。例如，上文说到的监控，通知分为前置通知、后置通知、环绕通知、异常通知和最终通知。
* Aspect（切面）：Pointcut和Advice的结合。
* Target（目标对象）：要增强的类称为Target。

这些是对AOP的简单介绍

#### 4.12.2 Spring Boot支持

Spring Boot在Spring的基础上对AOP的配置提供了自动化配置解决方案 spring-boot-starter-aop，使开发者能够更加便捷的在Spring Boot项目中使用AOP。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

在 `com.dk.aop.service`包下创建UserService

```java
@Service
public class UserService {
    public String getUserById(Integer id) {
        System.out.println("getUserById");
        return "user";
    }

    public void deleteUserById(Integer id) {
        System.out.println("delete...");
    }
}
```

接下来创建切面

```java
@Component
@Aspect
public class LogAspect {
    @Pointcut("execution(* com.dk.aop.service.*.*(..))")
    public void pc() {}

    @Before(value = "pc()")
    public void before(JoinPoint jp) {
        String name = jp.getSignature().getName();
        System.out.println(name + "方法开始执行");
    }

    @After(value = "pc()")
    public void after(JoinPoint jp) {
        String name = jp.getSignature().getName();
        System.out.println(name + "方法执行结束");
    }

    @AfterReturning(value = "pc()", returning = "result")
    public void afterReturning(JoinPoint jp, Object result) {
        String name = jp.getSignature().getName();
        System.out.println(name + "方法返回值为：" + result);
    }

    @AfterThrowing(value = "pc()", throwing = "e")
    public void afterThrowing(JoinPoint jp, Exception e) {
        String name = jp.getSignature().getName();
        System.out.println(name + "方法抛异常了，异常是：" + e.getMessage());
    }
    
    @Around(value = "pc()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        return pjp.proceed();
    }
}
```

代码解释：

* @Aspect注解表示这是一个切面类
* 定义pc方法，使用了@Pointcut注解，这是一个切入点定义，execution中的第一个`*`表示任意返回值，第二个`*`表示service包下的任意类，第三个`*`表示类中的任意方法，括号中的两个`..`表示方法参数任意，即这里描述的切入点为service包下所有中的所有方法。
* @Before注解表示这是一个前置通知，该方法在目标方法执之前执行。通过JoinPoint参数可以获取目标方法的方法名，修饰符等信息。
* @After注解表示这是一个后置通知，在目标方法执行之后执行。
* @AfterReturning注解表示这是一个返回通知，该方法获取目标方法的返回值，@AfterReturning注解的returning参数是指返回值的变量名，对应方法的参数。注意，在方法参数中定义了result的类型为Object，表示目标方法的返回值可以是任意类型，若result参数的类型为Long，则该方法只能处理目标方法返回值为Long的情况
* @AfterThrowing注解表示这是一个异常通知，即当目标方法发生异常时，该方法会被调用，异常类型为Exception表示所有的异常都会进入该方法中执行，若异常类型为ArithmeticException，则表示只有目标方法抛出的ArithmeticException才会进入该方法中处理。
* @Around注解表示这是一个环绕通知，环绕通知时所有通知里功能最强大的通知，可以实现前置通知、后置通知、异常通知、以及返回值通知的功能。目标方法进入环绕通知后，通过调用ProceedingJoinPoint对象的proceed方法使目标方法继续执行，开发者可以在此修改目标方法的执行参数、返回值等。并且可以在此处理目标方法的异常。

配置完成后，接下来在Controller中创建接口，分别调用UserService中的两个方法，即可看到LogAspect中的代码动态的嵌入到目标方法中执行了

```java
@RestController
public class UserController {
    @Autowired
    UserService userService;

    @GetMapping("/getUser")
    public String getUser(Integer id) {
        return userService.getUserById(id);
    }

    @GetMapping("/deleteUser")
    public void deleteUser(Integer id) {
        userService.deleteUserById(id);
    }
}
```

### 4.13 其他

#### 4.13.1 自定义欢迎页

Spring Boot项目在启动后，首先会去静态资源路径下查找index.html作为首页文件，若查找不到，则会去查找动态的index文件作为首页。

例如，如果想使用静态的index.html页面作为项目首页，那么只需要在**resources/static/**路径下创建index.html文件即可。若想使用动态页面作为首页，则需要在 **resources/templates/**目录下创建index.html（Thymeleaf模板）或者index.ftl（FreeMarker），然后在Controller中返回逻辑视图名

```java
@Controller
public class IndexController {
    @GetMapping("/")
    public String index() {
        return "index";
    }
}
```

> 自定义错误页静态/模板页面条件相同时，优先展示模板页面；首页欢迎页静态/模板页面相同条件时，优先展示静态页面

#### 4.13.2 自定义favicon

favicon.ico是浏览器选项卡左上角的图标，可以放在静态资源路径下或者类路径下，静态资源路径下的favicon.ico优先级高于类路径下的favicon.ico

将一张.ico图片重命名为favicon，复制到 **resources/static/**路径下

![](../images/spring boot + vue/favicon.png)

启动项目，查看效果

![](../images/spring boot + vue/替换favicon效果.png)

#### 4.13.3 除去某个自动配置

Spring Boot中提供了大量的自动化配置，例如上文提到的ErrorMvcAutoConfiguration、ThymeleafAutoConfiguration、FreeMarkerAutoConfiguration、MultipartAutoConfiguration等，这些自动化配置可以减少相应的操作配置，达到开箱即用的效果，在Spring Boot的入口类上有一个@SpringBootApplication注解，该注解是一个组合注解，由@SpringBootConfiguration、@EnableAutoConfiguration以及@ComponentScan组成，其中@Enable Auto Configuration注解开启自动化配置，相关的自动化配置类就会被使用，如果开发者不想使用某个自动化配置，按如下方式排除去自动化配置即可。

```java
@SpringBootApplication(exclude = {ErrorMvcAutoConfiguration.class, GsonAutoConfiguration.class})
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

此时访问出错不会自动跳转了，由于exclude属性值是一个数组，因此有多个要排除的自动化配置类时只需要继续添加即可。除了这种配置外，也可以在application.properties配置文件中配置

```properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration
```

## 5.0 Spring Boot整合持久层技术

持久层是JavaEE中访问数据库的核心操作，Spring Boot中对常见的持久层框架都提供了自动化配置，例如JdbcTemplate、JPA等，MyBatis的自动化配置则是官方提供的。

### 5.1 整合JdbcTemplate

JdbtTemplate是Spring提供的一套JDBC模板框架，利用AOP技术来解决直接使用JDBC时大量重复代码的问题。JdbcTemplate虽然没有MyBatis那么灵活，但是比直接使用JDBC要方便很多。Spring Boot中对JdbcTemplate的使用提供了自动化配置类JdbcTemplateAutoConfiguration

```java
@Configuration
@ConditionalOnClass({DataSource.class, JdbcTemplate.class})
@ConditionalOnSingleCandidate(DataSource.class)
@AutoConfigureAfter({DataSourceAutoConfiguration.class})
@EnableConfigurationProperties({JdbcProperties.class})
public class JdbcTemplateAutoConfiguration {
    public JdbcTemplateAutoConfiguration() {
    }

    @Configuration
    @Import({JdbcTemplateAutoConfiguration.JdbcTemplateConfiguration.class})
    static class NamedParameterJdbcTemplateConfiguration {
        NamedParameterJdbcTemplateConfiguration() {
        }

        @Bean
        @Primary
        @ConditionalOnSingleCandidate(JdbcTemplate.class)
        @ConditionalOnMissingBean({NamedParameterJdbcOperations.class})
        public NamedParameterJdbcTemplate namedParameterJdbcTemplate(JdbcTemplate jdbcTemplate) {
            return new NamedParameterJdbcTemplate(jdbcTemplate);
        }
    }

    @Configuration
    static class JdbcTemplateConfiguration {
        private final DataSource dataSource;
        private final JdbcProperties properties;

        JdbcTemplateConfiguration(DataSource dataSource, JdbcProperties properties) {
            this.dataSource = dataSource;
            this.properties = properties;
        }

        @Bean
        @Primary
        @ConditionalOnMissingBean({JdbcOperations.class})
        public JdbcTemplate jdbcTemplate() {
            JdbcTemplate jdbcTemplate = new JdbcTemplate(this.dataSource);
            Template template = this.properties.getTemplate();
            jdbcTemplate.setFetchSize(template.getFetchSize());
            jdbcTemplate.setMaxRows(template.getMaxRows());
            if (template.getQueryTimeout() != null) {
                jdbcTemplate.setQueryTimeout((int)template.getQueryTimeout().getSeconds());
            }

            return jdbcTemplate;
        }
    }
}
```

从上面这段源码中可以看出，当classpath下存在DataSource和JdbcTemplate并且DataSource只有一个实例时，自动配置才会生效，若开发者没有提供JdbcOperations，则Spring Boot会自动向容器注入一个JdbcTemplate（JdbcTemplate时JdbcOperations的子类）。

1. 创建数据库和表

在数据库中创建表

```sql
-- 创建库
CREATE DATABASE `CHAPTER` DEFAULT CHARACTER SET utf8;
-- 切换当前数据库为 CHATPER
use `CHAPTER`;
-- 创建表
CREATE TABLE `book` ( `id` int(11) NOT NULL AUTO_INCREMENT, `name` varchar(50) DEFAULT NULL, `author` varchar(20)DEFAULT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB DEFAULT CHARSET=utf8;
-- 插入数据
INSERT INTO `book` (`id`, `name`, `author`) VALUES (1, 'SanGuoYanYi', 'LuoGuanZhong'),(2, 'ShuiHuZhuan', 'ShiNaiAn');
```

2. 创建项目，添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.9</version>
    </dependency>
</dependencies>
```

spring-boot-starter-jdbc中提供了spring-jdbc，另外还加入了数据库驱动依赖和数据库连接池依赖。

3. 数据库配置

   在application.properties中配置数据库基本连接信息：

   ```yaml
   spring:
     datasource:
       type: com.alibaba.druid.pool.DruidDataSource
       url: jdbc:mysql:///CHAPTER
       username: root
       password: 123456
   ```

4. 创建实体类

```java
@Data
public class Book {
    private Integer id;
    private String name;
    private String author;
}
```

5. 创建数据库访问层

```java
@Repository
public class BookDao {
    @Autowired
    JdbcTemplate jdbcTemplate;

    public int addBook(Book book) {
        return jdbcTemplate.update("INSERT INTO book (name, author) values (?, ?)", book.getName(), book.getAuthor());
    }
    
    public int updateBook(Book book) {
        return jdbcTemplate.update("UPDATE book SET name = ? , author = ? WHERE id = ?", book.getName(),
                book.getAuthor(), book.getId());
    }
    
    public int deleteBookById(Integer id) {
        return jdbcTemplate.update("DELETE FROM book where id = ?", id);
    }
    
    public Book getBookById(Integer id) {
        return jdbcTemplate.queryForObject("SELECT * FROM book WHERE id = ?", new BeanPropertyRowMapper<>(Book.class), id);
    }
    
    public List<Book> getAllBooks() {
        return jdbcTemplate.query("SELECT * FROM book", new BeanPropertyRowMapper<>(Book.class));
    }
}
```

代码解释：

* 创建BookDao，注入JdbcTemplate。由于已经添加了Spring-jdbc相关的依赖，JdbcTemplate会被自动注入到Spring容器中，因此这里可以直接注入JdbcTemplate使用
* 在JdbcTemplate中，增删改三种类型的操作主要使用update和batchUpdate方法来完成，query和queryForObject方法主要用来完成查询功能，另外，execute方法可以用来执行任意SQL、call方法用来调用存储过程等。
* 在执行查询操作时，需要有一个RowMapper将查询出来的列和实体类中的属性一一对应，如果列名和属性名都是相同的，那么可以直接使用BeanPropertyRowMapper；如果列名和属性名不同，就需要开发者自己实现RowMapper接口，将列和实体类属性一一对应起来。

6. 创建Service和Controller

创建BookService和BookController

```java
@Service
public class BookServiceImpl implements BookService {
    @Autowired
    BookDao bookDao;

    @Override
    public int addBook(Book book) {
        return bookDao.addBook(book);
    }

    @Override
    public int updateBook(Book book) {
        return bookDao.updateBook(book);
    }

    @Override
    public int deleteBookById(Integer id) {
        return bookDao.deleteBookById(id);
    }

    @Override
    public Book getBookById(Integer id) {
        return bookDao.getBookById(id);
    }

    @Override
    public List<Book> getAllBooks() {
        return bookDao.getAllBooks();
    }
}
@RestController
public class BookController {
    @Autowired
    BookService bookService;

    @GetMapping("/bookOps")
    public void bookOps() {
        Book b1 = new Book();
        b1.setName("西厢记");
        b1.setAuthor("王实甫");
        int addBook = bookService.addBook(b1);
        System.out.println("addBook = " + addBook);
        Book b2 = new Book();
        b2.setId(1);
        b2.setName("朝花夕拾");
        b2.setAuthor("鲁迅");
        int updateBook = bookService.updateBook(b2);
        System.out.println("updateBook = " + updateBook);
        Book getBookById = bookService.getBookById(1);
        System.out.println("getBookById = " + getBookById);
        int deleteBookById = bookService.deleteBookById(2);
        System.out.println("deleteBookById = " + deleteBookById);
        List<Book> allBooks = bookService.getAllBooks();
        System.out.println("allBooks = " + allBooks);
    }
}
```

### 5.2 整合MyBatis

MyBatis是一款优秀的持久层框架，原名iBatis，2010年由ApacheSoftWareFoundation迁移到Google Code并改名为MyBatis，2013年又迁移到GitHub上。MyBatis支持定制化SQL、存储过程以及高级映射，MyBatis几乎避免了所有JDBC代码手动设置参数以及获取结果集。在传统的SSM框架整合中，使用MyBatis需要大量的XML配置，而在Spring Boot中提供了一套自动化配置方案。

1. 创建项目

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.9</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

2. 创建数据库、表、实体类等

3. 创建数据库访问层

```java
@Mapper
public interface BookMapper {
    int addBook(Book book);
    int deleteBookById(Integer id);
    int updateBookById(Book book);
    Book getBookById(Integer id);
    List<Book> getAllBooks();
}
```

代码解释：

* 在项目的根包下面创建一个子包mapper，在mapper创建BookMapper
* 有两种方式指明该类是一个Mapper；第一种如前面代码中所示，在BookMapper上添加一个@Mapper注解，表明该接口是一个MyBatis中的Mapper，这种方式需要在每一个Mapper上都添加注解；还有一种简单的方式是在配置类上添加@MapperScan（"包名"）注解，表示扫描某个包下的所有接口作为Mapper；

4. 创建BookMapper.xml

```xml
<mapper namespace="com.dk.mapper.BookMapper">
    <insert id="addBook" parameterType="com.dk.entity.Book">
        INSERT INTO book(name, author) VALUES (#{name}, #{author})
    </insert>
    <delete id="deleteBookById" parameterType="com.dk.entity.Book">
        DELETE FROM book where id = #{id}
    </delete>
    <update id="updateBookById" parameterType="com.dk.entity.Book">
        UPDATE book SET name = #{name}, author = #{author} WHERE id = #{id}
    </update>
    <select id="getBookById" parameterType="integer" resultType="com.dk.entity.Book">
        SELECT * FROM book WHERE id = #{id}
    </select>
    <select id="getAllBooks" resultType="com.dk.entity.Book">
        SELECT * FROM book
    </select>
</mapper>
```

代码解释：

* 针对BookMapper接口中的每一个方法都在BookMapper.xml中列出了实现
* #{}用来代替接口中的参数，实体类中的属性可以直接通过#{实体类属性名}获取

5. 创建Service和Controller

6. 配置pom.xml

在Maven 工程中，XML配置文件建议写在resources路径下，但是上文的Mapper.xml文件写在包下，Maven在运行时会忽略包下的XML文件，因此需要在pom.xml文件中重新指明资源文件位置

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
    </resources>
</build>
```

启动项目

![](../images/spring boot + vue/mybatis操作book表日志.png)

### 5.3 整合Spring Data JPA

JPA（Java Persistence API）和Spring Data是两个范畴的概念；

Hibernate是一个ORM框架，而JPA是一种ORM规范，JPA和Hibernate的关系就像JDBC和JDBC驱动的关系，即JPA指定了ORM规范，而Hibernate是这些规范的实现（事实上，是先有Hibernate后有JPA，JPA的规范起草者也是Hibernate的作者），因此从功能上来说，JPA相当于Hibernate的子集；

Spring Data是Spring的一个子项目，致力于简化数据库访问，通过规范的方法名来分析开者的意图，进而减少数据库访问层的代码量；Spring Data不仅支持关系型数据库，也支持非关系型数据库；

1. 创建数据库

创建数据库jpa

```sql
CREATE DATABASE `jpa` DEFAULT CHARACTER SET utf8
```

2. 创建项目

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.9</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

3. 数据库配置

```yaml
spring:
  datasource:
  ## 基本数据库配置
    username: root
    password: 123456
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql:///jpa
  ## jpa配置
  jpa:
    ## 是否打印sql
    show-sql: true
    ## mysql数据库
    database: mysql
    hibernate:
      ## 项目启动时，根据实体类更新数据库中的表（其他可选值为 crete、create-drop、validate、no）
      ddl-auto: update
    properties:
      hibernate:
        ## 使用方言是MySQL57Dialect
        dialect: org.hibernate.dialect.MySQL57Dialect
```

4. 创建实体类

```java
@Data
@Entity(name = "t_book")
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "book_name", nullable = false)
    private String name;

    @Column(name = "book_author", nullable = false)
    private String author;

    @Column(name = "book_price", nullable = false)
    private Float price;

    @Transient
    private String description;
}
```

代码解释：

* @Entity注解表示该类是一个实体类，在项目启动时会根据该类自动生成一张表，表的名称即@Entity注解中的name值，如果不配置name，则默认为类名
* 所有的实体类都要有主键，@Id注解表示该属性是一个主键，@GeneratedValue注解表示可以定制建的自动生成，strategy表示主键的生成策略
* 默认情况下，生成的表中字段的名称就是实体类中属性的名称，通过@Column注解可以定制生成字段的属性，name表示该属性对应的数据表中字段的名称，nullable表示该字段是否可以为空
* @Transient注解表示在生成数据库时的表映射，该属性被忽略，即不生成对应的字段。

5. 创建BookDao接口

创建BookDao接口，继承JpaRepository

```java
public interface BookDao extends JpaRepository<Book, Integer> {
    List<Book> getBooksByAuthorStartingWith(String author);
    List<Book> getBooksByPriceGreaterThan(Float price);
    @Query(value = "select * from t_book where id = (select max(id) from t_book)", nativeQuery = true)
    Book getMaxIdBook();
    @Query(value = "select b from t_book b where b.id > :id and b.author = :author")
    List<Book> getBooksByIdGreaterThanAndAuthor(@Param("author") String author, @Param("id") Integer id);
    @Query(value = "select b from t_book b where b.id < ?2 and b.name like %?1%")
    List<Book> getBooksByIdAndNameLike(String name, Integer id);
}
```

代码解释：

* 自定义BookDao继承自JpaRepository。其中提供了一些基本的数据库操作方法，有基本的增删改查、分页查询、排序查询等。
* 第二行定义的方法表示查询以某个字符开始的所有书
* 第三行定义的方法表示查询单价大于某个值的所有书
* Spring Data JPA中，只要方法的定义符合既定规范，Spring Data就能分析出开发者的意图。从而避免开发者定义SQL，所谓的既定规范，就是一定的方法命名规则

| KeyWords            | 方法命名举例                      | 对应的SQL                      |
| ------------------- | --------------------------------- | ------------------------------ |
| And                 | findByNameAndAge                  | where name = ? and age = ?     |
| Or                  | findByNameOrAge                   | where name = ? or age = ?      |
| Is                  | findByAgeIs                       | where age = ?                  |
| Equals              | findByEquals                      | where id = ?                   |
| Between             | findByAgeBetween                  | where age between ? and ?      |
| LessThan            | findByAgeLessThan                 | where age < ?                  |
| LessThanEquals      | findByAgeLessThanEquals           | where age <= ?                 |
| GreaterThan         | findByAgeGreaterThan              | where age > ?                  |
| GreaterThanEquals   | findByAgeGreaterThanEquals        | where age >= ?                 |
| After               | findByAgeAfter                    | where age > ?                  |
| Before              | findByAgeBefore                   | where age < ?                  |
| IsNull              | findByNameIsNull                  | where name is null             |
| IsNotNull,NotNull   | findByNameNotNull                 | where name is not null         |
| Not                 | findByGenderNot                   | where gender <>?               |
| In                  | findByAgeIn                       | where age in (?)               |
| NotIn               | findByAgeNotIn                    | where age not in (?)           |
| NotLike             | findByNameNotLike                 | where name not like ?          |
| Like                | findByNameLike                    | where name like ?              |
| StartingWith        | findByNameStartingWith            | where name like '?%'           |
| EndingWith          | findByNameEndingWith              | where name like '%?'           |
| Containing,Contains | findByNameContaining              | where name like '%?%'          |
| OrderBy             | findByAgeGreaterThanOrderByIdDesc | where age > ? order by id desc |
| True                | findByEnableTrue                  | where enabled = true           |
| False               | findByEnabledFalse                | where enabled = false          |
| IgnoreCase          | findByNameIgnoreCase              | where UPPER(name) = UPPER(?)   |

* 既定的方法命名规则不一定满足所有的开发需求，因此Spring Data JPA也支持自定义JPQL（Java Persistence Query Language）或者原生SQL，nativeQuery = true表示使用原生的SQL查询
* 第7~9行表示根据id和author进行查询，这里使用默认的JPQL语句。JPQL是一种面向对象表达式语言，可以将SQL语法和简单查询语句绑定在一起，使用这种语言编写的查询是可移植的，可以被编译成所有主流数据库服务器上面的SQL。JPQL与原生SQL语句类似，并且完全面向对象，通过类名和属性访问，而表示表名和表的属性，第7~9行的查询使用 `：id`、 `：name`这种方式进行参数绑定，注意：这里使用的列名是属性的名称而不是数据库中列的名称。
* 第10、11行也是自定义JPQL查询，不同的是传参方式使用 `？1、？2`这种方式，注意：方法中参数的顺序要与参数声明的顺序一致
* 如果BookDao中的方法涉及修改操作，就需要添加@Modifying注解并添加事务。

6. 创建BookService

```java
@Service
public class BookServiceImpl implements BookService {
    @Autowired
    BookDao bookDao;

    @Override
    public void addBook(Book book) {
        bookDao.save(book);
    }

    @Override
    public Page<Book> getBookByPage(Pageable pageAble) {
        return bookDao.findAll(pageAble);
    }

    @Override
    public List<Book> getBooksByAuthorStartingWith(String author) {
        return bookDao.getBooksByAuthorStartingWith(author);
    }

    @Override
    public List<Book> getBooksByPriceGreaterThan(Float price) {
        return bookDao.getBooksByPriceGreaterThan(price);
    }

    @Override
    public Book getMaxIdBook() {
        return bookDao.getMaxIdBook();
    }

    @Override
    public List<Book> getBooksByIdGreaterThanAndAuthor(String author, Integer id) {
        return bookDao.getBooksByIdGreaterThanAndAuthor(author, id);
    }

    @Override
    public List<Book> getBooksByIdAndNameLike(String name, Integer id) {
        return bookDao.getBooksByIdAndNameLike(name, id);
    }
}
```

代码解释：

* 第六行使用save方法将对象数据保存到数据库，save方法是由JpaRepository接口提供的。
* 第九行是一个分页查询，使用findAll方法，返回值为Page<Book>，该对象中包含分页常用数据，例如总记录数，总页数，每页记录数，当前页记录数等。

7. 创建BookController

创建Controller，对数据测试

```java
@RestController
public class BookController {
    @Autowired
    BookService bookService;

    @GetMapping("findAll")
    public void findAll() {
        PageRequest pageRequest = PageRequest.of(2, 3);
        Page<Book> bookByPage = bookService.getBookByPage(pageRequest);
        System.out.println("总页数：" + bookByPage.getTotalPages());
        System.out.println("总记录数：" + bookByPage.getTotalElements());
        System.out.println("查询结果：" + bookByPage.getContent());
        System.out.println("当前页数：" + (bookByPage.getNumber() + 1));
        System.out.println("当前页记录数：" + bookByPage.getNumberOfElements());
        System.out.println("每页记录数：" + bookByPage.getSize());
    }

    @GetMapping("/search")
    public void search() {
        List<Book> b1 = bookService.getBooksByIdGreaterThanAndAuthor("鲁迅", 7);
        List<Book> b2 = bookService.getBooksByAuthorStartingWith("吴");
        List<Book> b3 = bookService.getBooksByIdAndNameLike("西", 8);
        List<Book> b4 = bookService.getBooksByPriceGreaterThan(30F);
        Book maxIdBook = bookService.getMaxIdBook();
        System.out.println("b1:" + b1);
        System.out.println("b2:" + b2);
        System.out.println("b3:" + b3);
        System.out.println("b4:" + b4);
        System.out.println("maxIdBook:" + maxIdBook);
    }

    @GetMapping("/save")
    public void save() {
        Book book = new Book();
        book.setAuthor("鲁迅");
        book.setName("呐喊");
        book.setPrice(23F);
        bookService.addBook(book);
    }
}
```

代码解释：

* 在findAll接口中，首先通过调用PageReques中的`of`方法构造PageRequest对象，`of`方法接受两个参数：第一个参数是页数，从0开始；第二个参数是每页显示的条数；
* 在save接口中构造一个Book对象，直接调用save方法保存起来即可；

### 5.4 多数据源

所谓多数据源，就是一个JavaEE项目中采用了不同数据库实例中的多个库，或者同一个数据库实例中多个不同的库，一般来说，采用MyCat等分布式数据库中间件是比较好的解决方案。这样可以把数据库读写分离、分库分表、备份等操作交给中间件去做，Java代码只需要专注业务即可。不过，这并不意味着无法使用Java代码解决类似的问题。在Spring Framework中就可以配置多数据源，在Spring Boot中继承其衣钵，只不过配置方式有所变化

#### 5.4.1 JdbcTemplate多数据源

JdbcTemplate多数据源的配置是比较简单的，因为一个JdbcTemplate对应一个DataSource，开发者只需要手动提供多个DataSource，再手动配置JdbcTemplate即可

1. 创建数据库

创建两个数据库：`chapter01` 和 `chapter02`，两个库中都创建表 `book`，再各预设一条数据

```sql
-- 创建数据库
CREATE DATABASE `chapter01` DEFAULT CHARACTER SET utf8;
CREATE DATABASE `chapter02` DEFAULT CHARACTER SET utf8;
-- 创建表
USE `chapter01`;
CREATE TABLE `book` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(128) DEFAULT NULL,
`author` varchar(128) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
-- 插入数据
INSERT INTO `book` (`id`, `name`, `author`) VALUES (1, '水浒传', '施耐庵');
-- 创建表
USE `chapter02`;
CREATE TABLE `book` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(128) DEFAULT NULL,
`author` varchar(128) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
-- 插入数据
INSERT INTO `book` (`id`, `name`, `author`) VALUES (1, '三国演义', '罗贯中');
```

2. 创建项目

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

注意，这里添加的连接池依赖是 `druid-spring-boot-starter`，它可以帮助开发者在Spring Boot项目中轻松集成Druid数据库连接池和监控。

3. 配置数据库连接

在application.properties中配置数据库连接信息

```yaml
spring:
  datasource:
    ## 数据源1
    one:
      type: com.alibaba.druid.pool.DruidDataSource
      username: root
      password: 123456
      url: jdbc:mysql:///chapter01?useUnicode=true&characterEncoding=UTF-8
    ## 数据源2
    two:
      type: com.alibaba.druid.pool.DruidDataSource
      username: root
      password: 123456
      url: jdbc:mysql:///chapter02?useUnicode=true&characterEncoding=UTF-8
```

配置两个数据源，区别主要是数据库不同

4. 配置数据源

创建DataSourceConfig配置数据源，根据Application.properties中的配置生成两个数据源：

```java
@Configuration
public class DataSourceConfig {
    @Bean
    @ConfigurationProperties("spring.datasource.one")
    DataSource dataSourceOne() {
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.two")
    DataSource dataSourceTwo() {
        return DruidDataSourceBuilder.create().build();
    }
}
```

代码解释：

* DataSourceConfig中提供了两个数据源，dataSourceOne和dataSourceTwo，默认方法名即实例名；
* @ConfigurationProperties注解表示使用不同前缀的配置文件来创建不同的DataSource实例

5. 配置JdbcTemplate

```java
@Configuration
public class JdbcTemplateConfig {
    @Bean
    JdbcTemplate jdbcTemplateOne(@Qualifier("dataSourceOne") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean
    JdbcTemplate jdbcTemplateTwo(@Qualifier("dataSourceTwo") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

代码解释：

* JdbcTemplateConfig中提供两个JdbcTemplate实例。每个JdbcTemplate实例都需要提供DataSource，由于Spring容器中有两个DataSource实例，因此需要通过方法名查找。@Qualifier注解表示查找不同名称的DataSource实例注入进来。

6. 创建Book、BookController

```java
@Data
public class Book {
    private Integer id;
    private String name;
    private String author;
}
```

```java
@RestController
public class BookController {
    @Resource(name = "jdbcTemplateOne")
    JdbcTemplate jdbcTemplateOne;

    @Autowired
    @Qualifier("jdbcTemplateTwo")
    JdbcTemplate jdbcTemplateTwo;

    @GetMapping("/test1")
    public void test1() {
        List<Book> bookList1 = jdbcTemplateOne.query("select * from book", new BeanPropertyRowMapper<>(Book.class));
        List<Book> bookList2 = jdbcTemplateTwo.query("select * from book", new BeanPropertyRowMapper<>(Book.class));
        System.out.println("bookList1 = " + bookList1);
        System.out.println("bookList2 = " + bookList2);
    }
}
```

简单起见，没有添加Service层，直接将JdbcTemplate注入到了Controller中，在Controller中注入两个不同的JdbcTemplate有两种方式：一种是使用@Resource注解，并指明 **name** 属性，即按 name 进行装配，此时会根据实例名查找相应的实例注入；另一种是使用@AutoWired注解结合@Qualifier注解，效果等同于@Resource注解

测试输出结果：

```
bookList1 = [Book(id=1, name=水浒传, author=施耐庵)]
bookList2 = [Book(id=1, name=三国演义, author=罗贯中)]
```

#### 5.4.2 MyBatis多数据源

JdbcTemplate可以配置多数据源，MyBatis也可以配置，但是步骤稍微复杂

1. 准备工作

本案例中使用数据库与5.4.1小节一致，依赖基本一致，只不过将spring-boot-starter-jdbc依赖换成MyBatis依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
</dependencies>
```

2. 创建MyBatis配置

配置MyBatis主要提供SqlSessionFactory实例和SqlSessionTemplate实例

```java
@Configuration
@MapperScan(value = "com.dk.mapper.one", sqlSessionFactoryRef = "sqlSessionFactoryBeanOne")
public class MyBatisConfigOne {
    @Autowired
    @Qualifier("dataSourceOne")
    DataSource dataSource;

    @Bean
    SqlSessionFactory sqlSessionFactoryBeanOne() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        return factoryBean.getObject();
    }

    @Bean
    SqlSessionTemplate sqlSessionTemplateOne() throws Exception {
        return new SqlSessionTemplate(sqlSessionFactoryBeanOne());
    }
}
```

数据源二同理：

```java
@Configuration
@MapperScan(value = "com.dk.mapper.two", sqlSessionFactoryRef = "sqlSessionFactoryBeanTwo")
public class MyBatisConfigTwo {

    @Resource(name = "dataSourceTwo")
    DataSource dataSource;

    @Bean
    SqlSessionFactory sqlSessionFactoryBeanTwo() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        return factoryBean.getObject();
    }

    @Bean
    SqlSessionTemplate sqlSessionTemplateTwo() throws Exception {
        return new SqlSessionTemplate(sqlSessionFactoryBeanTwo());
    }
}
```

代码解释：

* 在@MapperScan注释中指定Mapper接口所在位置，同时指定SqlSessionFactory的实例名，则该位置下的Mapper将使用SqlSessionFactory实例
* 提供SqlSessionFacory实例，直接创建出来，同时将DataSource的实例设置给SqlSessionFactory，这里创建的SqlSessionFacory实例也就是@MapperScan注解的sqlSessionFactoryRel参数指定的实例
* 提供一个SqlSessionTemplate实例，这是一个线程安全类，主要用来管理MyBatis中的SqlSession操作。

3. 创建Mapper

```java
@Mapper
public interface MapperOne {
    List<Book> getAllBooks();
}
```

对应的Mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.dk.mapper.one.MapperOne" >
    <select id="getAllBooks" resultType="com.dk.entity.Book">
        SELECT * FROM book;
    </select>
</mapper>
```

数据源二同理；

4. 创建Controller，简便起见，直接将Mapper注入Controller中

```java
@RestController
public class BookController {

    @Autowired
    MapperOne mapperOne;

    @Autowired
    MapperTwo mapperTwo;

    @GetMapping("/test")
    public void test() {
        List<Book> bookList1 = mapperOne.getAllBooks();
        List<Book> bookList2 = mapperTwo.getAllBooks();
        System.out.println("bookList1 = " + bookList1);
        System.out.println("bookList2 = " + bookList2);
    }
}
```

5. 测试结果

```
bookList1 = [Book(id=1, name=水浒传, author=施耐庵)]
bookList2 = [Book(id=1, name=三国演义, author=罗贯中)]
```

#### 5.4.3 JPA多数据源

JPA和MyBatis配置多数据源类型，不同的是，JPA配置时主要提供不同的LocalContainerEntityManagerFactoryBean以及事务管理器。

1. 准备工作

将jdbc依赖替换成jpa依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

application.properties在多数据源原有的基础上再添加如下配置

```properties
## 多数据源配置
spring.datasource.one.password=123456
spring.datasource.one.username=root
spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.one.url=jdbc:mysql:///chapter01?useUnicode=true&characterEncoding=UTF-8
spring.datasource.two.password=123456
spring.datasource.two.username=root
spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.two.url=jdbc:mysql:///chapter02?useUnicode=true&characterEncoding=UTF-8
## 新增jpa配置
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57InnoDBDialect
spring.jpa.hibernate.ddlAuto=update
spring.jpa.properties.hibernate.database=mysql
spring.jpa.showSql=true
spring.jpa.openInView=true
```

> 注意：这里的配置与单独的JPA有区别，因为在后文的配置中要从JpaProperties中的getProperties方法中获取JPA配置，因此这里的JPA属性前缀使用 spring.jpa.properties

JpaProperties部分源码如下

```java
@ConfigurationProperties(
    prefix = "spring.jpa"
)
public class JpaProperties {
    private Map<String, String> properties = new HashMap();
    private final List<String> mappingResources = new ArrayList();
    private String databasePlatform;
    private Database database;
    private boolean generateDdl = false;
    private boolean showSql = false;
    private Boolean openInView;
    private JpaProperties.Hibernate hibernate = new JpaProperties.Hibernate();

    public JpaProperties() {
    }
    //...
}
```

DataSourceConfig的配置也基本相同，不同的是这里多了一个@Primary注解

```java
@Configuration
public class MyDataSource {
    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.one")
    DataSource dataSourceOne() {
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.two")
    DataSource dataSourceTwo() {
        return DruidDataSourceBuilder.create().build();
    }
}
```

2. 创建实体类

```java
@Data
@Entity(name = "t_user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private String gender;
    private Integer age;
}
```

3. 创建JPA配置

接下来是核心配置，根据两个配置好的数据源创建两个不同的JPA配置

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = "com.dk.dao.one", entityManagerFactoryRef = "entityManagerFactoryBeanOne",
 transactionManagerRef = "platformTransactionManagerOne")
public class JpaConfigOne {
    @Resource(name = "dataSourceOne")
    DataSource dataSource;
    @Autowired
    JpaProperties jpaProperties;
    @Bean
    @Primary
    LocalContainerEntityManagerFactoryBean entityManagerFactoryBeanOne(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(dataSource)
                .properties(jpaProperties.getProperties())
                .packages("com.dk.entity")
                .persistenceUnit("pu1")
                .build();
    }

    @Bean
    PlatformTransactionManager platformTransactionManagerOne(EntityManagerFactoryBuilder builder) {
        LocalContainerEntityManagerFactoryBean factoryOne = entityManagerFactoryBeanOne(builder);
        return new JpaTransactionManager(Objects.requireNonNull(factoryOne.getObject()));
    }
}
```

代码解释：

* 使用@EnableJpaRepository注解进行JPA的配置，该注解中主要配置三个属性：basePackages、entityManagerFactoryRef以及transactionManagerReg。其中，basePackages用来指定Repository所在的位置，entityManagerFactoryRef用来指定实体类管理工厂Bean的名称、transactionManagerRef则用来指定事务管理器的引用名称，这里的引用名称就是JpaConfigOnw类中注册的Bean的名称（默认的Bean名称为方法名）
* 创建LocalContainerEntityManagerFactoryBean，该Bean将用来提供EntityManager实例，在该类的创建过程中，首先配置数据源，然后设置JPA相关配置（JpaProperties由系统自动加载），再设置实体类所在的位置，最后配置持久化单元名，若项目中只有一个EntityManagerFactory，则persistenceUnit可以省略，若有多个必须明确指定持久化单元名。
* 由于项目中会提供两个LocalContainerEntityManagerFactoryBean实例，@Primary表示当存在多个LocalContainerEntityManagerFactoryBean实例时，该实例将被优先使用。
* platformTransactionManagerOne方法表示创建一个事务管理器，JpaTransactionManager提供对单个EntityManagerFactory的事务支持，专门用于解决JPA中的事务管理。

JPA配置类JpaConfigTwo同样，不同点是JpaConfigTwo的LocalContainerEntityManagerFactoryBean实例不需要加@Primary注解；

4. 创建Repository

```java
public interface UserDaoOne extends JpaRepository<User, Integer> {
}
public interface UserDaoTwo extends JpaRepository<User, Integer> {
}
```

5. 创建Controller

```java
@RestController
public class UserController {
    @Autowired
    UserDaoOne userDaoOne;
    @Autowired
    UserDaoTwo userDaoTwo;

    @GetMapping("/user")
    public void saveUser() {
        User userOne = new User();
        userOne.setAge(55);
        userOne.setName("鲁迅");
        userOne.setGender("男");
        userDaoOne.save(userOne);
        User userTwo = new User();
        userTwo.setAge(80);
        userTwo.setName("泰戈尔");
        userTwo.setGender("男");
        userDaoTwo.save(userTwo);
    }
}
```

## 6.0 Spring Boot整合NoSQL

`NoSQL`是指非关系型数据库，非关系型数据库与关系型数据库两者存在许多显著的不同点，其中最重要的是NoSQL不使用SQL作为查询语言。其数据存储可以不需要固定的表格模式，一般都有水平可扩展性的特征。NoSQL主要有以下几种不同的分类：

* Key/Value键值存储。这种数据存储通常都是无数据结构的，一般被当作字符串或者是二进制数据，但是数据加载速度快，典型的使用场景是处理高并发或者用于日志系统等，这一类的数据库有 `redis`、 `Tokyo Cabinet`等。
* 列存储数据库。列存储数据库功能相对局限，但是查找速度快，容易进行分布式扩展，一般用于分布式文件系统中，这一类的数据库有 `HBase`、 `Cassandra`等。
* 文档型数据库。和Key/Value键值存储类似，文档型数据库也没有严格的数据格式，这既是缺点也是有点，因为不需要预先创建表结构，数据格式更加灵活，一般用于Web应用中，这一类的数据库有`MongoDB` 、 `CouchDB`等。
* 图形数据库。图形数据库专注于构建关系图谱，例如社交网络，推荐系统等，这一类的数据库有 `Neo4j` 、 `DEX`等。

### 6.1 整合Redis

#### 6.1.1 Redis简介

Redis是一个使用C编写的`基于内存`的NoSQL数据库，它是目前最流行的键值对存储数据库。Redis由一个Key、Value映射的字典构成，与其他NoSQL不同，Redis中的Value的类型不局限于字符串，还支持列表、集合、有序集合、散列等。Redis不仅可以当作缓存用，也可以配置数据持久化后当作NoSQL数据库使用，目前支持两种持久化方式：`快照持久化`和`AOF持久化`。另一方面，Redis也可以搭建集群或者主从复制结构，在高并发环境下具有高可用性。

#### 6.1.2 Redis安装

Redis版本使用4.0.10版本

1. 下载Redis

```
wget http://download.redis.io/releases/redis-4.0.10.tar.gz
```

提示未找到 **wget**命令

```
yum -y install wget
```

2. 安装Redis

解压下载的文件，进入解压目录进行编译

```
tar -zxf redis-4.0.10.tar.gz
cd redis-4.0.10
make MALLOC=libc
make install
```

若在执行 make MALLOC=libc 命令时提示 "gcc：未找到命令"，则先安装gcc

```
yum -y install gcc
```

3. 配置Redis

Redis安装成功后，修改redis.conf

```
daemonize yes
#bind 127.0.0.1
requirepass 123456
protected-mode no
```

配置解释：

* 第一行配置表示允许Redis在后台启动
* 第二行配置表示绑定允许连接Redis实例的地址，注释该配置，外网则可以连接redis了
* 第三行配置表示登陆该Redis实例所需的密码
* 由于有了第三行的配置，需要密码登陆，所以第四行可以关闭保护模式了

4. 配置Centos

关闭Centos7的防火墙

```
systemctl stop firewalld.service
systemctl disable firewalld.service
```

第一行表示关闭防火墙，第二行表示禁止防火墙开机启动

5. Redis启动/关闭

```
redis-server redis.conf
```

Redis启动成功后，再执行如下命令进入Redis控制台

```
redis-cli -a 123456
```

进入控制台执行`ping`命令，如果能看到 PONG，表示Redis安装成功。

如果想关闭Redis实例，可以在控制台执行 `SHUTDOWN`，然后使用 `exit`退出，或者直接在命令行执行如下命令

```
redis-cli -p 6379 -a 123456 shutdown
```

-p 表示端口号，-a 表示登陆密码

**使用Docker安装Redis**

```powershell
docker pull redis:4.0.10
docker run --restart=always -p 11080:11080 --name redis -v /c/Users/Kay/Documents/Dev/DockerData/redis4.0.10/redis.conf:/etc/redis/redis.conf -v /c/Users/Kay/Documents/Dev/DockerData/redis4.0.10/data:/data -d redis:4.0.10 redis-server /etc/redis/redis.conf
```

命令解释：

* 从Docker Hub上下载redis:4.0.10镜像
* docker run 启动一个docker容器，--restart=always表示自动启动，-p 11080：11080表示容器内的端口映射，--name redis容器名为redis，-v 表示宿主机的磁盘目录挂载到容器内的文件路径，-d 表示实例容器使用的镜像，redis-server /etc/redis/redis.conf表示启动redis服务使用映射的redis.conf配置文件

#### 6.1.3 整合Spring Boot

Redis的Java客户端又很多，例如：Jedis、JRedis、Spring Data Redis等，Spring Boot借助于Spring Data Redis为Redis提供了开箱即用的自动化配置。

1. 创建Spring Boot项目

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

默认情况下，spring-boot-starter-data-redis使用的Redis工具是Lettuce，也可以自定义引入Jedis

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

2. 配置Redis

接下来在application.properties中配置Redis连接信息

```properties
spring.redis.port=11080
spring.redis.host=localhost
spring.redis.database=0
spring.redis.password=123456
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-idle=8
spring.redis.jedis.pool.max-wait=-1
spring.redis.jedis.pool.min-idle=0
```

* 第一行表示redis的端口是11080
* 第二行表示redis连接的url是本机
* 第三行表示数据库的编号，Redis默认提供了16个Database，编号从0~15
* 第四行表示连接Redis的auth密码
* 第五行表示连接池的最大连接数
* 第六行表示连接池中最大空闲连接数
* 第七行表示连接池最大阻塞等待时间，默认为-1，表示没有限制
* 第八行表示连接池最小空闲连接数
* 如果项目使用了Lettuc，则只需要修改第五~八行配置中的jedis修改成lettuce即可

在Spring Boot中的自动配置类中提供了RedisAutoConfiguration进行了Redis的配置

```java
@Configuration
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```

由这一段源码可以看出，当我们没有提供额外的Redis实例时，Spring Boot会根据Application.properties配置中的RedisProperties，注入并创建一个默认的RedisTemplate。

3. 创建实体类

```java
@Data
public class Book implements Serializable {
    private static final long serialVersionUID = -6295498343158900155L;
    private String name;
    private String author;
    private float price;
}
```

4. 创建Controller

```java
@RestController
public class BookController {
    RedisTemplate redisTemplate;
    StringRedisTemplate stringRedisTemplate;
    @Autowired
    public void setRedisTemplate(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    @Autowired
    public void setStringRedisTemplate(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @GetMapping("/test")
    private void test() {
        ValueOperations<String, String> stringOps = stringRedisTemplate.opsForValue();
        stringOps.set("bookName", "三国演义");
        String bookName = stringOps.get("bookName");
        System.out.println("bookName = " + bookName);
        ValueOperations ops = redisTemplate.opsForValue();
        Book book = new Book();
        book.setName("红楼梦");
        book.setAuthor("曹雪芹");
        book.setPrice(23.5F);
        ops.set("book", book);
        Object redisBook = ops.get("book");
        System.out.println("redisBook = " + redisBook);
    }
}
```

代码解释：

* StringRedisTemplate是RedisTemplate的子类，StringRedisTemplate中的Key和Value都是字符串，采用的序列化方案是 `StringRedisSerializer`，而RedisTemplate则可以用来操作对象，RedisTemplate采用的序列化方案是 `JdkSerializationRedisSerializer`，无论是StringRedisTemplate还是RedisTemplate，操作Redis的方法都是一致的。
* StringRedisTemplate和RedisTemplate都是通过opsForValue，opsForZSet或者opsForSet等方法首先获取一个操作对象，再使用该操作对象完成数据的读写。

#### 6.1.4 Redis集群整合Spring Boot

为了提高Redis的可扩展性，往往需要搭建Redis集群环境，这样就涉及到了Redis集群整合Spring Boot

1. 搭建Redis集群

   **集群原理**

   在Redis集群中，所有的Redis节点彼此互联，节点内部使用二进制协议优化传输速度和带宽。当一个节点挂掉后，集群中超过半数的节点检测失效时才认为该节点已失效。不同于Tomcat集群需要使用反向代理服务器，Redis集群中的任意节点都可以直接和Java客户端连接。Redis集群上的数据分配则是采用哈希槽（HASH SLOT），Redis集群中内置了`16384`个哈希槽，当有数据需要存储时，Redis会首先使用CRC16算法对Key进行计算，将计算获得的结果对16384取模，这样每一个Key都会对应一个取值在 `0~16383`之间的哈希槽，Redis则根据这个值将该条数据存储到对应的Redis节点上
   
   **集群规划**
   
   主节点：
   
   127.0.0.1:8001;127.0.0.1:8002;127.0.0.1:8003
   
   从节点：
   
   127.0.0.1:9001;127.0.0.1:9002;127.0.0.1:9003
   
   **集群配置**
   
   根据节点端口创建目录并配置各个节点配置；
   
   ![](../images/spring boot + vue/redis集群配置.png)
   
   以8001主节点的redis.conf为例
   
   ```properties
   取消绑定仅本机可访问的限制
   # bind 127.0.0.1
   配置了密码，关闭保护模式
   protected-mode no
   端口
   port 8001
   开启后台启动
   daemonize yes
   进程文件
   pidfile /data/redis/cluster/8001/redis_8001.pid
   日志文件
   logfile "/data/redis/cluster/8001/redis-8001.log"
   集群中每个节点都开启了密码认证，masterauth使从机可以登陆到主机上
   masterauth 123456
   密码认证
   requirepass 123456
   持久化开启
   appendonly yes
   集群开启
   cluster-enabled yes
   集群配置文件
   cluster-config-file nodes-8001.conf
   ```
   
   其他端口配置只需要修改端口号即可；
   
   ```
   sed -i 's/8001/8002/g' redis.conf
   ```
   
   启动所有实例；
   
   修改redis-trib.rb，由于每个节点都添加了密码，而该命令默认没有密码，因此登陆不上各个redis实例
   
   ```ruby
   @r = Redis.new(:host => @info[:host], :port => @info[:port], :password => 123456, :timeout => 60)
   ```
   
   **创建集群**
   
   执行如下命令创建redis集群
   
   ```powershell
   ./redis-trib.rb create --replicas 1 127.0.0.1:8001 127.0.0.1:8002 127.0.0.1:8003 127.0.0.1:9001 127.0.0.1:9002 127.0.0.1:9003
   ```
   
   命令解释：
   
   * replicas 表示每个主节点的slave数量，在集群的创建过程中会分配主机和从机，每个集群在创建过程中都将分配到一个唯一的id并分配到一段slot；
   
   执行命令后，输出如下信息；
   
   ```powershell
   >>> Creating cluster
   >>> Performing hash slots allocation on 6 nodes...
   Using 3 masters:
   127.0.0.1:8001
   127.0.0.1:8002
   127.0.0.1:8003
   Adding replica 127.0.0.1:9002 to 127.0.0.1:8001
   Adding replica 127.0.0.1:9003 to 127.0.0.1:8002
   Adding replica 127.0.0.1:9001 to 127.0.0.1:8003
   >>> Trying to optimize slaves allocation for anti-affinity
   [WARNING] Some slaves are in the same host as their master
   M: 440f991109947657dcbd746849e824e2670c8374 127.0.0.1:8001
      slots:0-5460 (5461 slots) master
   M: b3404b23021935b5d291f5541fd0c6de9df1ffc5 127.0.0.1:8002
      slots:5461-10922 (5462 slots) master
   M: 2a9baf6db12398a858f6348c3fc4edf2ed22aef4 127.0.0.1:8003
      slots:10923-16383 (5461 slots) master
   S: b1ccf934826c8223316db57795dbaa159c0b1b52 127.0.0.1:9001
      replicates 440f991109947657dcbd746849e824e2670c8374
   S: 195e39251721619d9071b6b85470895988ce41f8 127.0.0.1:9002
      replicates b3404b23021935b5d291f5541fd0c6de9df1ffc5
   S: 38d734c14a003686fd0de124896c302c6ae86769 127.0.0.1:9003
      replicates 2a9baf6db12398a858f6348c3fc4edf2ed22aef4
   Can I set the above configuration? (type 'yes' to accept): yes
   >>> Nodes configuration updated
   >>> Assign a different config epoch to each node
   >>> Sending CLUSTER MEET messages to join the cluster
   Waiting for the cluster to join...
   >>> Performing Cluster Check (using node 127.0.0.1:8001)
   M: 440f991109947657dcbd746849e824e2670c8374 127.0.0.1:8001
      slots:0-5460 (5461 slots) master
      1 additional replica(s)
   S: b1ccf934826c8223316db57795dbaa159c0b1b52 127.0.0.1:9001
      slots: (0 slots) slave
      replicates 440f991109947657dcbd746849e824e2670c8374
   S: 195e39251721619d9071b6b85470895988ce41f8 127.0.0.1:9002
      slots: (0 slots) slave
      replicates b3404b23021935b5d291f5541fd0c6de9df1ffc5
   M: 2a9baf6db12398a858f6348c3fc4edf2ed22aef4 127.0.0.1:8003
      slots:10923-16383 (5461 slots) master
      1 additional replica(s)
   M: b3404b23021935b5d291f5541fd0c6de9df1ffc5 127.0.0.1:8002
      slots:5461-10922 (5462 slots) master
      1 additional replica(s)
   S: 38d734c14a003686fd0de124896c302c6ae86769 127.0.0.1:9003
      slots: (0 slots) slave
      replicates 2a9baf6db12398a858f6348c3fc4edf2ed22aef4
   [OK] All nodes agree about slots configuration.
   >>> Check for open slots...
   >>> Check slots coverage...
   [OK] All 16384 slots covered.
   ```
   
   执行时报错；
   
   ```powershell
   /usr/bin/env: ‘ruby’: No such file or directory
   ```
   
   原因是该脚本需要 `ruby`环境
   
   安装ruby环境后报错
   
   ```powershell
   Traceback (most recent call last):
           2: from ./redis-trib.rb:25:in `<main>'
           1: from /usr/lib/ruby/2.5.0/rubygems/core_ext/kernel_require.rb:59:in `require'
   /usr/lib/ruby/2.5.0/rubygems/core_ext/kernel_require.rb:59:in `require': cannot load such file -- redis (LoadError)
   ```
   
   原因是缺少模块；执行如下命令
   
   ```powershell
    gem install redis
   ```
   
   当集群创建成功后，登陆任意Redis实例
   
   ```powershell
   ./redis-cli -p 8001 -a 123456 -c
   ```
   
   -p 表示要登陆的集群实例端口，-a 表示登陆集群节点的验证密码，-c 则表示以集群的方式登陆。登陆成功后，通过`cluster info`命令查看集群状态信息；通过 `cluster nodes` 命令可以查询节点信息，在集群节点信息中，可以看到每一个节点的id，该节点是slave还是master；如果是slave那么它的master的id是什么，如果是master，那么每一个master的slot范围是多少，这些信息都会显示出来；
   
   

![](../images/spring boot + vue/cluster-info.png)

![](../images/spring boot + vue/cluster-nodes.png)

**添加主节点**

当集群创建成功后，随着业务的增长，可能需要增加主节点，添加主节点需要先构建主节点实例；新增8004目录，修改redis.conf后启动该实例

启动成功后，执行如下命令新增节点

```powershell
./redis-trib.rb add-node 127.0.0.1:8004 127.0.0.1:8001
```

命令解释：

* add-node表示新增节点
* 127.0.0.1:8004 表示新增的实例
* 127.0.0.1:8001 表示集群中任意的节点实例

![](../images/spring boot + vue/redis-add-node.png)

新增节点成功，redis-cli登陆后查看集群信息；可以看到新实例已经被添加到集群中，但是由于slot已经被之前的实例分配完了，新增加的实例没有slot，也就意味着新增加的实例没有存储数据的机会；此时需要从之前的master中拿出一部分slot分配给新实例；

![](../images/spring boot + vue/新增master节点且未分配hashslot.png)

首先，对slot重新分配；

```powershell
root@Kay:/data/redis/cluster# ./redis-trib.rb reshard 127.0.0.1:8001
>>> Performing Cluster Check (using node 127.0.0.1:8001)
M: 440f991109947657dcbd746849e824e2670c8374 127.0.0.1:8001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: b1ccf934826c8223316db57795dbaa159c0b1b52 127.0.0.1:9001
   slots: (0 slots) slave
   replicates 440f991109947657dcbd746849e824e2670c8374
S: 195e39251721619d9071b6b85470895988ce41f8 127.0.0.1:9002
   slots: (0 slots) slave
   replicates b3404b23021935b5d291f5541fd0c6de9df1ffc5
M: 2a9baf6db12398a858f6348c3fc4edf2ed22aef4 127.0.0.1:8003
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: b3404b23021935b5d291f5541fd0c6de9df1ffc5 127.0.0.1:8002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: d314401e0ca39949505e5e2b5d1cdf13b8c794e4 127.0.0.1:8004
   slots: (0 slots) master
   0 additional replica(s)
S: 38d734c14a003686fd0de124896c302c6ae86769 127.0.0.1:9003
   slots: (0 slots) slave
   replicates 2a9baf6db12398a858f6348c3fc4edf2ed22aef4
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 3000
What is the receiving node ID? d314401e0ca39949505e5e2b5d1cdf13b8c794e4
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1:all
```

命令解释：

* `How many slots do you want to move (from 1 to 16384)? `拿出多少slot分配给新实例
* `What is the receiving node ID?`这些slot分配给的实例id
* `Please enter all the source node IDs.
    Type 'all' to use all the nodes as source nodes for the hash slots.
    Type 'done' once you entered all the source nodes IDs.
  Source node #1:all`这些slot由哪些master节点提供；输入all为所有master实例均摊提供这些slot

**添加从节点**

上面的添加的节点是主节点，从节点相对来说要容易一些，添加从节点命令如下；

```powershell
./redis-trib.rb add-node --slave --master-id d314401e0ca39949505e5e2b5d1cdf13b8c794e4 127.
0.0.1:9004 127.0.0.1:8001
```

![](../images/spring boot + vue/添加从节点成功.png)

**删除节点**

如果删除的是一个从节点，直接运行如下命令即可

```powershell
./redis-trib.rb del-node 127.0.0.1:8001 abcf36d1d8eb442364b640d61f83e8ecc5c6c203
```

中间的实例地址表示集群中的任意一个实例；最后的参数表示要删除节点的id；但若删除的节点占有slot，则会删除失败；此时需要将该节点的所有slot全部分配出去，然后再运行如上命令就可以删除一个slot的节点了。

```powershell
>>> Removing node abcf36d1d8eb442364b640d61f83e8ecc5c6c203 from cluster 127.0.0.1:8001
>>> Sending CLUSTER FORGET messages to the cluster...
>>> SHUTDOWN the node.
```

2. 配置Spring Boot项目

不同于单机版本，集群版本需要手动配置

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
    </dependency>
</dependencies>
```

由于节点有多个，可以保存在一个集合中，因此这里的配置文件使用YAML格式的

```yaml
spring:
  redis:
    cluster:
      ports:
        - 8001
        - 8002
        - 8003
        - 9001
        - 9002
        - 9003
      host: 127.0.0.1
      poolConfig:
        max-total: 8
        max-idle: 8
        max-wait-millis: -1
        min-idle: 0
```

由于Redis实例的host都是一样的，因此这里配置了一个host；而port配置成了一个集合；这些port都将被注入一个集合中，poolConfig则是基本的连接池信息配置；

创建RedisConfig，完成对Redis的配置；

```java
@Data
@Configuration
@ConfigurationProperties("spring.redis.cluster")
public class RedisConfig {
    List<Integer> ports;
    String host;
    JedisPoolConfig poolConfig;

    @Bean
    RedisClusterConfiguration redisClusterConfiguration() {
        RedisClusterConfiguration configuration = new RedisClusterConfiguration();
        List<RedisNode> nodes = new ArrayList<>();
        for (Integer port : ports) {
            nodes.add(new RedisNode(host, port));
        }
        configuration.setPassword(RedisPassword.of("123456"));
        configuration.setClusterNodes(nodes);
        return configuration;
    }

    @Bean
    JedisConnectionFactory jedisConnectionFactory() {
        return new JedisConnectionFactory(redisClusterConfiguration(), poolConfig);
    }

    @Bean
    RedisTemplate redisTemplate() {
        RedisTemplate redisTemplate = new RedisTemplate();
        redisTemplate.setConnectionFactory(jedisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());
        return redisTemplate;
    }

    @Bean
    StringRedisTemplate stringRedisTemplate() {
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate(jedisConnectionFactory());
        stringRedisTemplate.setKeySerializer(new StringRedisSerializer());
        stringRedisTemplate.setValueSerializer(new StringRedisSerializer());
        return stringRedisTemplate;
    }
}
```

代码解释：

* 通过@ConfigurationProperties注解声明配置文件前缀，配置文件中定义的ports数组、host以及连接池配置信息都将被注入port、host、poolConfig三个属性中；
* 配置RedisClusterConfiguration实例，设置Redis登陆密码以及Redis节点信息；
* 根据RedisClusterConfiguration实例以及连接池配置信息创建Jedis连接工厂JedisConnectionFactory
* 根据JedisConnectionFactory创建RedisTemplate和StringRedisTemplate，同时配置Key和Value的序列化方式，有了RedisTemplate和StringRedisTemplate，剩下的用法就和单实例的用法一致；

创建Controller和entity；略；

测试，启动项目，在浏览器中访问  `http://localhost/test`

![](../images/spring boot + vue/redis集群控制台输出.png)

### 6.2 整合MongoDB

#### 6.2.1 MongoDB简介

MongoDB 是一种面向文档的数据库管理系统，它是一个介于关系型数据库和非关系型数据库之间的产品， MongoDB 功能丰富，它支持一种类似 JSON 的 BSON 数据格式，既可以存储简单的数据格式， 也可以存储复杂的数据类型。 MongoDB 最大的特点是它支持的查询语言非常强大，并且还支持对数据建立索引。总体来说， MongoDB 是一款应用相当广泛的 NoSQL 数据库。

#### 6.2.2 MongoDB安装

下载并解压mongoDB；进入mongodb目录，创建data与logs目录

进入bin目录下，创建一个新的MongoDB配置文件mongo.conf

```properties
dbpath=/data/mongoDB/mongodb/data
logpath=/data/mongoDB/mongodb/logs
port=27017
fork=true
```

配置解释：

* 第 1 行配直表示数据存储目录。
* 第 2 行自己直表示日志文件位直。
* 第 3 行配直表示启动端口。
* 第 4 行配直表示以守护程序的方式启动 MongoDB，即九许 MongoDB 在后台运行。

**MongoDB的启动和关闭**

配置完成后执行如下命令启动MongoDB

```powershell
./mongod -f mongo.conf --bind_ip_all
```

`-f`表示指定配置文件的位置，`-- bind_ip_all` 则表示允许所有的远程地址连接该 MongoDB 实例。 MongoDB 启动成功后，在 bin 目录下再执行 mongo 命令，进入 MongoDB 控制台，然后输入`db.version()`，如果能看到 MongoDB 的版本号， 就表示安装成功：

```powershell
> db.version()
4.0.18
```

默认情况下，启动后连接的是 MongoDB 中的 test 库，而关闭MongoDB 的命令需要在 admin 库中执行，因此关闭 MongoDB 需要首先切换到 admin 库， 然后执行 `db.shutdownServer()`命令，完整操作步骤如下：

```powershell
> use admin
switched to db admin
> db.shutdownServer()
2020-05-25T00:56:56.866+0800 I NETWORK  [js] DBClientConnection failed to receive message from 127.0.0.1:27017 - HostUnreachable: Connection closed by peer
server should be down...
2020-05-25T00:56:56.870+0800 I NETWORK  [js] trying reconnect to 127.0.0.1:27017 failed
2020-05-25T00:56:56.870+0800 I NETWORK  [js] reconnect 127.0.0.1:27017 failed failed
> exit
bye
2020-05-25T00:57:14.597+0800 I NETWORK  [js] trying reconnect to 127.0.0.1:27017 failed
2020-05-25T00:57:14.598+0800 I NETWORK  [js] reconnect 127.0.0.1:27017 failed failed
2020-05-25T00:57:14.598+0800 I QUERY    [js] Failed to end session { id: UUID("eed9fa0f-29c8-434b-8c59-e57dc60fd1ce") } due to SocketException: socket exception [CONNECT_ERROR] server [couldn't connect to server 127.0.0.1:27017, connection attempt failed: SocketException: Error connecting to 127.0.0.1:27017 :: caused by :: Connection refused]
```

服务关闭后，执行 exit 命令退出控制台， 此时如果再执行 `./mongo` 命令就会执行失败

```powershell
MongoDB shell version v4.0.18
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
2020-05-25T00:57:49.624+0800 E QUERY    [js] Error: couldn't connect to server 127.0.0.1:27017, connection attempt failed: SocketException: Error connecting to 127.0.0.1:27017 :: caused by :: Connection refused :
connect@src/mongo/shell/mongo.js:344:17
@(connect):2:6
exception: connect failed
```

**安全管理**

默认情况下，启动的MongoDB没有登陆密码，在生产环境中这是非常不安全的，但是不同于MySQL、Oracle等关系型数据库，MongoDB中每一个库都有独立的密码，在哪一个库中创建用户就要在哪个库中验证密码，要配置密码，首先要创建一个用户。例如在admin库中创建一个用户

```powershell
> use admin;
switched to db admin
> db.createUser({user:"dingkay",pwd:"123456",roles:[{role:"readWrite",db:"test"}]})
Successfully added user: {
        "user" : "dingkay",
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "test"
                }
        ]
}
```

新创建的用户名为dingkay，密码是123456，roles表示该用户具有的角色，这里的配置表示该用户对test库具有读和写两项权限。

用户创建成功后，关闭当前实例，然后重新启动，启动命令如下；

```powershell
./mongod -f mongo.conf --auth --bind_ip_all
```

启动成功后，再次进入控制台，然后切换到admin库中验证登陆（默认连接上的是test库）验证成功后就可以对test库执行读写操作了

```powershell
> db.auth("dingkay","123456")
1
```

如果db.auth()命令执行结果为1，就表示认证成功。

#### 6.2.3 整合Spring Boot

借助于Spring Data MongoDB，Spring Boot为MongoDB也提供了开箱即用的自动化配置方案

1. 创建Spring Boot工程

创建Spring Boot Web工程，添加MongoDB依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
</dependencies>
```

2. 配置MongoDB

在application.properties中配置MongoDB的连接信息

```properties
spring.data.mongodb.authentication-database=admin
spring.data.mongodb.database=test
spring.data.mongodb.host=127.0.0.1
spring.data.mongodb.port=27017
spring.data.mongodb.username=dingkay
spring.data.mongodb.password=123456
```

代码解释：

* 第一行配置表示验证登陆信息的库
* 第二行配置表示要连接的库，认证信息不一定要在连接的库中创建，因此这两个分开配置。
* 第三行~第六行配置表示MongoDB的连接地址和认证信息等。

3. 创建实体类

```java
@Data
public class Book {
    private String author;
    private String name;
    private float price;
}
```

4. 创建BookDao

```java
public interface BookDao extends MongoRepository<Book, Integer> {
    List<Book> findByAuthorContains(String author);
    Book findByNameEquals(String name);
}
```

MongoRepository中已经预定了针对实体类的查询、添加、删除等操作。BookDao中可以按照5.3节提到的命名规则定义查询方法。

5. 创建Controller

```java
@RestController
public class BookController {
    @Autowired
    MongoDao mongoDao;

    @GetMapping("/test")
    public void test() {
        List<Book> books = new ArrayList<>();
        Book book = new Book();
        book.setName("朝花夕拾");
        book.setAuthor("鲁迅");
        book.setPrice(21.5F);
        books.add(book);
        book = new Book();
        book.setName("呐喊");
        book.setAuthor("鲁迅");
        book.setPrice(26.0F);
        books.add(book);
        mongoDao.insert(books);
        List<Book> bookList = mongoDao.findByAuthorContains("鲁迅");
        System.out.println("bookList = " + bookList);
        Book bookObj = mongoDao.findByNameEquals("呐喊");
        System.out.println("bookObj = " + bookObj);
    }
}
```

代码解释：

* 调用MongoRepository中的insert方法插入集合数据
* findByAuthorContains查询作者名中包含"鲁迅"的数据
* findByNameEquals查询书名为呐喊的数据

登录MongoDB服务器，认证身份后，可以查询到数据

![](../images/spring boot + vue/登录mongodb查看book数据.png)

6. 使用MongoTemplate

除了继承MongoRepositorty，Spring Data MongoDB还提供了MongoTemplate用来方便的操作MongoDB。相关配置在MongoDataAutoConfiguration类中。

```java
@Data
public class BookA {
    private Integer id;
    private String name;
    private String author;
    private Float price;
}
@RestController
public class BookController {
    @Autowired
    MongoTemplate mongoTemplate;

    @GetMapping("/mongo")
    public void mongo() {
        List<BookA> books = new ArrayList<>();
        BookA book = new BookA();
        book.setId(1);
        book.setName("围城");
        book.setAuthor("钱钟书");
        book.setPrice(25.5F);
        books.add(book);
        book = new BookA();
        book.setId(2);
        book.setName("宋诗选注");
        book.setAuthor("钱钟书");
        book.setPrice(24.5F);
        books.add(book);
        mongoTemplate.insertAll(books);
        List<BookA> all = mongoTemplate.findAll(BookA.class);
        System.out.println("all = " + all);
        BookA byId = mongoTemplate.findById(2, BookA.class);
        System.out.println("byId = " + byId);
    }
}
```

### 6.3 Session共享

![](../images/spring boot + vue/Session共享.png)

正常情况下，HttpSession是通过Servlet容器创建并存储在内存中的，如果需要对项目进行横向扩展搭建集群时，那么可以利用硬件或者软件来做负载均衡，此时来自同一用户的HTTP请求就有可能被分散到不同的实例上去，如何保证各个实例之间的Session同步就成为一个必须解决的问题。Spring Boot提供了自动化Session共享配置，它结合Redis可以非常方便地解决这个问题，使用Redis解决Session共享问题的原理非常简单，就是把原本存储在不同服务器上的Session拿出来存储在一个独立的服务器上。

当一个请求到达Nginx 服务器后，首先进行请求分发，假设请求被 real server 1 处理了， real server 1 在处理请求时，无论是存储 Session 还是读取 Session， 都去操作 Session 服务器而不是操作自身内存中的 Session，其他 real server 在处理请求时也是如此，这样就可以实现 Session 共享了。

#### 6.3.1 Session 共享配置

创建Spring Boot项目，添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
        <exclusions>
            <exclusion>
                <groupId>io.lettuce</groupId>
                <artifactId>lettuce-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
</dependencies>
```

除了Redis依赖之外，这里还要提供spring-session-data-redis依赖，Spring Session可以做到透明化的替换掉应用的Session容器，项目创建成功后，在application.yml中进行基本的redis配置

```yaml
spring:
  redis:
    host: 127.0.0.1
    port: 11080
    password: 123456
    database: 0
    jedis:
      pool:
        max-active: 8
        max-idle: 8
        max-wait: -1
        min-idle: 0
```

创建一个Controller用来测试操作

```java
@RestController
public class HelloController {
    @Value("${server.port}")
    String port;
    @PostMapping("/save")
    public String saveName(String name, HttpSession session) {
        session.setAttribute("name", name);
        return port;
    }

    @GetMapping("/get")
    public String getName(HttpSession session) {
        return port + ":" + session.getAttribute("name").toString();
    }
}
```

这里提供了两个方法，一个save接口用来向Session中存储数据，还有一个get接口用来从Session中获取数据，这里注入了项目启动的端口号server.port，主要是为了区分到底哪个服务器提供的服务，另外，虽然还是操作的HttpSession，但是实际上HttpSession容器已经被透明替换，真正的Session此时存储在Redis服务器上

创建项目后，打成jar包，上传到服务器上，然后执行如下两条命令启动项目

```powershell
nohup java -jar chapter01-1.0-SNAPSHOT.jar --server.port=8080 &
nohup java -jar chapter01-1.0-SNAPSHOT.jar --server.port=8081 &
```

nohup表示不挂断程序运行，即当终端窗口关闭后，程序依然在后台运行，最后的&表示让程序在后台运行。`--server.port`表示设置启动端口，一个为8080，另一个为8081。启动成功后，接下来就可以配置负载均衡了。

#### 6.3.2 Nginx负载均衡

安装Nginx

下载源码并解压，然后进入解压目录进行编译安装

```powershell
./configure --prefix=xxx
make
make install
```

安装过程中的异常可能是因为缺少依赖环境

> pcre/zlib/openssl/g++(ubuntu)

安装成功后，进入自定义安装配置的prefix中的路径，执行sbin目录下的nginx文件启动nginx

启动成功后，默认端口是80，可以直接访问查看

然后修改nginx的配置文件

```
    upstream dingkay.com {
    server 127.0.0.1:8080 weight=1;
    server 127.0.0.1:8081 weight=1;
    }

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass http://dingkay.com;
            proxy_redirect default;
        }
```

> 这里只列出了修改的配置，在修改的配置中首先配置上游服务器，即两个real server，两个real server的权重都是1，意味着请求将平均分配到两个real server上，然后在server中配置拦截规则，将拦截到的请求转发到定义好的real server上

配置完成后，重启Nginx

```powershell
./nginx -s reload
```

#### 6.3.3 请求分发

当服务和Nginx都启动后，调用save接口

![](../images/spring boot + vue/请求分发save.png)

调用的端口是80，即调用的是Nginx服务器，请求会被Nginx转发到real server上进行处理，返回值为8081，说明真正处理请求的是8081那台服务器，接下来调用get获取数据

![](../images/spring boot + vue/请求分发get.png)

调用的端口依然是80，但是返回值是8080，说明是8080这台服务器提供的服务。

## 7.0 构建RESTful服务

### 7.1 REST简介

REST(Representational State Transfer) 是一种Web软件架构风格，它是一种风格，而不是标准，匹配或兼容这种架构风格的网络服务称为REST服务。REST服务简洁并且有层次，REST通常基于HTTP、URI和XML以及HTML这些现有的广泛流行的协议和标准。在REST中，资源是由URI来指定的，对资源的增删改查操作可以通过HTTP协议提供的GET、POST、PUT、DELETE等方法实现。使用REST可以更高效的利用缓存来提高响应速度，同时REST中的通信会话状态由客户端来维护，这可以让不同的服务器处理一系列请求中的不同请求，进而提高服务器的扩展性。在前后端分离项目中，一个设计良好的Web软件架构必然要满足REST风格。

在Spring MVC框架中，开发者可以通过@RestController注解开发一个RESTful服务，不过，Spring Boot对此提供了自动化配置方案。

### 7.2 JPA实现REST

在Spring Boot中，使用Spring Data JPA和Spring Data Rest可以快速开发出一个RESTful应用。

#### 7.2.1 基本实现

1. 创建项目

添加所需依赖

```xml
<dependencies>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.9</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>
```

这里的依赖除了数据库相关的依赖外，还有Spring Data JPA的依赖以及Spring Data Rest的依赖。项目创建完成后，在application.properties中配置基本的数据库连接信息

```properties
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.url=jdbc:mysql:///jparestful?useUnicode=true&characterEncoding=utf-8
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.jpa.show-sql=true
spring.jpa.database=mysql
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
```

2. 创建实体类

```java
@Data
@Entity(name = "t_book")
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private String author;
}
```

3. 创建BookRepository

```java
public interface BookRepository extends JpaRepository<Book, Integer> {
}
```

创建BookRepository类继承JpaRepository，JpaRepository中默认提供了一些基本操作方法

```java
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    List<T> findAll();

    List<T> findAll(Sort var1);

    List<T> findAllById(Iterable<ID> var1);

    <S extends T> List<S> saveAll(Iterable<S> var1);

    void flush();

    <S extends T> S saveAndFlush(S var1);

    void deleteInBatch(Iterable<T> var1);

    void deleteAllInBatch();

    T getOne(ID var1);

    <S extends T> List<S> findAll(Example<S> var1);

    <S extends T> List<S> findAll(Example<S> var1, Sort var2);
}
```

从源码可以看出来，基本的增删改查、分页都提供了

4. 测试

经过如上几步，一个RESTful的服务就构成了，使用Postman测试

RESTful服务构建成功后，默认的请求路径是实体类名小写再加上后缀，此时向数据库添加一条数据非常容易，发起一个Post请求，请求地址为localhost:8080/books

![](../images/spring boot + vue/RESTful插入book数据.png)

5. 查询测试

查询请求是GET请求，分页查询的请求路径为/books

![](../images/spring boot + vue/RESTful查询book数据.png)

如果要根据id查询可以在请求后面加上id值

```
http://localhost:8080/books/2
```

分页查询数据，只需要在请求地址中携带上相关参数即可

```
http://localhost:8080/books?page=0&size=3
```

![](../images/spring boot + vue/RESTful分页查询book数据.png)

分页查询的同时还可以根据id排序查询

![](../images/spring boot + vue/RESTful分页查询book数据并排序.png)

发送PUT请求，根据id修改数据

![](../images/spring boot + vue/RESTful修改book数据.png)

发送DELETE请求可以实现对数据的删除操作

```
http://localhost:8080/books/1
```

DELETE请求没有返回值，上面的这个请求发送成功后，id为1的记录就被删除了

#### 7.2.2 自定义请求路径

默认情况下，请求路径为实体类名加小写s，如果开发者想对请求路径进行重定义，通过@RepositoryRestResource注解即可实现，下面的案例只需在BookRepository上添加@RepositoryRestResource注解即可

```java
@RepositoryRestResource(path = "bks", collectionResourceRel = "bk", itemResourceRel = "book")
public interface BookRepository extends JpaRepository<Book, Integer> {
}
```

path属性表示将所有的请求路径中的books都转换成bks，如localhost:8080/bks；collectionResourceRel属性表示返回JSON集合中的key修改为bk，itemResourceRel属性表示将返回的JSON集合中的单个book的key修改为book

![](../images/spring boot + vue/RESTful自定义请求路径.png)

#### 7.2.3 自定义查询方法

默认的查询方法支持分页查询、排序查询以及按照id查询，如果需要自定义根据某个属性查询，只需要在BookRepository中定义相关方法并暴露出去即可

```java
@RepositoryRestResource(path = "bks", collectionResourceRel = "bk", itemResourceRel = "book")
public interface BookRepository extends JpaRepository<Book, Integer> {
    List<Book> findByAuthorContains(@Param("author") String author);
    @RestResource(path = "name", rel = "name")
    Book findByName(@Param("name") String name);
}
```

代码解释：

* 自定义查询只需要在BookRepository中定义相关查询方法即可，方法定义好之后可以不添加@RestResource注解，默认路径就是方法名，如果想要自定义路径，只需要添加@RestResource注解，path属性为最新路径，rel表示实体中映射的属性名
* 直接访问localhost:8080/bks/search可以查看暴露了哪些查询方法

![](../images/spring boot + vue/RESTful查看自定义查询.png)

#### 7.2.4 隐藏方法

默认情况下，凡是继承了Repository接口或者Repository的子类，都会被暴露出来，即开发者可执行基本的增删改查，如果继承了Repository但又不想暴露则可以在类或者方法加上@RepositoryRestResource/@RestResource的exported属性置为false；那么整个类/接口就会失效

```java
@RepositoryRestResource(path = "bks", collectionResourceRel = "bk", itemResourceRel = "book")
public interface BookRepository extends JpaRepository<Book, Integer> {
    List<Book> findByAuthorContains(@Param("author") String author);
    @RestResource(path = "name", rel = "name")
    Book findByName(@Param("name") String name);
    @Override
    @RestResource(exported = false)
    Optional<Book> findById(Integer integer);
}
```

#### 7.2.5 配置CORS

之前介绍了CORS两种不同的配置方式，一种是直接在方法上添加@CrossOrigin注解，另一种是全局配置，全局配置依然适用，但是默认的RESTful工程不需要开发者自己提供Controller，因此添加在Controller的方法上的注解可以直接写在BookRepository上。

```java
@CrossOrigin
@RepositoryRestResource(path = "bks", collectionResourceRel = "bk", itemResourceRel = "book")
public interface BookRepository extends JpaRepository<Book, Integer> {
    @Override
    @RestResource(exported = false)
    Optional<Book> findById(Integer id);
    @RestResource(path = "list")
    List<Book> findBooksByAuthorContains(String author);
}
```

此时BookRepository中的所有方法都支持跨域。如果只需要某一个方法支持跨域，那么将@CrossOrigin注解添加到某一个方法上即可。

#### 7.2.6 其它配置

开发者也可以在application.properties中配置一些常用属性

```properties
# 每页默认记录数，默认值为20
spring.data.rest.default-page-size=2
# 分页查询页码参数名，默认值为page
spring.data.rest.page-param-name=page
# 分页查询记录数参数名，默认值为size
spring.data.rest.limit-param-name=size
# 分页查询排序参数名，默认值为sort
spring.data.rest.sort-param-name=sort
# base-path 表示给所有请求路径都加上前缀
spring.data.rest.base-path=/api
# 添加成功时是否返回添加内容
spring.data.rest.return-body-on-create=true
# 更新成功时是否返回更新内容
spring.data.rest.return-body-on-update=true
```

当然，这些配置也可以在Java代码中配置，且代码中配置的优先级高于application.properties配置的优先级

```java
@Configuration
public class RestConfig extends RepositoryRestConfigurerAdapter {
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config) {
        config.setBasePath("/api")
                .setDefaultPageSize(2)
                .setLimitParamName("size")
                .setPageParamName("page")
                .setSortParamName("sort")
                .setReturnBodyOnCreate(true)
                .setReturnBodyOnUpdate(true);
    }
}
```

### 7.3 MongoDB实现REST

创建项目，添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

创建实体类

```java
@Data
public class Book {
    private Integer id;
    private String author;
    private String name;
}
```

创建BookRepository

```java
public interface BookRepository extends MongoRepository<Book, Integer> {
}
```

配置MongoDB连接信息

```properties
spring.data.mongodb.password=123456
spring.data.mongodb.username=dingkay
spring.data.mongodb.port=27017
spring.data.mongodb.host=127.0.0.1
spring.data.mongodb.database=test
spring.data.mongodb.authentication-database=admin
```

如此之后，一个RESTful服务就搭建成功了；

## 8.0 开发者工具与单元测试

### 8.1 devtools简介

Spring Boot中提供了一组开发工具spring-boot-devtools，可以提高开发者的工作效率，开发者可以将该模块包含在任何项目中，spring-boot-devtools最方便的地方莫过于热部署了。

### 8.2 devtools实战

#### 8.2.1 基本用法

只需要添加相关依赖即可

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

> 这里多了一个optional选项，是为了防止将devtools依赖传递到其他模块中，当开发者将应用打包运行后，devtools会被自动禁用。

当引入项目依赖后，只要classpath路径下的文件发生了变化，项目就会自动重启，这极大地提高了项目的开发速度。如果开发者使用了Eclipse，那么在修改完代码并保存后，项目将自动编译并出发重启，而如果使用了Intellij IDEA，默认情况下，需要开发者手动编译才会出发重启。手动编译时，单机Build -> Build Project菜单或者按Ctrl + F9 快捷键进行编译，编译成功后就会触发项目重启。当然，使用Intellij IDEA的开发者也可以配置项目自动编译，步骤如下；

单击File -> Settings 菜单，打开Settings 页面，在左边的菜单栏找到Build，Execution，Deployment -> Compile，勾选Build project automatically

![](../images/spring boot + vue/IDEA自动编译.png)

按Ctrl + Shift + Alt + / 快捷键，调出Maintenance界面

![](../images/spring boot + vue/Maintenance界面.png)

单击Registry，在新打开的Registry页面中，勾选compiler.automake.allow.when.app.running复选框



做完这两部配置后，若开发者再次在Intellij IDEA中修改代码，则项目会自动重启。

> classpath路径下的静态资源或者视图模板等变化时，并不会导致项目重启

![](../images/spring boot + vue/Registry界面.png)

#### 8.2.2 基本原理

Spring Boot中使用的自动重启技术涉及两个类加载器，一个是baseclassloader，用来加载不会变化的类，例如第三方的jar；另一个是restartclassloader，用来加载开发者自己写的会变化的类，当项目需要重启时，restartclassloader将被一个新创建的类加载器代替，而baseclassloader则继续使用原来的，这种启动方式要比冷启动快很多，因为baseclassloader已经存在并且已经加载好。

#### 8.2.3 自定义监控资源

默认情况下，/META-INF/maven、/META-INF/resources、/resources、/static、/public以及/templates位置下资源的变化并不会触发重启，如果开发者想要对这些位置进行重定义，在application.properties中添加如下配置即可；

```properties
spring.devtools.restart.exclude=static/**
```

这表示从默认的不处罚重启的目录中除去static目录，即classpath:static 目录下的资源发生变化时也会导致项目重启。用户也可以反配置需要监控的目录，配置方式如下：

```properties
spring.devtools.restart.additional-path=stc/main/resources/startic
```

这个配置表示当src/main/resources/static目录下的文件发生变化时，自动重启项目。

由于项目的编码过程是一个连续的过程，并不是每修改一行代码就要重启项目，这样不仅浪费电脑性能，而且没有实际意义。鉴于这种情况，开发者也可以考虑使用出发文件，出发文件是一个特殊的文件，当这个文件发生变化时就会重启

```properties
spring.devtools.restart.trigger-file=.trigger-file
```

在项目resources目录下创建一个名为.trigger-file的文件，此时当开发者修改代码时，默认情况下项目不会重启，需要项目重启时，开发者只需要修改.trigger-file文件即可。但是注意，如果项目没有改变，只是单纯的修改了.trigger-file文件，那么项目不会重启。

#### 8.2.4 使用LiveReload

上一小节介绍了静态资源目录下的文件变化以及模板文件的变化不会引发重启，虽然开发者可以通过修改配置改变这一默认情况，但实际上并没有必要，因为静态文件不是class，devtools默认嵌入了LiveReload服务器，可以解决静态文件的热部署，LiveReload可以在资源发生变化时自动触发浏览器更新，LiveReload支持Chrome、Firefox以及Safari。以Chrome为例，在Chrome应用商店搜索LiveReload

在浏览器中打开项目的页面，然后单击浏览器右上角的LiveReload按钮，开启LiveReload连接，此时当静态资源发生改变时，浏览器就会自动加载。如果开发者不想使用这个特性，可以通过以下配置关闭

```properties
spring.devtools.livereload.enabled=false
```

> 建议开发者使用LiveReload策略而不是项目重启策略来实现静态资源的动态加载，因为项目重启所耗费的时间一般要超过LiveReload

#### 8.2.5 禁用自动重启

如果开发者添加了spring-boot-devtools依赖，但是不想使用自动重启特性，那么可以通过配置关闭自动重启

```properties
spring.devtools.restart.enabled=false
```

也可以在Java代码中配置禁止自动重启，配置方式如下

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        System.setProperty("spring.devtools.restart.enabled", "false");
        SpringApplication.run(App.class, args);
    }
}
```

#### 8.2.6 全局配置

如果项目模块众多，开发者可以在当前用户目录下创建`.spring-boot-devtools.properties`文件来对devtools进行全局配置，这个配置文件适用于当前计算机上任何使用了devtools模块的Spring Boot项目。

```properties
spring.devtools.restart.trigger-file=.trigger-file
```

此时，就实现了使用触发文件触发项目重启。

### 8.3 单元测试

在本书前面的章节中，遇到需要测试的地方都是创建一个Controller进行测试，这样操作臃肿，效率低下，在Spring Boot中使用单元测试可以实现对每一个环节的代码进行测试。Spring Boot中的单元测试与Spring中的测试一脉相承，但是又做了大量的简化，只需要少量的代码就能搭建一个测试环境，进而实现对Controller、Service或者Dao层的代码进行测试。

#### 8.3.1 基本用法

创建一个项目引入spring-boot-starter-test依赖，并且创建好了测试类、

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test01 {
    @Test
    public void contextLoads() {}
}
```

代码解释：

* 这里首先使用了@RunWith注解，该注解将JUnit执行类修改为SpringRunner，而SpringRunner是Spring Framework中测试类SpringJUnit4ClassRunner的别名。

* @SpringBootTest注解提供了SpringTestContext中的常规测试功能之外，还提供了其他特性：提供默认的ContextLoader、自动搜索@SpringBootConfiguration、自定义环境属性、为不同的webEnvironment模式提供支持，这里的webEnvironment模式主要有四种：

  MOCK，这种模式是当classpath下存在servletAPIS时，就会创建WebApplicationContext并提供一个mockservlet环境；当classpath下存在Spring WebFlux时，则创建ReactiveWebApplicationContext；若都不存在，则创建一个常规的ApplicationContext。

  RANDOM_PORT，这种模式将提供一个真实的Servlet环境，使用内嵌的容器，但是端口随机。

  DEFINED_PORT，这种模式也将提供一个真实的Servlet环境，使用内嵌的容器，但是使用定义好的端口。

  NONE，这种模式则加载一个普通的ApplicationContext，不提供任何Servlet环境。这种一般不适用于Web测试。

* 在Spring测试中，开发者一般使用@ContextConfiguration(classes = )或者@ContextConfiguration(locations = )来指定要加载的Spring配置，而在Spring Boot中则不需要这么麻烦，Spring Boot中的@*Test注解将会去包含测试类包下查找带有@SpringBootApplication或者@SpringBootConfiguration注解的主配置类。

* Junit中的@After、@AfterClass、@Before、@BeforClass、@Ignore等注解一样可以在这里使用。

#### 8.3.2 Service测试

Service层的测试就是常规测试，非常容易，例如现在有一个HelloService如下

```java
@Service
public class HelloService {
    public String sayHello(String name) {
        return "Hello " + name + "!";
    }
}
```

需要对HelloService进行测试，直接在测试类中注入HelloService即可；（测试类需要和启动类统一包下或其子包中）

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestHelloService {
    @Autowired
    HelloService helloService;
    @Test
    public void contextLoads() {
        String hello = helloService.sayHello("Ding");
        Assert.assertThat(hello, Matchers.is("Hello Ding!"));
    }
}
```

#### 8.3.3 Controller测试

Controller测试则要使用Mock测试，即对一些不易获取的对象采用虚拟的对象来创建而方便进行测试。而Spring中提供的MockMvc则提供了对HTTP请求的模拟，使开发者能够在不依赖网络环境的情况下实现对Controller的快速测试

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(String name) {
        return "Hello " + name + " !";
    }

    @PostMapping("/book")
    public String addBook(@RequestBody Book book) {
        return book.toString();
    }
}
```

对Controller进行测试需要借助Mock，代码如下；

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloControllerTest {
    @Autowired
    WebApplicationContext wac;
    MockMvc mockMvc;
    @Before
    public void before() {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }
    @Test
    public void test1() throws Exception {
        MvcResult mvcResult = mockMvc.perform(
                MockMvcRequestBuilders
                .get("/hello")
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                .param("name", "Ding"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcResultHandlers.print())
                .andReturn();
        System.out.println(mvcResult.getResponse().getContentAsString());
        Assert.assertThat(mvcResult.getResponse().getContentAsString(), Matchers.is("Hello Ding !"));
    }
    @Test
    public void test2() throws Exception {
        ObjectMapper om = new ObjectMapper();;
        Book book = new Book();
        book.setAuthor("罗贯中");
        book.setName("三国演义");
        book.setId(1);
        String s = om.writeValueAsString(book);
        MvcResult mvcResult = mockMvc.perform(
                MockMvcRequestBuilders
                .post("/book")
                .contentType(MediaType.APPLICATION_JSON)
                .content(s))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn();
        System.out.println(mvcResult.getResponse().getContentAsString());
    }
}
```

代码解释：

* @before表示在Test类之前执行，注入了一个WebApplicationContext用来模拟ServletContext环境，声明了一个MockMvc对象，并在每个测试方法执行前进行进行MockMvc的初始化操作
* 调用MockMvc中的perform方法开启一个RequestBuilder请求，具体的请求则通过MockMvcRequestBuilders进行构建，调用MockMvcRequestBuilders中的get方法表示发起一个GET请求，调用post方法则发起一个POST请求，其它的DELETE和PUT请求也是一样，最后通过调用param方法设置请求参数
* 添加返回验证规则，利用MockMvcResultMatchers进行验证，这里表示验证响应码是否为200
* test2方法演示了POST请求如何传递JSON数据，首先将一个book对象转换为JSON，然后设置请求的contentType为APPLICATION_JSON，最后设置content为上传的JSON即可。

除了MockMvc这种测试方式之外，Spring Boot还专门提供了TestRestTemplate用来实现集成测试，若开发者使用了@SpringBootTest注解，则TestRestTemplate将自动可用，直接在测试类中注入即可。主要，要使用TestRestTemplate进行测试，需要将SpringBootTest注解中的webEnvironment属性的默认值由WebEnvironment.MOCK修改为WebEnvironment.DEFINED_PORT或者WebEnvironment.RANDOM_PORT，因此这两种都是使用一个真实的Servlet环境而不是模拟的Servlet环境

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
public class HelloControllerDefinedPortTest {
    @Autowired
    TestRestTemplate testRestTemplate;
    @Test
    public void test3() {
        String name = "Ding";
        ResponseEntity<String> hello = testRestTemplate.getForEntity("/hello?name={0}", String.class, name);
        System.out.println("hello = " + hello.getBody());
        Assert.assertThat(hello.getBody(), Matchers.is(name));
    }
}
```

#### 8.3.4 JSON测试

开发者可以使用@JsonTest测试JSON序列化和反序列化是否工作正常，该注解将自动配置Jackson ObjectMapper、@JsonComponent以及Jackson Modules。如果开发者使用Gson代替Jackson，该注解将配置Gson；具体代码如下

```java
@RunWith(SpringRunner.class)
@JsonTest
public class BookJsonTest {
    @Autowired
    JacksonTester<Book> jacksonTester;
    @Test
    public void jsonTest1() throws IOException {
        Book book = new Book();
        book.setId(1);
        book.setName("三国演义");
        book.setAuthor("罗贯中");
        Assertions.assertThat(jacksonTester.write(book)).isEqualToJson("book.json");
        Assertions.assertThat(jacksonTester.write(book)).hasJsonPathStringValue("@.name");
        Assertions.assertThat(jacksonTester.write(book)).extractingJsonPathStringValue("@.name")
                .isEqualTo("三国演义");
    }
    @Test
    public void jsonTest2() throws Exception {
        String jsonString = "{\"id\":\"1\",\"name\":\"三国演义\",\"author\":\"罗贯中\"}";
        Assertions.assertThat(jacksonTester.parseObject(jsonString).getName()).isEqualTo("三国演义");
    }
}
```

book.json文件内容如下

```json
{"id":1,"name":"三国演义","author":"罗贯中"}
```

代码解释：

* 添加JacksonTester注解，使用JacksonTest进行JSON的序列化和反序列化测试
* 在序列化完成后判断序列化结果是否是所期待的json，book.json是一个定义在当前包下的JSON文件
* 判断对象序列化之后生成的JSON中是否有一个名为name的key
* 判断序列化后的name对应的值是否为"三国演义"
* 反序列化完成时判断对象的name属性是否为"三国演义"

## 9.0 Spring Boot缓存

Spring 3.1中开始对缓存提供支持，核心思路是对方法的缓存，当看调用一个方法时，将方法的参数和返回值作为key/value缓存起来，当再次调用该方法时，如果缓存中有数据，就直接从缓存中获取，否则再去执行该方法。但是，Spring中并未提供缓存的实现，而是提供了一套缓存API。目前Spring Boot支持的缓存有如下几种：

* JCache(JSR-107)
* EhCache 2.x
* Hazelcast
* Infinispan
* Couchbase
* Redis
* Caffeine
* Simple

目前常用的缓存实现Ehcache 2.x 和 Redis。由于Spring早已将缓存领域统一，因此无论使用哪种缓存实现，不同的只是缓存配置，开发者使用的缓存注解是一致的（Spring 缓存注解和各种缓存实现的关系就像JDBC和各种数据库驱动的关系一样）

### 9.1 Ehcache 2.x 缓存

Ehcache缓存在Java开发领域已是久负盛名，在Spring Boot中，只需要一个配置文件就可以将Ehcache集成到项目中

1. 创建项目，添加缓存依赖

创建SpringBoot项目，添加spring-boot-starter-cache依赖以及Ehcache依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>net.sf.ehcache</groupId>
        <artifactId>ehcache</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

2. 添加缓存配置文件

如果Ehcache的依赖存在，并且在classpath下有一个名为ehcache.xml的Ehcache配置文件，那么EhcacheCacheManager将会自动作为缓存的实现。因此，在resources目录下创建ehcache.xml文件作为Ehcache缓存的配置文件

```xml
<ehcache>
    <diskStore path="java.io.tmpdir/cache"/>
<defaultCache maxElementsInMemory="10000"
              eternal="false"
              timeToIdleSeconds="120"
              timeToLiveSeconds="120"
              overflowToDisk="false"
              diskPersistent="false"
              diskExpiryThreadIntervalSeconds="120"/>
<cache name="book_cache"
       maxElementsInMemory="10000"
       eternal="false"
       timeToIdleSeconds="120"
       timeToLiveSeconds="120"
       overflowToDisk="false"
       diskPersistent="false"
       diskExpiryThreadIntervalSeconds="600"/>
</ehcache>
```

这是一个常规的Ehcache配置文件，提供了两个缓存策略，一个是默认的，另一个名为book_cache。其中，name表示缓存名称；maxElementsInMemory表示缓存的最大格式；eternal表示缓存对象是否永久有效，一旦设置了永久有效，timeout将不起作用；timeToIdleSeconds表示缓存对象在失效前的允许闲置时间（单位：秒），当eternal=false对象不是永久有效时，该属性才生效；timeToLiveSeconds表示缓存对象在失效前允许存活的时间（单位：秒），当eternal=false时该配置才生效；overflowToDisk表示当内存中的对象数量达到maxElementsInMemory时，Ehcache是否将对象写到磁盘中，diskPersistent表示写入到磁盘中的对象是否时持久的；diskExpiryThreadIntervalSeconds表示磁盘失效线程运行时间间隔。如果想要自定义Ehcache配置文件的名称和位置，可以在application.properties中添加如下配置：

```properties
spring.cache.ehcache.config=classpath:config/another-config.xml
```

3. 开启缓存

```java
@SpringBootApplication
@EnableCaching
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

4. 创建BookDao

```java
@Repository
@CacheConfig(cacheNames = "book_cache")
public class BookDao {
    @Cacheable
    public Book getBookById(Integer id) {
        System.out.println("getBookById");
        Book book = new Book();
        book.setId(1);
        book.setAuthor("罗贯中");
        book.setName("三国演义");
        return book;
    }
    @CachePut(key = "#book.id")
    public Book updateBookById(Book book) {
        System.out.println("updateBookById");
        book.setName("三国演义2");
        return book;
    }
    @CacheEvict(key = "#id")
    public void deleteBookById(Integer id) {
        System.out.println("deleteBookById");
    }
}
```

实体类代码

```java
@Data
public class Book implements Serializable {
    private static final long serialVersionUID = -296852542896847046L;
    private Integer id;
    private String name;
    private String author;
}
```

代码解释：

* 在BookDao上添加@CacheConfig注解指明使用的缓存的名字，这个配置可选，若不使用@CacheConfig注解，则直接在@Cacheable注解中指明缓存名字
* getBookById方法上添加@Cacheable注解表示对该方法进行缓存，默认情况下，缓存的key时方法的参数，缓存的value是方法的返回值，当开发者在其它类中调用该方法时，首先会根据调用参数查看缓存中是否有相关数据，若有，则直接使用缓存数据，该方法不会执行，否则执行该方法，执行成功后将返回值缓存起来，但若是在当前类中调用该方法，则缓存不会生效
* @Cacheable注解中还有一个属性condition用来描述缓存的执行时机，例如@Cacheable（condition = "#id%2==0"）表示当id对2取模为0时才进行缓存，否则不缓存
* 如果开发者不想使用默认的key，也可以自定义key，除了使用参数定义key的方式之外，Spring还提供了一个root对象用来生成key;

| 属性名称    | 属性描述              | 用法示例             |
| ----------- | --------------------- | -------------------- |
| methodName  | 当前方法名            | #root.methodName     |
| method      | 当前方法对象          | #root.method.name    |
| caches      | 当前方法使用的缓存    | #root.caches[0].name |
| target      | 当前被调用的对象      | #root.target         |
| targetClass | 当前被调用对象的class | #root.targetClass    |
| args        | 当前方法参数数组      | #root.args[0]        |

* 如果这些key不能满足开发需求，也可以自定义缓存key的生成器KeyGenerator

```java
@Component
public class MyKeyGenerator implements KeyGenerator {
    @Override
    public Object generate(Object o, Method method, Object... objects) {
        return Arrays.toString(objects);
    }
}
```

则BookDao代码修改如下

```java
@Repository
@CacheConfig(cacheNames = "book_cache")
public class BookDao {
    @Autowired
    MyKeyGenerator myKeyGenerator;
	// ...
	// ...
    @Cacheable(keyGenerator = "myKeyGenerator")
    public Book getBookByIdWithMyKeyGenerator(Integer id, String name) {
        System.out.println("getBookByIdWithMyKeyGenerator");
        Book book = new Book();
        book.setId(1);
        book.setAuthor("Test");
        book.setName("MyKey");
        return book;
    }
}
```

代码解释：

* 自定义MyKeyGenerator实现KeyGenerator接口，然后实现该接口中的generate方法，该方法的三个参数分别是当前对象、当前请求的方法以及方法的参数，开发者可根据这些信息组成一个新的key返回，返回值就是缓存的key。@Cacheable注解中引用MyKeyGenerator实例即可
* @CachePut注解一般用于数据更新方法上，与@Cacheable注解不同，添加了@CachePut注解的方法每次再执行时都不会检查缓存中是否存在数据，而是直接执行方法，然后将方法的执行结果缓存起来，如果该key对应的数据已经被缓存，就会覆盖之前的数据，这样可以避免再次加载数据时获取到脏数据。同时，@CachePut具有和@Cacheable类似的属性
* @CacheEvict注解一般用于删除方法上，表示移除一个key对应的缓存。@CacheEvict注解有两个特殊的属性：allEntries和beforeInvocation，其中allEntries表示是否将所有缓存数据都删除，默认为false；beforeInvocation表示是否再方法执行前移除缓存中的数据，默认为false，即在方法执行后移除缓存中的数据。

5. 创建测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class EhcacheTest {
    @Autowired
    BookDao bookDao;
    @Test
    public void test() {
        bookDao.getBookById(1);
        bookDao.getBookById(1);
        bookDao.deleteBookById(1);
        Book book = bookDao.getBookById(1);
        System.out.println("test:getBookById.book = " + book);
        book.setId(1);
        bookDao.updateBookById(book);
        book = bookDao.getBookById(1);
        System.out.println("test:after updateBookById getBookById.book = " + book);
        bookDao.getBookByIdWithMyKeyGenerator(1, "Test");
        bookDao.getBookByIdWithMyKeyGenerator(1, "Test");
    }
}
```

执行该方法，控制台打印日志如下

```
getBookById
deleteBookById
getBookById
test:getBookById.book = Book(id=1, name=三国演义, author=罗贯中)
updateBookById
test:after updateBookById getBookById.book = Book(id=1, name=三国演义change, author=罗贯中change)
getBookByIdWithMyKeyGenerator
```

一开始执行了两次查询，但是查询方法只打印了一次，因为第二次使用了缓存，接下来执行删除方法，删除方法执行后再次执行查询，查询方法又被执行了，因为删除方法中缓存已经被删除了。接下来执行更新方法，更新方法中不仅更新数据，也更新了缓存，所以最后的查询方法中，查询方法没有打印日志，说明该方法没有执行，而是使用了缓存中的数据，而缓存中的数据已经被更新了。

### 9.2 Reids缓存

和Ehcache一样，如果在classpath下存在Redis并且Redis已经配置好了，此时默认就会使用RedisCacheManager作为缓存提供者，Redis单机使用步骤如下：

1. 创建项目，添加spring-boot-starter-cache依赖和redis依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
        <exclusions>
            <exclusion>
                <groupId>io.lettuce</groupId>
                <artifactId>lettuce-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

2. 缓存配置

Redis单机缓存只需要开发者配置application.properties中的redis配置及缓存配置即可

```properties
# 缓存配置
spring.cache.cache-names=c1,c2
spring.cache.redis.time-to-live=1800s
# redis配置
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.password=123456
spring.redis.port=11080
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-idle=8
spring.redis.jedis.pool.max-wait=-1ms
spring.redis.jedis.pool.min-idle=0
```

配置解释：

* 配置缓存名称，Redis中的key都有一个前缀，默认前缀就是`缓存名::`
* 配置缓存有效期，Redis中key的有效时间
* Redis基本连接配置

3. 开启缓存

```Java
@SpringBootApplication
@EnableCaching
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

### 9.3 Redis集群缓存

不同于Redis单机缓存，Redis集群缓存的配置要复杂一些，主要体现在配置上，缓存的使用还是和9.1节中介绍的一样。搭建Redis集群缓存主要分为三个步骤：1.搭建Redis集群；2.配置缓存；3.使用缓存。

#### 9.3.1 搭建Redis集群

Redis集群的搭建过程在6.1.4小节；

#### 9.3.2 配置缓存

集群redis的连接配置

```yaml
spring:
  redis:
    cluster:
      ports:
        - 8001
        - 8002
        - 8003
        - 9001
        - 9002
        - 9003
      host: 127.0.0.1
      password: 123456
      poolConfig:
        max-total: 8
        max-idle: 8
        max-wait-millis: -1
        min-idle: 0
```

配置集群的JedisConnectionFactory

```java
@Data
@Configuration
@ConfigurationProperties("spring.redis.cluster")
public class RedisClusterConfig {
    List<Integer> ports;
    String host;
    JedisPoolConfig poolConfig;
    String password;
    @Bean
    RedisClusterConfiguration redisClusterConfiguration() {
        RedisClusterConfiguration configuration = new RedisClusterConfiguration();
        List<RedisNode> nodes = new ArrayList<>();
        for (Integer port : ports) {
            RedisNode node = new RedisNode(host, port);
            nodes.add(node);
        }
        configuration.setClusterNodes(nodes);
        configuration.setPassword(RedisPassword.of(password));
        return configuration;
    }
    @Bean
    JedisConnectionFactory jedisConnectionFactory() {
        return new JedisConnectionFactory(redisClusterConfiguration(), poolConfig);
    }
}
```

配置集群RedisCache配置

```java
@Configuration
public class RedisCacheConfig {
    @Autowired
    JedisConnectionFactory jedisConnectionFactory;
    @Bean
    RedisCacheManager redisCacheManager() {
        Map<String, RedisCacheConfiguration> configMap = new HashMap<>();
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration
                .defaultCacheConfig()
                .prefixKeysWith("Ding")
                .disableCachingNullValues()
                .entryTtl(Duration.ofMinutes(30));
        configMap.put("c1", redisCacheConfiguration);
        RedisCacheWriter cacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(jedisConnectionFactory);
        return new RedisCacheManager(cacheWriter,
                RedisCacheConfiguration.defaultCacheConfig(), configMap);
    }
}
```

代码解释：

* 在配置Redis集群时，Spring容器中注入了JedisConnectionFactory的实例，这里注入到RedisCacheConfig中（RedisConnectionFactory是JedisConnectionFactory的父类）
* 在RedisCacheConfig中提供RedisCacheManager的实例，该实例的构建需要三个参数，第一个参数是一个cacheWriter，直接通过nonLockIngRedisCacheWriter的方法构造出来即可；第二个参数是默认的缓存配置；第三个参数是提前定义好的缓存配置。
* RedisCacheManager构造方法中第三个参数是一个提前定义好的缓存参数，它是一个Map类型的参数，该Map中的key就是指缓存名字，value就是该名称的缓存所对应的缓存配置，例如key的前缀、缓存过期时间等，若缓存注解中使用的缓存名称不存在于Map中，则使用RedisCacheManager构造方法中第二个参数所定义的默认缓存策略进行数据缓存；例如如下两个缓存配置：

```java
@Cacheable(value = "c1")
@Cacheable(value = "c2")
```

第一行注解中，c1存在于configMap集合中，因此使用的缓存策略是configMap集合中c1所对应的缓存策略；c2不存在于configMap集合中，因此使用的缓存策略是默认的缓存策略。

* 默认的缓存策略通过调用RedisCacheConfiguration中的defaultCacheConfig方法获取

```java
public static RedisCacheConfiguration defaultCacheConfig() {
    DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
    registerDefaultConverters(conversionService);
    return new RedisCacheConfiguration(Duration.ZERO, true, true, CacheKeyPrefix.simple(), SerializationPair.fromSerializer(new StringRedisSerializer()), SerializationPair.fromSerializer(new JdkSerializationRedisSerializer()), conversionService);
}
```

由这一段代码可以看到，默认的缓存过期时间为0，即永不过期；第二个参数true表示允许存储null值；第三个参数true表示开启key的前缀；第四个参数表示key的默认前缀是`缓存名::`；接下来两个参数表示key和value的序列化方式；最后一个参数则是一个类型转换器

#### 9.3.3 使用缓存

通过@EnableCache注解开启缓存，创建一个BookDao

```java
@Repository
public class BookDao {
    @Cacheable(cacheNames = "c1")
    public String findBookById(Integer id) {
        System.out.println("BookDao.findBookById");
        Book book = new Book();
        book.setId(id);
        book.setName("Book");
        book.setAuthor("Kay");
        return book.toString();
    }
    @CachePut(value = "c2")
    public String updateBookById(Integer id) {
        System.out.println("BookDao.updateBookById");
        Book book = new Book();
        book.setName("UpdateBook");
        book.setId(id);
        book.setAuthor("newBook");
        return book.toString();
    }
    @Cacheable(value = "c2")
    public String getBookById(Integer id) {
        System.out.println("BookDao.getBookById");
        Book book = new Book();
        book.setName("UpdateBook");
        book.setId(id);
        book.setAuthor("newBook");
        return book.toString();
    }
}
```

然后创建单元测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class BookCacheTest {
    @Autowired
    BookDao bookDao;
    @Test
    public void test() {
        System.out.println(bookDao.findBookById(5));
        System.out.println(bookDao.updateBookById(10));
        System.out.println(bookDao.getBookById(10));
    }
}

```

访问redis集群查看缓存数据；

![](../images/spring boot + vue/redis集群缓存.png)

## 10.0 Spring Boot安全管理

安全可以说是公司的红线了，一般项目都会有严格的认证和授权操作，在Java开发领域常见的安全框架有Shiro和Spring Security。Shiro是一个轻量级的安全管理框架，提供了认证、授权、会话管理、密码管理、缓存管理等功能，Spring Security是一个相对复杂的安全管理框架，功能比Shiro更加强大，权限控制细粒度更高，对OAuth 2的支持也更友好，又因为Spring Security源自Spring家族，因此可以和Spring框架无缝整合，特别是Spring Boot中提供的自动化配置方案，可以让Spring Security的使用更加便捷

### 10.1 Spring Security的基本配置

Spring Boot针对Spring Security提供了自动化配置方案，因此可以使Spring Security非常容易的整合进Spring Boot项目中，这也是Spring Boot项目中使用Spring Security的优势。

#### 10.1.1 基本用法

基本整合步骤如下

1. 创建项目，添加依赖

创建一个Spring Boot Web项目，然后添加spring-boot-starter-security依赖即可

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```

只要添加了spring-boot-starter-security依赖，项目中的所有资源都会被保护起来。

2. 添加接口

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }
}
```

3. 添加静态文件

在resources/路径下创建static路径，新增一张Laugh.jpg文件

4. 启动项目

启动项目，访问/hello或者/Laugh.jpg会跳转到登陆页面，这个登陆页面是由Spring Security提供的

![](../images/spring boot + vue/SpringSecurity登陆页面.png)

默认的用户名是`user`；默认的登陆密码则是每次启动项目时，随机生成的

```java
Using generated security password: e6b58e10-5322-4d1a-b6c0-c532ea85e060
```

登陆成功后就可以访问Laugh.jpg以及/hello

#### 10.1.2 配置用户名和密码

如果对默认的用户名和密码不满意，可以在application.properties中配置默认的用户名、密码以及用户角色

```yaml
spring:
  security:
    user:
      name: Ding
      password: 123456
      roles: admin
```

配置了默认的用户名和密码后，再次启动项目，启动日志中就不会打印出随机生成的密码了，用户可以直接使用配置好的用户名和密码登陆，登陆成功后，用户还具有一个角色——`admin`

#### 10.1.3 基于内存的认证

当然，也可以自定义类继承自WebSecurityConfigurerAdapter，进行实现对Spring Security更多的自定义配置，例如基于内存的认证；

```java
@Configuration
public class MyAccountConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("Kay")
                .password("123456")
                .roles("ADMIN", "USER")
                .and()
                .withUser("Xiu")
                .password("123")
                .roles("USER");
    }
}
```

代码解释：

* 自定义MyAccountConfig继承自WebSecurityConfigurerAdapter，并重写configure（AuthernticationManagerBuilder auth）方法，在该方法中配置两个用户，一个用户名是`Kay`密码`123456`，具备两个角色`ADMIN`和`USER`；另一个用户名是`Xiu`，密码`123`具备一个角色`USER`
* NoOpPasswordEncoder即不对密码进行加密

> 基于内存的用户配置在配置角色时不需要添加`ROLE_`前缀，这点和基于数据库的认证有差别

配置完成后，重启Spring Boot项目，就可以使用这里配置的两个用户进行登陆了

#### 10.1.4 HttpSecurity

受保护的资源都是默认的，而且不能根据实际情况进行角色管理，如果要实现这些功能，就需要重写WebSecurityConfigurerAdapter中的另一个方法

```java
@Configuration
public class MyAccountConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("root")
                .password("123456")
                .roles("ADMIN", "DBA")
                .and()
                .withUser("Kay")
                .password("123456")
                .roles("ADMIN", "USER")
                .and()
                .withUser("Xiu")
                .password("123")
                .roles("USER");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/db/**")
                .access("hasRole('ADMIN') and hasRole('DBA')")
                .antMatchers("/admin/**")
                .hasRole("ADMIN")
                .antMatchers("/user/**")
                .access("hasAnyRole('ADMIN', 'USER')")
                .anyRequest()
                .authenticated()
                .and()
                .formLogin()
                .loginProcessingUrl("/login")
                .permitAll()
                .and()
                .csrf()
                .disable();
    }
}
```

代码解释：

* 配置了三个用户，分别具备不用的角色
* authorizeRequests()方法开启HttpSecurity的配置，分别配置用户访问`/admin/**`模式的URL必须具备`ADMIN`角色；用户访问`/user/**`模式的URL必须具备`ADMIN`或者`USER`的角色；用户访问`/db/**`模式的URL必须具备`ADMIN`和`DBA`的角色
* 用户访问其他的URL都必须认证后访问
* 开启表单登陆，同时配置了登陆接口为`/login`，即可以直接调用`/login`接口，发起一个POST请求进行登陆，登陆参数中用户名必须命名为username，密码必须命名为password，配置loginProcessingUrl接口主要时方便Ajax或者移动端调用登陆接口，最后还配置了permitAll，表示和登陆相关的接口都不需要认证即可访问
* 关闭csrf

配置完成后，新增Controller进行测试

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }
    @GetMapping("/db/hello")
    public String getDBA() {
        return "hello DBA!";
    }
    @GetMapping("/admin/hello")
    public String getAdmin() {
        return "hello Admin!";
    }
    @GetMapping("/user/hello")
    public String getUser() {
        return "hello User!";
    }
}
```

#### 10.1.5 登陆表单详细配置

表单登陆一直使用Spring Security提供的页面，登陆成功后也是默认的页面跳转，但是，前后端分离正在成为企业级应用开发的主流，在前后端分离的开发方式中，前后端的数据交互通过JSON进行，这是，登陆成功后就不是页面跳转了，而是JSON提示，要实现这些功能，只需要完善上文的配置

```java
@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("ding")
                .password("123")
                .roles("ADMIN")
                .and()
                .withUser("xiu")
                .password("123")
                .roles("USER")
                .accountExpired(true)
                .and()
                .withUser("test")
                .password("123")
                .roles("GUEST")
                .accountLocked(true);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/**")
                .access("hasRole('ADMIN')")
                .anyRequest()
                .authenticated()
                .and()
                .formLogin()
                .loginPage("/login_page")
                .loginProcessingUrl("/login")
                .usernameParameter("username")
                .passwordParameter("password")
                .successHandler((HttpServletRequest httpServletRequest,
                                 HttpServletResponse httpServletResponse,
                                 Authentication authentication) -> {
                    Object principal = authentication.getPrincipal();
                    httpServletResponse.setStatus(200);
                    httpServletResponse.setContentType("application/json;charset=utf-8");
                    PrintWriter writer = httpServletResponse.getWriter();
                    Map<String, Object> map = new HashMap<>(2);
                    map.put("status", 200);
                    map.put("msg", principal);
                    ObjectMapper om = new ObjectMapper();
                    writer.write(om.writeValueAsString(map));
                    writer.flush();
                    writer.close();
                })
                .failureHandler((HttpServletRequest httpServletRequest,
                                 HttpServletResponse httpServletResponse,
                                 AuthenticationException e) -> {
                    httpServletResponse.setStatus(200);
                    httpServletResponse.setContentType("application/json; charset=UTF-8");
                    PrintWriter writer = httpServletResponse.getWriter();
                    Map<String, Object> map = new HashMap<>(2);
                    map.put("status", 401);
                    String message = "";
                    if (e instanceof LockedException) {
                        message = "账号被锁定，登陆失败！";
                    } else if (e instanceof BadCredentialsException) {
                        message = "账号名或密码输入错误，登陆失败！";
                    } else if (e instanceof DisabledException) {
                        message = "账户被禁用，登陆失败";
                    } else if (e instanceof AccountExpiredException) {
                        message = "账户已过期，登陆失败！";
                    } else if (e instanceof CredentialsExpiredException) {
                        message = "密码已过期，登陆失败！";
                    } else {
                        message = "登陆失败！";
                    }
                    map.put("msg", message);
                    ObjectMapper om = new ObjectMapper();
                    writer.write(om.writeValueAsString(map));
                    writer.flush();
                    writer.close();
                })
                .permitAll()
                .and()
                .csrf()
                .disable();
    }
}
```

代码解释：

* 配置loginPage，即登陆页面，配置了loginPage后，如果用户未登录授权就访问一个需要授权才能访问的接口，就会自动跳转到`/login_page`页面让用户登陆，这个login_page时自定义的登陆页面，而不是spring security提供的默认登陆页。
* 配置了loginProcessUrl，表示登陆请求处理接口，无论是自定义登陆页面还是移动端登陆，都需要使用该接口。
* 定义了认证所需的用户名和密码的参数，默认用户名参数是`username`密码参数是`password`
* 定义了登陆成功的处理逻辑，用户登陆成功后可以跳转一个页面，也可以返回登陆成功的JSON。这个要看具体业务逻辑，这里使用的是第二种，返回一个登陆成功的JSON；onAuthenticationSuccess方法的第三个参数一般用来获取当前登陆用户的信息，在登陆成功后，可以获取当前登陆用户的信息一起返回给客户端
* 定义了登陆失败的处理逻辑，和登陆成功类似，不同的是，登陆失败的回调方法里有一个AuthenticationException参数，通过这个异常参数可以获取失败的原因，进而给用户一个明确的提示

配置完以后，增加thymeleaf模板依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

创建一个简单的登陆模板以及controller

```java
@RestController
public class HelloController {
    @GetMapping("/admin/hello")
    public String hello() {
        return "Hello ADMIN!";
    }
    @GetMapping("/login_page")
    public ModelAndView login() {
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.setViewName("login");
        return modelAndView;
    }
}
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body>
    <form action="/login" method="post" content="application/json;charset=utf-8">
        <input name="username" placeholder="请输入账号">
        <input name="password" placeholder="请输入密码">
        <button type="submit">登陆</button>
    </form>
</body>
</html>
```

测试登陆，登陆成功返回账号信息；

![](../images/spring boot + vue/SpringSecurity登陆成功.png)

使用过期的账号登陆

![](../images/spring boot + vue/SpringSecurity账号过期.png)

#### 10.1.6 注销登陆配置

配置注销登陆接口

```java
@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("ding")
                .password("123")
                .roles("ADMIN")
                .and()
                .withUser("xiu")
                .password("123")
                .roles("USER")
                .accountExpired(true)
                .and()
                .withUser("test")
                .password("123")
                .roles("GUEST")
                .accountLocked(true);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/**")
                .access("hasRole('ADMIN')")
                .antMatchers("/out", "/favicon.ico")
                .permitAll()
                .anyRequest()
                .authenticated()
                .and()
                .formLogin()
                .loginPage("/login_page")
                .loginProcessingUrl("/login")
                .usernameParameter("username")
                .passwordParameter("password")
                .successHandler((HttpServletRequest httpServletRequest,
                                 HttpServletResponse httpServletResponse,
                                 Authentication authentication) -> {
                    Object principal = authentication.getPrincipal();
                    httpServletResponse.setStatus(200);
                    httpServletResponse.setContentType("application/json;charset=utf-8");
                    PrintWriter writer = httpServletResponse.getWriter();
                    Map<String, Object> map = new HashMap<>(2);
                    map.put("status", 200);
                    map.put("msg", principal);
                    ObjectMapper om = new ObjectMapper();
                    writer.write(om.writeValueAsString(map));
                    writer.flush();
                    writer.close();
                })
                .failureHandler((HttpServletRequest httpServletRequest,
                                 HttpServletResponse httpServletResponse,
                                 AuthenticationException e) -> {
                    httpServletResponse.setStatus(200);
                    httpServletResponse.setContentType("application/json; charset=UTF-8");
                    PrintWriter writer = httpServletResponse.getWriter();
                    Map<String, Object> map = new HashMap<>(2);
                    map.put("status", 401);
                    String message = "";
                    if (e instanceof LockedException) {
                        message = "账号被锁定，登陆失败！";
                    } else if (e instanceof BadCredentialsException) {
                        message = "账号名或密码输入错误，登陆失败！";
                    } else if (e instanceof DisabledException) {
                        message = "账户被禁用，登陆失败";
                    } else if (e instanceof AccountExpiredException) {
                        message = "账户已过期，登陆失败！";
                    } else if (e instanceof CredentialsExpiredException) {
                        message = "密码已过期，登陆失败！";
                    } else {
                        message = "登陆失败！";
                    }
                    map.put("msg", message);
                    ObjectMapper om = new ObjectMapper();
                    writer.write(om.writeValueAsString(map));
                    writer.flush();
                    writer.close();
                })
                .permitAll()
                .and()
                .logout()
                .logoutUrl("/logout")
                .clearAuthentication(true)
                .invalidateHttpSession(true)
                .addLogoutHandler((HttpServletRequest httpServletRequest,
                                   HttpServletResponse httpServletResponse,
                                   Authentication authentication) -> {})
                .logoutSuccessHandler((HttpServletRequest httpServletRequest,
                                       HttpServletResponse httpServletResponse,
                                       Authentication authentication) ->
                    httpServletResponse.sendRedirect("/login_page"))
                .permitAll()
                .and()
                .csrf()
                .disable();
    }
}
```

新建一个简单的登出页面；controller代码如下

```java
@RestController
public class HelloController {
    @GetMapping("/admin/hello")
    public String hello() {
        return "Hello ADMIN!";
    }
    @GetMapping("/login_page")
    public ModelAndView login() {
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.setViewName("login");
        return modelAndView;
    }
    @GetMapping("/out")
    public ModelAndView out() {
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.setViewName("out");
        return modelAndView;
    }
}
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login Out</title>
</head>
<body>
    <form action="/logout" method="post">
        <button type="submit">登出</button>
    </form>
</body>
</html>
```

在antMatchers方法中对`/out`以及自定义`favicon.ico`图标的免授权过滤，在不登陆的情况下可以访问；

代码解释：

* 开启一个注销登陆的配置
* 配置注销登陆请求URL为`/logout`，默认也是`/logout`
* 配置清楚身份认证信息，默认为true，表示清除
* 配置使session失效，默认为true
* 配置一个LogoutHandler，可以在其中完成一些数据清楚工作，例如Cookie的清除，Spring Security提供了一些常见的实现
* LogoutSuccessHandler可以在这里处理注销成功后的业务逻辑，例如返回一段JSON提示或者跳转到登陆页等

![](../images/spring boot + vue/LogoutHandler实现类.png)

#### 10.1.7 多个HttpSecurity

如果业务复杂，可以配置多个HttpSecurity，实现对WebSecurityConfigurerAdapter的多次扩展

```java
@Configuration
public class MySecurityConfig {
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
    @SneakyThrows
    @Autowired
    protected void configure(AuthenticationManagerBuilder authenticationManagerBuilder) {
        authenticationManagerBuilder.inMemoryAuthentication()
                .withUser("admin")
                .password("123")
                .roles("ADMIN")
                .and()
                .withUser("guest")
                .password("123")
                .roles("GUEST");
    }
    @Configuration
    @Order(1)
    public static class AdminSecurityConfig extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests()
                    .antMatchers("/admin/**")
                    .hasRole("ADMIN")
                    .and()
                    .formLogin()
                    .loginProcessingUrl("/login")
                    .permitAll();
        }
    }
    @Configuration
    public static class GuestSecurityConfig extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests()
                    .antMatchers("/guest/**")
                    .permitAll();
        }
    }
}
```

代码解释：

* 配置多个HttpSecurity时，MySecurityConfig不需要继承WebSecurityConfigurerAdapter，在类中创建静态内部类继承WebSecurityConfigurerAdapter即可，静态内部类上添加@Configuration注解和@Order注解，@Order注解表示该配置的优先级，数字越小优先级越大，为加上@Order注解的配置优先级最小；
* 优先处理`/admin/**`模式的URL，其他的URL在其他的HttpSecurity中进行处理
* 基本上每个HttpSecurity都需要`antMatcher`去匹配URL模式（最后一个可不设置），若不设置就不会执行到下面的HttpSecurity配置中去

#### 10.1.8 密码加密

1. 加密方案

密码加密一般会用到散列函数，又称散列算法，哈希函数，这是一种从任何数据中创建数字“指纹”的方法。散列函数把消息或者数据压缩成摘要，使得数据量变小，将数据的格式固定下来，然后将数据打乱混合，重新创建一个散列值。散列值通常用一个短的随机字母和数字组成的字符串来代表。好的散列函数在输入域中很少出现散列冲突，在散列表和数据处理中，不抑制冲突来区别数据会使得数据库记录更难找到。我们常用的散列函数有MD5消息摘要算法、安全散列算法（Secure Hash Algorithm）

2. 实践

在Spring Boot中配置密码加密非常容易，只需要修改上文配置的PasswordEncoder这个Bean的实现即可

```java
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    @SneakyThrows
    @Autowired
    protected void configure(AuthenticationManagerBuilder authenticationManagerBuilder) {
        authenticationManagerBuilder.inMemoryAuthentication()
                .withUser("admin")
                .password("$2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq")
                .roles("ADMIN");
    }
```

创建BCryptPasswordEncoder时传入的参数10就是strength，即密钥的迭代次数（也可以不配置，默认10）。同时，配置的内存用户密码也不再是123了；

#### 10.1.9 方法安全

上文介绍的认证与授权都是基于URL的，开发者也可以通过注解来灵活地配置方法安全，要使用相关注解，首先要通过@EnableGlobalMethodSecurity注解开启基于注解的安全配置

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class MultiSecurityConfig {
//...
}
```

代码解释：

* prePostEnable=true会解锁@PreAuthorize 和 @PostAuthorize两个注解，@PreAuthorize注解会在方法执行前进行验证，而@PostAuthorize注解在方法执行后进行验证
* securedEnable=true会解锁@Secured注解

创建MethodService测试

```java
@Service
public class HelloServiceImpl implements HelloService {
    @PreAuthorize("hasRole('ADMIN')")
    @Override
    public String admin() {
        System.out.println("HelloServiceImpl.admin");
        return "admin Service!";
    }
    @Secured("GUEST")
    @Override
    public String guest() {
        System.out.println("HelloServiceImpl.guest");
        return "guest Service!";
    }
}
```

代码解释：

* @Secured("ROLE_GUEST")注解表示访问该方法需要`GUEST`角色，角色前缀需要加一个`ROLE_`
* @PreAuthorize("hasRole('ADMIN')") 注解表示访问该方法需要ADMIN角色
* @PreAuthorize和@PostAuthorize中都可以使用基于表达式的语法

### 10.2 基于数据库的认证

上文向读者介绍的认证数据都是定义在内存中的，在真实的项目中，用户的基本信息以及角色等都是存储在数据库中，因此需要从数据库中获取数据进行认证

1. 设计数据表

首先需要设计一个基本的用户角色表，一共三张表，分别是用户表、角色表以及用户角色关联表。为了方便测试，预置几条测试数据

![](../images/spring boot + vue/springsecurity数据库设计.png)

```sql
CREATE TABLE user 
(
	id INT(11) PRIMARY KEY,
	username VARCHAR(32) NOT NULL,
	password VARCHAR(255) NOT NULL,
	enabled TINYINT(1) DEFAULT 1,
	locked TINYINT(1) DEFAULT 0
);

CREATE TABLE role 
(
	id INT(11) PRIMARY KEY,
	name VARCHAR(32) DEFAULT '',
	nameZh VARCHAR(32) DEFAULT ''
);

CREATE TABLE user_role 
(
	id INT(11) auto_increment PRIMARY KEY,
	uid INT(11) NOT NULL,
	rid INT(11) NOT NULL
);

INSERT INTO `test`.`user` (id, username, `password`, enabled, locked) VALUES (1, 'root', '$2a$10$Fvy62OC9/9tcNJGYnCpmzOhldIb4Ju2uFUVs410ZM5KUg8xeRn8/u', 1, 0);
INSERT INTO `test`.`user` (id, username, `password`, enabled, locked) VALUES (2, 'admin', '$2a$10$Fvy62OC9/9tcNJGYnCpmzOhldIb4Ju2uFUVs410ZM5KUg8xeRn8/u', 1, 0);
INSERT INTO `test`.`user` (id, username, `password`, enabled, locked) VALUES (3, 'user', '$2a$10$Fvy62OC9/9tcNJGYnCpmzOhldIb4Ju2uFUVs410ZM5KUg8xeRn8/u', 1, 0);

INSERT INTO `test`.`role` (id, `name`, nameZh) VALUES (1, 'ROLE_DBA', '系统管理员');
INSERT INTO `test`.`role` (id, `name`, nameZh) VALUES (2, 'ROLE_ADMIN', '超级管理员');
INSERT INTO `test`.`role` (id, `name`, nameZh) VALUES (3, 'ROLE_user', '普通用户');

INSERT INTO `test`.`user_role` (uid, rid) VALUES (1, 1);
INSERT INTO `test`.`user_role` (uid, rid) VALUES (1, 2);
INSERT INTO `test`.`user_role` (uid, rid) VALUES (2, 2);
INSERT INTO `test`.`user_role` (uid, rid) VALUES (3, 3);
```

> 角色名有一个默认的前缀`ROLE_`

2. 创建项目

Mybatis灵活，JPA便捷，创建项目添加依赖；

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.10</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

3. 数据库配置

在application.properties中添加数据库连接配置

```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql:///test?useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=123456
```

4. 创建实体类

分别创建角色表和用户表对应的实体类，代码如下：

```java
public class User implements UserDetails {
    private static final long serialVersionUID = -3647073428504036390L;
    private Integer id;
    private String username;
    private String password;
    private Boolean enabled;
    private Boolean locked;
    private List<Role> roles;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        for (Role role : roles) {
            authorities.add(new SimpleGrantedAuthority(role.getName()));
        }
        return authorities;
    }

    @Override
    public String getPassword() {
        return this.password;
    }

    @Override
    public String getUsername() {
        return this.username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return !this.locked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return this.enabled;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public void setEnabled(Boolean enabled) {
        this.enabled = enabled;
    }

    public Boolean getLocked() {
        return locked;
    }

    public void setLocked(Boolean locked) {
        this.locked = locked;
    }

    public List<Role> getRoles() {
        return roles;
    }

    public void setRoles(List<Role> roles) {
        this.roles = roles;
    }
}
```

代码解释：

* 用户类实体需要实现UserDetails接口，并实现该接口中的七个方法

| 方法名                    | 解释                             |
| ------------------------- | -------------------------------- |
| getAuthorities()          | 获取当前用户对象所具有的角色信息 |
| getPassword()             | 获取当前用户对象的密码           |
| getUsername()             | 获取当前用户对象的用户名         |
| isAccountNonExpired()     | 当前账户是否未过期               |
| isAccountNonLocked()      | 当前账户是否未锁定               |
| isCredentialsNonExpired() | 当前账户密码是否未过期           |
| isEnabled()               | 当前账户是否可用                 |

* 用户根据实际情况设置这七个方法的返回值，因为默认情况下不需要开发者自己进行密码角色等信息的比对，开发者只需要提供相关信息即可，例如getPassword()方法返回的密码和用户输入的登录密码不匹配，会自动抛出BadCredentialsException异常，isAccountNonExpired()方法返回了false，会自动抛出AccountExpiredException异常，因此对开发者而言，只需要按照数据库中的数据在这里返回相应的配置即可。本案例因为数据库中只有enabled和locked字段，故账户未过期和密码未过期两个方法都返回true
* getAuthorities()方法用来获取当前用户所具有的角色信息，本案例中，用户所具有的角色存储在roles属性中，因此该方法直接遍历roles属性，然后构造SimpleGrantedAuthority集合并返回

> 实现了UserDetails类，故不能使用@Data注解；isxxx 方法等于 getxxx；导致JavaBean里面有两个getxxx方法，违反了JavaBean的规范。

5. 创建UserService

```java
@Service
public class UserService implements UserDetailsService {
    @Autowired
    UserMapper userMapper;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userMapper.loadUserByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("账号不存在!");
        }
        user.setRoles(userMapper.getUserRolesByUid(user.getId()));
        return user;
    }
}
```

代码解释：

* 定义UserService实现UserDetailsService接口，并实现该接口中的loadUserByUsername方法，该方法的参数就是用户登录时输入的用户名，通过用户名去数据库中查找用户，如果没有查找到用户，就抛出一个账户不存在的异常，如果查找到了用户，就继续查找该用户所具有的角色信息，并将获取到的user对象返回，再由系统提供的DaoAuthenticationProvider类去比对密码是否正确
* loadUserBydUsername方法将在用户登录时自动调用

当然这里还涉及UserMapper以及UserMapper.xml

```java
@Mapper
public interface UserMapper {
    User loadUserByUsername(String username);
    List<Role> getUserRolesByUid(Integer id);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.dk.mapper.UserMapper">
    <select id="loadUserByUsername" parameterType="string" resultType="com.dk.entity.User">
        select * from user where username = #{username}
    </select>
    <select id="getUserRolesByUid" parameterType="integer" resultType="com.dk.entity.Role">
        select * from role r, user_role ur where r.id = ur.rid and ur.uid = #{id}
    </select>
</mapper>
```

6. 配置Spring Security

接下来对Spring Security进行配置

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    UserService userService;
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(10);
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/**")
                .hasRole("ADMIN")
                .antMatchers("/dba/**")
                .hasRole("DBA")
                .antMatchers("/user/**")
                .hasRole("user")
                .anyRequest()
                .authenticated()
                .and()
                .formLogin()
                .loginProcessingUrl("/login")
                .permitAll()
                .and()
                .csrf()
                .disable();
    }
}
```

### 10.3 高级配置

#### 10.3.1 角色继承

在10.2节中定义了三个角色，但是这三种角色之间不具备任何关系，一般来说角色之间是有关系的，例如ROLE_admin一般具有admin的权限，又具有user的权限，那么如何配置这种继承关系呢？

在Spring Security中只需要提供一个RoleHierarchy即可，假设ROLE_dba具备所有权限，ROLE_admin具有ROLE_user的权限，ROLE_user则是一个公共角色

```java
@Bean
RoleHierarchy hierarchy() {
    RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
    String hierarchy = "ROLE_DBA > ROLE_ADMIN ROLE_ADMIN > ROLE_USER";
    roleHierarchy.setHierarchy(hierarchy);
    return roleHierarchy;
}
```

配置完RoleHierarchy之后，具有ROLE_DBA角色的用户就可以访问所有资源了，具有ROLE_ADMIN角色的用户也可以访问具有ROLE_USER才能访问的资源

> `String hierarchy = "ROLE_DBA > ROLE_ADMIN ROLE_ADMIN > ROLE_USER"`中的角色区分大小写，需要与数据库中一致；

#### 10.3.2 动态配置权限

使用HttpSecurity配置的认证授权规则还是不够灵活，无法实现资源和角色之间的动态调整，要实现动态配置URL权限，就需要自定义权限配置

1. 数据库设计

这里的数据库在上一节的基础上再增加一张资源表和资源角色管理表，资源表中定义了用户能够访问的URL模式，资源角色表则定义了访问该模式的URL需要什么样的角色。

```sql
CREATE TABLE menu
(
	id INT(11) PRIMARY KEY,
	pattern VARCHAR(255) NOT NULL
);

CREATE TABLE menu_role
(
	id INT(11) AUTO_INCREMENT PRIMARY KEY,
	mid INT(11) NOT NULL,
	rid INT(11) NOT NULL
);

INSERT INTO menu(id, pattern) VALUES(1, "/db/**");
INSERT INTO menu(id, pattern) VALUES(2, "/admin/**");
INSERT INTO menu(id, pattern) VALUES(3, "/user/**");

INSERT INTO menu_role(mid, rid) VALUES(1, 1);
INSERT INTO menu_role(mid, rid) VALUES(2, 2);
INSERT INTO menu_role(mid, rid) VALUES(3, 3);
```

2. 自定义FilterInvocationSecurityMetadataSource

要实现动态配置权限，首先要自定义FilterInvocationSecurityMetadataSource，Spring Security中通过FilterInvocationSecurityMetadataSource接口中的getAttributes方法来确定一个请求需要哪些角色，FilterInvocationSecurityMetadataSource接口中的默认实现类是DefaultFilterInvocationSecurityMetadataSource，参考其我们可以实现自己的FilterInvocationSecurityMetadataSource

```java
@Component
public class CustomFilterInvocationSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {
    AntPathMatcher antPathMatcher = new AntPathMatcher();
    @Autowired
    MenuMapper menuMapper;
    @Override
    public Collection<ConfigAttribute> getAttributes(Object o) throws IllegalArgumentException {
        String request = ((FilterInvocation) o).getRequestUrl();
        List<Menu> menus = menuMapper.getAllMenus();
        for (Menu menu : menus) {
            if (antPathMatcher.match(menu.getPattern(), request)) {
                List<Role> roles = menu.getRoles();
                String [] roleArr = new String[roles.size()];
                for (int i = 0; i < roles.size(); i++) {
                    roleArr[i] = roles.get(i).getName();
                }
                return SecurityConfig.createList(roleArr);
            }
        }
        return SecurityConfig.createList("ROLE_LOGIN");
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return null;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return FilterInvocation.class.isAssignableFrom(aClass);
    }
}
```

代码解释：

* 开发者自定义FilterInvocationSecurityMetadataSource主要实现该接口中的getAttributes方法，该方法的参数是一个FilterInvocation，开发者可以从FilterInvocation中提取出当前请求的URL，返回值是个Collection<ConfigAttribute>，表示当前请求URL所需的角色
* 创建一个antPathMatcher，主要用来实现ant风格的URL匹配
* 从参数中提取出所有资源信息，即本案例中的menu表以及menu所对应的role，在真实项目环境中，开发者可以将该资源信息缓存在Redis或者其它缓存数数据库中
* 遍历资源信息，遍历过程中，获取当前请求的URL所需要的角色信息并返回，如果当前请求的URL在资源表中不存在相应的模式，说明该请求登录后即可访问，即直接返回`ROLE_LOGIN`
* getAllConfigAttribute方法用来返回所有定义好的权限资源，Spring Security在启动时会校验相关配置是否正确，如果不需要校验，那么该方法直接返回`null`即可
* supports方法返回类对象是否支持校验

3. 自定义AccessDecisionManager

当一个请求走完FilterInvocationSecurityMetadataSource中的getAttributes方法后，接下来就会来到AccessDecisionManger类中进行角色信息的比对

```java
@Component
public class CustomAccessDecisionManager implements AccessDecisionManager {
    @Autowired
    RoleHierarchy roleHierarchy;
    @Override
    public void decide(Authentication authentication, Object o, Collection<ConfigAttribute> collection)
            throws AccessDeniedException, InsufficientAuthenticationException {
        Collection<? extends GrantedAuthority> grantedAuthorities =
                roleHierarchy.getReachableGrantedAuthorities(authentication.getAuthorities());
        for (ConfigAttribute configAttribute : collection) {
            if ("ROLE_LOGIN".equals(configAttribute.getAttribute())
            && authentication instanceof UsernamePasswordAuthenticationToken) {
                return;
            }
            for (GrantedAuthority grantedAuthority : grantedAuthorities) {
                if (configAttribute.getAttribute().equals(grantedAuthority.getAuthority())) {
                    return;
                }
            }
        }
        throw new AccessDeniedException("没有访问权限!");
    }

    @Override
    public boolean supports(ConfigAttribute configAttribute) {
        return true;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return true;
    }
}
```

代码解释：

* 自定义AccessDecisionManager并重写decide方法，在该方法中判断当前登陆的用户是否具备当前请求URL所需的角色信息，如果不具备，就抛出AccessDeniedException异常，否则不做任何处理
* decide方法有三个参数，第一个参数包含当前登陆用户的信息；第二个参数则是一个Filter Invocation对象，可以获取当前请求对象等；第三个参数就是FilterInvocationSecurityMetadataSource中的getAttributes方法的返回值，即当前请求URL所需要的角色
* 进行角色对比，如果需要的角色是ROLE_LOGIN，说明当前请求的URL用户登陆后即可访问，如果authentication是UsernamePasswordAuthenticationToken的实例，那么说明当前用户已登陆，该方法到此结束，否则进入正常的判断流程，如果当前用户具备当前请求需要的角色，那么方法结束

涉及到的Mapper、Mapper.xml、以及实体类

```java
@Mapper
public interface MenuMapper {
    List<Menu> getAllMenus();
}
```

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.dk.mapper.MenuMapper">
    <resultMap id="BaseResultMap" type="com.dk.entity.Menu">
        <id property="id" column="id"/>
        <result property="pattern" column="pattern"/>
        <collection property="roles" ofType="com.dk.entity.Role">
            <id property="id" column="rid"/>
            <result property="name" column="rname"/>
            <result property="nameZh" column="rnameZh"/>
        </collection>
    </resultMap>
    <select id="getAllMenus" resultMap="BaseResultMap">
        select m.*, r.id as rid, r.name as rname, r.nameZh as rnameZh from menu m left join
            menu_role mr on m.`id` = mr.`mid` left join role r on mr.`rid` = r.`id`
    </select>
</mapper>
```

```java
@Data
public class Menu {
    private Integer id;
    private String pattern;
    private List<Role> roles;
}
```

4. 配置

最后在SecurityConfig中配置如下

```java
@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    UserServiceImpl userService;
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(10);
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService);
    }
    @Bean
    RoleHierarchy roleHierarchy() {
        RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
        String hierarchy = "ROLE_DBA > ROLE_ADMIN ROLE_ADMIN > ROLE_USER";
        roleHierarchy.setHierarchy(hierarchy);
        return roleHierarchy;
    }
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest()
                .authenticated()
                .withObjectPostProcessor(
                        new ObjectPostProcessor<FilterSecurityInterceptor>() {
                            @Override
                            public <O extends FilterSecurityInterceptor> O postProcess(O o) {
                                o.setSecurityMetadataSource(cfisms());
                                o.setAccessDecisionManager(cadm());
                                return o;
                            }
                        }
                )
                .and()
                .formLogin()
                .loginProcessingUrl("/login")
                .and()
                .csrf()
                .disable();

    }
    @Bean
    CustomFilterInvocationSecurityMetadataSource cfisms() {
        return new CustomFilterInvocationSecurityMetadataSource();
    }
    @Bean
    CustomAccessDecisionManager cadm() {
        return new CustomAccessDecisionManager();
    }
}
```

### 10.4 OAuth 2

#### 10.4.1 OAuth 2 简介

OAuth是一个开放标准，该标准允许用户让第三方应用访问该用户在某一网站上存储的私密资源（如头像、照片、视频等），而在这个过程中无须将用户名和密码提供给第三方应用，实现这个功能是通过一个令牌（token），而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的网站在特定的时段访问特定的资源。这样，OAuth让用户可以授权第三方网站灵活的访问存储在另外一些资源服务器的特定信息，而非所有内容。

采用令牌的方式可以让用户灵活地对第三方应用授权或者收回权限。OAuth2是OAuth协议的下一个版本，但不向下兼容OAuth 1.0。OAuth2关注客户端开发者的简易性，同时为Web应用、桌面应用、移动设备等提供专门的认证流程。传统的Web开发登陆认证一般都是基于Session的，但是在前后端分离的架构中继续使用Session会有许多不便，因为移动端（Android、IOS、微信小程序）要么不支持Cookie（微信小程序），要么使用非常不方便，对于这些问题，使用OAuth2认证都能解决

#### 10.4.2 OAuth 2 角色

要了解OAuth 2，需要先了解OAuth 2 中几个基本的角色

* 资源所有者：资源所有者即用户，具有头像、照片、视频等资源。
* 客户端：客户端即第三方应用
* 授权服务器：授权服务器用来验证用户提供的信息是否正确，并返回一个令牌给第三方应用。
* 资源服务器：资源服务器是提供给用户资源的服务器

一般来说，授权服务器和资源服务器可以是同一台服务器

#### 10.4.3 OAuth 2 授权流程

OAuth 2 的授权流程到底是什么样的呢？

具体步骤如下：

> 步骤一：客户端（第三方应用）向用户请求授权
>
> 步骤二：用户单击客户端所呈现的服务授权页面上的同意按钮后，服务端返回一个授权许可凭证给客户端
>
> 步骤三：客户端拿着授权许可凭证去授权服务器申请令牌
>
> 步骤四：授权服务器验证信息无误后，发放令牌给客户端
>
> 步骤五：客户端拿着令牌去资源服务器访问资源
>
> 步骤六：资源服务器验证令牌无误后开放资源

这是一个大致的流程，因为OAuth 2中有四种不同的授权模式，每种授权模式的授权流程又会有差异，基本流程如下：

![](../images/spring boot + vue/OAuth授权基本流程.png)

#### 10.4.4 授权模式

OAuth协议的授权模式分为四种，分别如下：

* 授权码模式：授权码模式（Authentication code）是功能最完整、流程最严谨的授权模式。它的特点就是通过客户端的服务器与授权服务器进行交互，国内最常见的第三方平台登陆功能基本都是使用这种模式
* 简化模式：简化模式不需要客户端服务器参与，直接在浏览器中向授权服务器申请令牌，一般若网站是纯静态页面，则可以采用这种方式
* 密码模式：密码模式是用户把用户名密码直接告诉客户端，客户端使用这些信息向授权服务器申请令牌，这需要用户对客户端高度信任，例如客户端应用和服务提供商是同一家公司
* 客户端模式：客户端模式是指客户端使用自己的名义而不是用户的名义向服务器提供者申请授权，严格来说，客户端模式并不能算作OAuth协议要解决的问题的一种解决方案，但是，对于开发者而言，在一些前后端分离应用或者为移动端提供的认证授权服务器上使用这种模式还是非常方便的

这四种模式各有千秋，分别适用于不同的开发场景，开发者根据实际情况进行选择。

#### 10.4.5 实践

本案例要介绍的是在前后端分离应用提供的认证服务器中如何快速搭建OAuth服务，因此主要介绍密码模式

1. 创建项目，添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
        <exclusions>
            <exclusion>
                <groupId>io.lettuce</groupId>
                <artifactId>lettuce-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security.oauth</groupId>
        <artifactId>spring-security-oauth2</artifactId>
        <version>2.3.3.RELEASE</version>
    </dependency>
</dependencies>
```

由于Spring Boot中的OAuth协议是在Spring Security的基础上完成的，因此首先要添加Spring Security的依赖，要用到OAuth 2，因此添加OAuth 2相关依赖，令牌可以存储在Redis缓存服务器上，同时Redis具有过期等功能，很适合令牌的存储，因此也加入Redis依赖。

项目创建后，接下来在Application.properties中配置一下Redis服务器的连接信息，代码如下：

```properties
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-idle=8
spring.redis.jedis.pool.max-wait=-1ms
spring.redis.jedis.pool.min-idle=0
spring.redis.password=123456
spring.redis.port=11080
spring.redis.host=127.0.0.1
spring.redis.database=0
```

2. 配置授权服务器

授权服务器和资源服务器可以是同一台服务器，也可以是不同服务器，本案例中假设是同一台服务器，通过不同的配置分开开启授权服务器和资源服务器，首先是授权服务器；

```java
@Configuration
@EnableAuthorizationServer
public class AuthenticationServerManager extends AuthorizationServerConfigurerAdapter {
    @Autowired
    AuthenticationManager authenticationManager;
    @Autowired
    UserDetailsService userDetailsService;
    @Autowired
    RedisConnectionFactory redisConnectionFactory;
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(10);
    }
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("password")
                .authorizedGrantTypes("password", "refresh_token")
                .accessTokenValiditySeconds(1800)
                .resourceIds("rid")
                .scopes("all")
                .secret("$2a$10$Fvy62OC9/9tcNJGYnCpmzOhldIb4Ju2uFUVs410ZM5KUg8xeRn8/u");
    }
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(new RedisTokenStore(redisConnectionFactory))
                .authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService);
    }
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.allowFormAuthenticationForClients();
    }
}
```

代码解释：

* 自定义类继承自AuthorizationServerConfigurerAdapter，完成对授权服务器的配置，然后通过@EnableAuthorizationServer注解开启授权服务器
* 注入了AuthenticationManager，该对象将用来支持password模式
* 注入了RedisConnectionFactory，该对象用来完成Redis缓存，将令牌信息存储到Redis缓存中
* 注入了UserDetailsService，该对象将为刷新token提供支持
* 配置了password授权模式，authorizedGrantTypes表示OAuth 2中的授权模式为password和refresh_token两种，在标准的OAuth 2 中，授权模式并不包括refresh_token，但是在Spring Security的实现中将其归为一种，因此如果需要实现access_token的刷新，就需要添加这样一种授权模式，accessTokenValiditySeconds方法配置了access_token的过期时间，resourceIds配置了资源id，secret方法配置了加密后的密码，明文是123456
* 配置了令牌的存储，AuthenticationManager和UserDetailsService主要用于支持password模式以及令牌的刷新
* 最后的配置表示支持client_id和client_secret做登陆认证

3. 配置资源服务器

配置资源服务器代码如下

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.resourceId("rid")
                .stateless(true);
    }
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/**")
                .hasRole("admin")
                .antMatchers("/user/**")
                .hasRole("user")
                .anyRequest()
                .authenticated();
    }
}
```

代码解释：

* 自定义类继承自ResourceServerConfigurerAdapter，并添加@EnableResourceServer注解开启资源服务器配置
* 配置资源id，这里的资源id和授权服务器中的资源id一致，然后设置这些资源仅基于令牌认证

4. 配置Security

```java
@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        return super.userDetailsService();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("admin")
                .password("$2a$10$Fvy62OC9/9tcNJGYnCpmzOhldIb4Ju2uFUVs410ZM5KUg8xeRn8/u")
                .roles("admin");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/oauth/**")
                .authorizeRequests()
                .antMatchers("/oauth/**")
                .permitAll()
                .and()
                .csrf()
                .disable();
    }
}
```

这里Spring Security的配置基本与前文一致，唯一不同的是多了两个Bean，这里两个Bean将注入授权服务器配置类中使用。另外，这里的HttpSecurity配置主要是`/oauth/**`模式的URL，这一类的请求直接放行。在Spring Security配置和资源服务器配置中，一共涉及两个HttpSecurity，其中Spring Security中的配置优先级高于资源服务器中的配置，即请求地址先经过Spring Security的HttpSecurity，再经过资源服务器的HttpSecurity

5. 测试验证

首先创建三个简单的请求地址，代码如下：

```java
@RestController
public class HelloController {
    @GetMapping("/admin/hello")
    public String admin() {
        return "Hello! admin";
    }
    @GetMapping("/user/hello")
    public String user() {
        return "Hello! user";
    }
    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }
}
```

根据前文的配置，要请求这三个地址，分别需要admin角色、user角色以及登陆后访问。

所有都配置完成后，启动Redis服务器，再启动Spring Boot项目，首先发送一个POST请求获取token，请求地址如下

```properties
请求URL：
http://127.0.0.1:8080/oauth/token
请求类型：
POST
请求参数：
username:admin
password:123456
grant_type:password
client_id:password
scope:all
client_secret:123456
返回结果值：
{
    "access_token": "dbec9a68-19d9-4ecd-a4bf-6bd5341804fa",
    "token_type": "bearer",
    "refresh_token": "68f3f9b0-dcca-4c2b-b2d0-90da3052e639",
    "expires_in": 1799,
    "scope": "all"
}
```

请求地址中包含的参数有用户名、密码、授权模式、客户端id、scope以及客户端密码，基本就是授权服务器中所配置的数据

返回结果有access_token、token_type、refresh_token、expires_in以及scope，其中access_token是获取其他资源时要用的令牌，refresh_token用来刷新令牌，expires_in表示access_token的过期时间，当access_token过期后，使用refresh_token来重新获取新的access_token（前提是refresh_token未过期），请求地址如下

```properties
请求URL：
http://127.0.0.1:8080/oauth/token
请求类型：
POST
请求参数：
grant_type:refresh_token
refresh_token:09e0f834-847e-4387-af7e-11c61dd03bf4
client_id:password
client_secret:123456
返回结果值：
{
    "access_token": "0df6941c-8e66-4f1d-995c-47641d5f9917",
    "token_type": "bearer",
    "refresh_token": "09e0f834-847e-4387-af7e-11c61dd03bf4",
    "expires_in": 1799,
    "scope": "all"
}
```

获取新的access_token时需要携带上refresh_token，同时授权模式设置为refresh_token，在获取的结果中access_token会变化，同时access_token有效期也会变化

接下来访问所有资源，携带上access_token参数即可，例如`/user/hello`接口

![](../images/spring boot + vue/OAuth授权-测试-1.png)

如果访问一个非法资源，例如admin用户访问`/user/hello`接口

![](../images/spring boot + vue/OAuth授权-测试-2.png)

最后，再来看一下Redis中的数据

![](../images/spring boot + vue/OAuth授权-Redis缓存accessToken.png)

到此，一个password模式的OAuth认证体系就搭建成功了。

OAuth中的认证模式有4种，需要结合自己开发的实际情况选择其中一种，本案例介绍的时在前后端分离应用种常用的password模式，其他的授权模式也都有自己的使用场景。

### 10.5 Spring Boot 整合 Shiro

#### 10.5.1 Shiro简介

Apache Shiro是一个开源的轻量级的Java安全框架，它提供身份认证、授权、密码管理以及会话管理等功能。相对于Spring Security，Shiro框架更加直观、易用，同时页能提供健壮的安全性。

在传统的SSM框架中，手动整合Shiro的步骤还是比较多的，针对Spring Boot，Shiro官方提供了shiro-spring-boot-web-starter用来简化Shiro在Spring Boot中的配置。

#### 10.5.2 整合Shiro

1. 创建项目

首先创建一个普通的Spring Boot项目，添加Shiro依赖以及页面模板依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring-boot-web-starter</artifactId>
        <version>1.4.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>com.github.theborakompanioni</groupId>
        <artifactId>thymeleaf-extras-shiro</artifactId>
        <version>2.0.0</version>
    </dependency>
</dependencies>
```

2. Shiro基本配置

首先在application.yml中配置Shiro的基本信息

```yaml
shiro:
  enabled: true
  web:
    enabled: true
  loginUrl: /login
  successUrl: /index
  unauthorizedUrl: /unauthorized
  sessionManager:
    sessionIdUrlRewritingEnabled: true
    sessionIdCookieEnabled: true
```

代码解释：

* 配置一表示开启Shiro配置，默认为true
* 配置二表示开启Shiro Web配置，默认为true
* 配置三表示登陆地址，默认为"/login.jsp"
* 配置四表示登陆成功地址，默认为"/"
* 配置五表示未获授权默认跳转地址
* 配置六表示是否开启通过url参数实现会话跟踪，如果网站支持Cookie，可以关闭此选项，默认为true
* 配置七表示是否允许通过Cookie实现会话跟踪，默认为true

基本信息配置完毕后，接下来在Java代码中配置Shiro，提供两个最基本的Bean即可

```java
@Configuration
public class ShiroConfig {
    @Bean
    public Realm realm() {
        TextConfigurationRealm realm = new TextConfigurationRealm();
        realm.setUserDefinitions("ding=123,user\n admin=123,admin");
        realm.setRoleDefinitions("admin=read,write\n user=read");
        return  realm;
    }

    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition definition = new DefaultShiroFilterChainDefinition();
        definition.addPathDefinition("/login", "anon");
        definition.addPathDefinition("/doLogin", "anon");
        definition.addPathDefinition("/logout", "logout");
        definition.addPathDefinition("/**", "authc");
        return definition;
    }

    @Bean
    public ShiroDialect shiroDialect() {
        return new ShiroDialect();
    }
}
```

代码解释：

* 这里提供两个关键Bean，一个是Realm，另一个是ShiroFilterChainDefinition。至于ShiroDialect，则是为了支持在Thymeleaf中使用Shiro标签，如果不在Thymeleaf中使用Shiro标签，那么可以不提供ShiroDialect
* Realm可以是自定义Realm，也可以是Shiro提供的Realm，这里没有配置数据库连接，直接配置了两个用户；分别对应角色admin以及user，user有read权限，admin则有read和write权限
* ShiroFilterChainDefinition Bean中配置了基本的过滤规则，"/login"和"/doLogin"可以匿名访问，"logout"是一个注销登陆请求，其余请求则都需要认证后才能访问

接下来配置登陆接口以及页面访问接口

```java
@Controller
public class UserController {
    @PostMapping("/doLogin")
    public String doLogin(String username, String password, Model model) {
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(token);
        } catch (AuthenticationException e) {
            System.out.println("UserController ## login error");
            model.addAttribute("error", "用户名或密码错误！");
            return "login";
        }
        return "redirect:/index";
    }

    @RequiresRoles("admin")
    @GetMapping("/admin")
    public String admin() {
        return "admin";
    }

    @RequiresRoles(value = {"admin", "user"}, logical = Logical.OR)
    @GetMapping("/user")
    public String user() {
        return "user";
    }
}
```

代码解释：

* 在doLogin中，首先构造一个UsernamePasswordToken实例，然后获取一个Subject对象，并调用对象中的login方法执行登陆操作，在登陆操作执行中，当有异常抛出时，说明登陆失败，携带错误信息返回登陆视图；当登陆成功时，则重定向到index页面
* 接下来暴露两个接口，"/admin"和"/user"，对于"/admin"接口，需要具有admin角色才可以访问；对于"/user"接口，具备admin角色或者user角色其中任意一个即可访问。

对于其他不需要角色就能访问的接口，直接在WebMvc中配置即可

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/login").setViewName("login");
        registry.addViewController("/index").setViewName("index");
        registry.addViewController("/unauthorized").setViewName("unauthorized");
    }
}
```

接下来创建全局异常处理器进行全局异常处理，本案例中主要是处理授权异常

```java
@ControllerAdvice
public class ExceptionController {
    @ExceptionHandler(AuthorizationException.class)
    public ModelAndView error(AuthorizationException e) {
        System.out.println("ExceptionController ## catch AuthenticationException ");
        ModelAndView modelAndView = new ModelAndView("unauthorized");
        modelAndView.addObject("error", e.getMessage());
        return modelAndView;
    }
}
```

当用户访问未授权的资源时，跳转到unauthorized视图中，并携带出错信息

配置完成后，最后在`resource/templates`目录下创建五个HTML页面进行测试

1. index.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
<head>
    <meta charset="UTF-8">
    <title>Index</title>
</head>
<body>
    <h3>Hello,<shiro:principal/></h3>
    <h3><a href="/logout">注销登陆</a></h3>
    <h3><a shiro:hasRole="admin" href="/admin">管理员页面</a></h3>
    <h3><a shiro:hasAnyRole="admin,user" href="/user">普通用户页面</a></h3>
</body>
</html>
```

index.html是登陆成功后跳转的页面，首先展示当前用户的用户名，然后展示"注销登陆"的🔗链接，若当前用户具备admin角色，则展示一个"管理员页面"的超链接，若用户具备user角色或者admin角色，则展示一个"普通用户页面"的超链接，注意这里导入的名称空间是 `xmlns:shiro="http://www.pollix.at/thymeleaf/shiro"`和以下页面中导入的Shiro名称空间不一致

2. login.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body>
    <div>
        <form action="/doLogin" method="post">
            用户名：<input type="text" name="username"><br/>
            密  码：<input type="password" name="password"><br/>
            <div th:text="${error}"/>
            <input type="submit" value="登陆">
        </form>
    </div>
</body>
</html>
```

login页面是一个普通的登陆页面，在登陆失败时，展示登陆失败的错误信息

3. user.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>User</title>
</head>
<body>
    <h1>普通用户页面</h1>
</body>
</html>
```

4. admin.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Admin</title>
</head>
<body>
    <h1>管理员页面</h1>
</body>
</html>
```

5. unauthorized.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>Unauthorized</title>
</head>
<body>
    <div>
        <h3>未授权，非法访问</h3>
        <h3 th:text="${error}"></h3>
    </div>
</body>
</html>
```

配置完成后，启动项目，访问登陆页面，分别使用两个账号登陆；

![](../images/spring boot + vue/Shiro-Ding.png)

![](../images/spring boot + vue/Shiro-Admin.png)

当user用户访问admin的接口时

![](../images/spring boot + vue/Shiro-unauthorized.png)

### 10.6 小结

本章节介绍了Spring Security以及Shiro在Spring Boot中的使用，对于Spring Security，有基于传统方式的Session认证，也有使用OAuth协议的认证，一般来说，在传统的Web框架中，使用Session认证方便快捷，但是，若结合微服务、前后端分离等架构，则使用OAuth认证更加方便，具体使用哪一种，需要根据实际情况进行取舍。对于Shiro，虽然功能不及Spring Security强大，但是简单易用，而且也能胜任大部分中小型项目，当然在Spring Boot项目中，Spring Security的整合显然更加容易，因此可以首选Spring Security。

## 11.0 Spring Boot整合WebStocket

### 11.1 为什么需要WebSocket

在HTTP协议中，所有的请求都是由客户端发起的，由服务端进行响应，服务端无法向客户端推送消息，但是在一些需要即时通讯的应用中，又不可避免地需要服务端向客户端推送消息，传统的解决方案主要有如下几种

1. **轮询**

轮询是最简单的一种解决方案，所谓轮询，就说客户端在固定的时间间隔下不停地向服务端发送请求，查看服务端是否有最新数据，若服务端有最新数据，则返回给客户端，若服务端没有，则返回一个空的JSON或者XML文档。轮询对开发人员而言实现方便，但是弊端也很明显：客户端每次都要新建HTTP请求，服务端需要处理大量的无效请求，在高并发的情况下会严重拖慢服务端的运行效率，同时服务端的资源被极大的浪费了，因此这种方式并不可取。

2. **长轮询**

长轮询是传统轮询的升级版，不同于传统轮询，在长轮询中，服务端不是每次都会立即响应客户端的请求，只有在服务端有最新数据的时候才会立即响应客户端的请求，否则服务端会持有这个请求而不返回，知道有新数据时才返回。这种方式可以在一定程度上节省网络资源和服务器资源，但是也存在一些问题，例如：

* 如果浏览器在服务器响应之前有了新数据要发送，就只能创建一个新的并发请求，或者先尝试断掉当前请求，再创建新的请求。
* TCP和HTTP规范中都有链接超时一说，所以所谓的长轮询并不能一直链接，服务端和客户端的连接需要定期的连接和关闭再连接，这增大了程序员的工作量，当然也有一些技术能够延长每次连接的时间，但毕竟不是主流方案。

3. Applet和Flash

Applet和Flash都已经是明日黄花，不过在这两项技术存在的岁月里，除了可以让我们的HTML页面更加绚丽之外，还可以解决消息推送的问题，开发者可以使用Applet和Flash来模拟全双工通信，通过创建一个只有一个像素点大小的透明的Applet或者Flash，然后将之镶嵌在网页中，再从Applet或者Flash的代码中建立一个Socket连接进行双向通信。这种连接方式消除了HTTP协议中的诸多限制，当服务器有消息发送到客户端的时候，开发者可以在Applet或者Flash中调用JavaScript函数将数据显示在页面上，当浏览器有数据要发送给服务器时也一样，通过Applet或者Flash来传递。这种方式真正地实现了全双工通信，不过也有问题，说明如下：

* 浏览器必须能够运行Java或者Flash
* 无论是Applet还是Flash都存在安全问题
* 随着HTML5标准被各浏览器厂商广泛支持，Flash下架已经被提上日程（Adobe宣布2020年正式停止支持Flash）

其实，传统的解决方案不止这三种，但是无论哪种解决方案都是有自身的缺陷，于是有了WebSocket

### 11.2 WebSocket简介

WebSocket是一种在单个TCP连接上进行全双工通信的协议，已被W3C定为标准。使用WebSocket可以使得客户端和服务器之间的数据交换变得更简单，它允许服务端主动向客户端推送数据。在WebSocket协议中，浏览器和服务器只需要完成一次握手，两者之间就可以直接创建持久性的连接，并进行双向数据传输。

WebSocket使用了HTTP/1.1的协议升级特性，一个WebSocket请求首先使用非正常的HTTP请求以特定的模式访问一个URL，这个URL有两种模式，分别是`ws`和`wss`，对应HTTP协议中的HTTP和HTTPS，在请求头中有一个Connection：Upgrade字段，表示客户端想要对协议进行升级，另外还有一个Upgrade：WebSocket字段，表示客户端想要将请求协议升级为WebSocket协议。这两个字段共同告诉服务器要将连接升级为WebSocket这样一种全双工协议，如果服务端同意协议升级，那么在握手完成后，文本消息或者其他二进制消息就可以同时在两个方向上进行发送，而不需要关闭和重新连接。此时的客户端和服务端关系是对等的，他们可以互相向对方主动发消息，和传统的解决方案相比，WebSocket主要有如下特点：

* WebSocket使用时需要先创建连接，这使得WebSocket成为一种有状态的协议，在之后的通信过程中可以省略部分状态信息（例如身份认证等）
* WebSocket连接在端口80（ws）或者443（wss）上创建，与HTTP使用端口相同，这样，基本所有的防火墙都不会阻止WebSocket连接。
* WebSocket使用HTTP协议进行握手，因此它可以自然而然地集成到网络浏览器和HTTP服务器中，而不需要额外的成本
* 心跳消息（ping和pong）将被反复的发送，进而保持WebSocket连接一直处于活跃状态
* 使用该协议，当消息启动或者到达的时候，服务端和客户端都可以知道
* WebSocket连接关闭时将发送一个特殊的关闭消息
* WebSocket支持跨域，可以避免Ajax的限制
* HTTP规范要求浏览器将并发连接数限制为每个主机名两个连接，但是当我们使用WebSocket的时候，当握手完成之后，该限制就不存在了，因此此时的连接已经不再是HTTP连接了
* WebSocket协议支持扩展，用户可以扩展协议，实现部分自定义的子协议
* 更好的二进制支持以及更好的压缩效果

WebSocket既然具有这么多优势，当然使用场景也是非常广泛的，例如：

* 在线股票网站
* 即时聊天
* 多人在线游戏
* 应用集群通信
* 系统性能实时监控

### 11.3 Spring Boot整合WebSocket

Spring Boot对WebSocket提供了非常友好的支持，可以方便开发者在项目中快速集成WebSocket功能，实现单聊或者群聊

#### 11.3.1 消息群发

1. **创建项目**

