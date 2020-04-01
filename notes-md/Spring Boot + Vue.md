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



