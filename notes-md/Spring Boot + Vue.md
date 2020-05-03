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

