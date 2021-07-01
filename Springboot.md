# 一、基础入门

## 01 基本使用

> 文档及教程地址

https://spring.io/

https://www.yuque.com/atguigu/springboot/rmxq85



### 步骤

#### 1.1 创建maven工程

#### 1.2 引入依赖

```xml
<!-- 基本的依赖引入 -->
<!-- 父项目 -->
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.2</version>
</parent>

<!-- web依赖 -->
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
</dependencies>
```

#### 1.3 编写主程序

```java
/**
* 主程序类 MainApplication
* @SpringBootApplication，SpringBoot应用
**/
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
```

#### 1.4 编写业务

```java
//@ResponseBody
//@Controller
@RestController
public class helloController {
    @RequestMapping("/hello")
    public String handle01(){
        return "Hello, SpringBoot 2";
    } 
}
```

#### 1.5 简化配置

```application.properties```

```xml
<!-- 通过application.properties文件配置 -->
<!-- 更改端口号 -->
server.port=8888
```

#### 1.6 简化部署

```xml
<!-- pom.xml下jar打包 -->
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
</build>
```



## 02 自动配置

### 2.1 依赖管理

+ ==父项目==做依赖管理

```xml
<parent>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-parent</artifactId>
     <version>2.4.2</version>
</parent>

<!-- 他的父项目 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.4.2</version>
</parent>
<!-- 其中声明了几乎所有开发中常用的依赖的版本号--> 
```

+ 开发只需导入==starter==场景启动器

```xml
<!-- 1、spring-boot-starter-*: *为某种场景 -->
<!-- 2、只要引入starter，场景所需的依赖就自动引入 -->
<!-- 3、*-spring-boot-starter：第三方-->
<!-- 4、所有场景启动器最底层的依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.4.2</version>
    <scope>compile</scope>
</dependency>
```

+ ==无需关注版本号==，自动版本仲裁

```xml
<!-- 1、引入依赖默认不用写版本号 -->
<!-- 3、引入非版本仲裁的jar需要写版本号 -->
```

+ 可以修改版本号

```xml
<!-- 查看spring-boot-dependencies中规定当前依赖的版本 -->
<!-- 在pom.xml中设置，Maven就近优先原则 -->
<properties>
	<mysql.version>5.1.43</mysql.version>
</properties>
```



### 2.2 自动配置

+ 自动配置Tomcat
  + 引入Tomcat依赖
  + 配置Tomcat

```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-tomcat</artifactId>
     <version>2.4.2</version>
     <scope>compile</scope>
</dependency>
```

+ 自动配置SpringMVC
  + 引入了SpringMVC全套组件
  + 自动配置了SpringMVC常用组件
+ 自动配置web常见功能，如：CharacterEncoding
  + Springboot配置了所有web开发的常见场景
  + 无需配置如Spring的包扫描
  + 改变扫描路径
    + ```SpringBootApplication(scanBasePackages="com.maziyao")```

```java
@SpringBootApplication(scanBasePackages="com.maziyao")
//等同于
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.maziyao")
```

+ 默认的包结构
  + ==主程序所在的包==及其所有==子包==的组件都会默认扫描

+ 各种配置拥有默认值
  + 默认配置最终都是映射到MultipartProperties
  + 配置文件的值最终会绑定每个类上，这个类会在容器中创建对象

+ 按需加载所有自动配置项
  + SpringBoot自动配置功能在spring-boot-autoconfigure包中



## 03 容器功能



### 3.1 组件添加

#### @Configuration

+ 基本使用
  + 默认使用@Configuration即CGlib代理，对象实例都为单例对象

```java
//包config中创建类MyConfig
package com.maziyao.boot.config;

//假设bean中有User和Cat类，使用@Configuration注解
//@Configuration即告诉SpringBoot：MyConfig是一个配置类
//等同于Spring的xml配置文件
@Configuration
public class MyConfig{
    /**
    *@Bean给组件添加容器，以方法名作为组件id名，可自定义
    *返回类型就是组件类型，返回值就是组件在容器中的对象实例，默认为单例
    **/
    @Bean
    public User user01(){
        return new User("Zhangsan",18);
    }
    @Bean
    public Cat cat01(){
        return new Cat("Tomcat");
    }
}
```



+ Full模式和Lite模式

  + Full：==@Configuration(proxyBeanMethods = true)==

  ```java
  //配置类中的组件之间有依赖关系时，使用Full模式
  //每次SpringBoot都会检查这个组件是否在容器中，保持组件单实例
  //如：User类依赖Cat类
  public class User{
      private String name;
      private Integer age;
      private Cat cat;
      // ...
  }
  //此时user01()中调用cat01()成立，实例对象为单例，且为动态代理模式
  //zhangsan.getCat()结果和cat是相同的
  @Configuration(proxyBeanMethods = true)
  public class MyConfig{
      @Bean
      public User user01(){
          User zhangsan = new User("Zhangsan",18);
          zhangsan.setCat(cat01());
          return zhangsan;
      }
      @Bean
      public Cat cat01(){
          return new Cat("Tomcat");
      }
  }
  ```

  + Lite：==@Configuration(proxyBeanMethods = false)==

  ```java
  //配置类中的组件之间没有依赖关系时，使用Lite模式
  //可以加速SpringBoot的加载，不去检查组件是否在容器中，而是新建一个对象
  //相当于创建新组件，前后创建的不是同一个组件
  @Configuration(proxyBeanMethods = false)
  public class MyConfig{
      @Bean
      public User user01(){
          User zhangsan = new User("Zhangsan",18);
          zhangsan.setCat(cat01());//此时不成立，报错 Method annotated with @Bean is called directly in a @Configuration where proxyBeanMethods set to false. Set proxyBeanMethods to true or use dependency injection. 
          return zhangsan;
      }
      @Bean
      public Cat cat01(){
          return new Cat("Tomcat");
      }
  }
  //MainApplication中，user01.getCat()名为Tomcat，与tom相同
  //但两者不是同一个对象，因此结果为false
  	User user01 = run.getBean("user01", User.class);
      Cat tom = run.getBean("tom", Cat.class);
      Cat cat = user01.getCat();
      System.out.println(cat == tom);
  ```



#### @Import

+ 给容器中自动创建所需组件，默认组件名为全类名

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {
    Class<?>[] value();//class类对象数组
}
```



+ 示例

```java
//MyConfig中
@Import({User.class, DBHelper.class})
@Configuration
public class MyConfig {
    @Bean
    public User user01(){//...}
    //...
}
    
//MainApplication中
@SpringBootApplication
public class MainApplication{
    public static void main(String[] args){
        //IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class,args);
        
        //获取组件
        String[] beanNamesForType=run.getBeanNamesForType(User.class);
        for(String s : beanNamesForType){
            System.out.println(s);
        }
        DBHelper bean = run.getBean(DBHelper.class);
        System.out.println(bean);
    }
}
/** 结果：
* com.maziyao.boot.bean.User
* user01
* ch.qos.logback.core.db.DBHelper@61f2c3f0
**/
```



#### @Conditional

+ 条件装配：满足Conditional指定的条件，则进行组件注入

+ 示例：==@ConditionalOnBean==

```java
//示例1：方法定义@Conditional注解
@Configuration
public class MyConfig{
    /**
    *若组件中没有名为tom的组件
    *则user01也不加入容器
    **/
    @ConditionalOnBean(name = "tom")
    @Bean
    public User user01(){
        //...
    }
    
    public Cat cat01(){
        //...
    }
}
```

```java
//示例2：类定义@Conditional注解
@Configuration
@ConditionalOnBean(name = "tom")
/**
*若MyConfig配置类中有tom组件，类中组件才加入容器
**/
public class MyConfig{
    //...
}
```



### 3.2 原生配置文件引入

​	以Spring的方式，在resources中创建bean.xml进行组件的注册，但此时容器中并没有id为haha和hehe的组件

```xml
<bean id="haha" class="com.maziyao.boot.bean.User">
     <property name="name" value="zhangsan"/>
     <property name="age" value="18"/>
</bean>

<bean id="hehe" class="com.maziyao.boot.bean.Cat">
     <property name="name" value="tom"/>
</bean>
```



#### @ImportResource

+ 使用@ImportResource注解，可指定XML路径导入资源

```java
@Configuration
@ImportResource("classpath:bean.xml")
//相当于将Spring的配置文件bean.xml进行解析并将组件放入容器中
public class Myconfig{
    //....
}
```



### 3.3 配置绑定

​	如何使用java读取properties文件中的内容，并且将内容封装到JavaBean中，以供随时使用



#### @ConfigurationProperties

+ 假设在application.properties中定义以下属性

```properties
mycar.brand=BYD
mycar.price=100000
```

+ 相应地bean包中编写一个Car类
  + ==@Component==即将Car加入IOC容器中，加入容器的组件才有相应Springboot功能
  + ==@ConfigurationProperties(prefix="mycar")==配置properties文件，前缀mycar对应

```java
@Component
@ConfigurationProperties(prefix="mycar")
public class Car {
    private String brand;
    private Integer price;
    public String getBrand() {//...}
    public void setBrand(String brand) {//...}
    public Integer getPrice() {//...}
    public void setPrice(Integer price) {//...}
    @Override
    public String toString() {//...}
}
```

+ Controller中使用==@Autowired==加载Car

```java
@RestController
public class helloController{
    @Autowired
    Car car;
    @RequestMapping("/car")
    public Car car(){
        return car;
    }
}
```

+ 测试结果

![image-20210622141751715](image-20210622141751715.png)



#### @EnableConfigurationProperties

+ 在MyConfig配置类使用==@EnableConfigurationProperties==注解
  + 功能1：开启指定类的属性配置绑定功能
  + 功能2：把指定类自动封装为Bean注入到容器中

```java
@Configuration
@EnableConfigurationProperties(Car.class)
public class MyConfig{
    //...
}
```

+ bean包中的类Car使用==@ConfigurationProperties==配置文件

```java
@ConfigurationProperties(prefix="mycar")
public class Car {
    private String brand;
    private Integer price;
    public String getBrand() {//...}
    public void setBrand(String brand) {//...}
    public Integer getPrice() {//...}
    public void setPrice(Integer price) {//...}
    @Override
    public String toString() {//...}
}
```

+ Controller中同样使用==@Autowired==自动装载Car类

+ ==较@Component的优势：可以方便地加载第三方的Jar包==



## 04 自动配置原理



### 4.1 引导加载自动配置类

==@SpringBootApplication==主配置类注解等同于：

```java
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
public @interface SpringBootApplication{
    //...
}
```



#### 4.1.1 @SpringBootConfiguration

+ 本质是==@Configuration==：代表当前类是一个***配置类***

```java
@Configuration
public @interface SpringBootConfiguration {
    //...
}
```



#### 4.1.2 @ComponentScan

+ 指定包扫描：Spring注解



#### 4.1.3 @EnableAutoConfiguration

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    //...
}
```



**1、@AutoConfigurationPackage**

+ 自动配置包，指定了默认的包规则

```java
/**
* 利用Register给容器中批量导入组件
* 将指定的一个包下的所有组件导入到容器中，即MainApplication所在包下
**/
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
    //...
}
```



**2、@Import({AutoConfigurationImportSelector.class})**

+ 利用==getAutoConfigurationEntry(annotationMetadata)==给容器中导入组件

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
     if (!this.isEnabled(annotationMetadata)) {
         return EMPTY_ENTRY;
     } else {
       AnnotationAttributes attributes=this.getAttributes(annotationMetadata);
       //getCandidateConfigurations()获取所有候选配置类/组件
       List<String> configurations = 		              		this.getCandidateConfigurations(annotationMetadata, attributes); 
       configurations = this.removeDuplicates(configurations);
       Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
       this.checkExcludedClasses(configurations, exclusions);
       configurations.removeAll(exclusions);
       configurations = this.getConfigurationClassFilter().filter(configurations);
       this.fireAutoConfigurationImportEvents(configurations, exclusions);
       return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
```

+ 调用==getCandidateConfigurations(annotationMetadata, attributes)==获取到所有需要导入到容器中得候选配置类/组件

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
     List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
     Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
     return configurations;
}
```

+ 利用工厂加载==Map< String,List< String > > loadSpringFactories(@Nullable ClassLoader cl)==得到所有得组件

```java
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader){
    //...
}
```

+ 从==META-INF/spring.factories==位置加载文件，默认扫描我们当前系统里所有==META-INF / spring.f-actories==位置的文件

```java
Enumeration urls = classLoader.getResources("META-INF/spring.factories");
```



### 4.2 自动配置原理总结

+ SpringBoot先加载所有的自动配置类 XXXAutoConfiguration
+ 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。从XXXProperties中获取值。一般XXXProperties和配置文件进行了绑定
+ 生效的配置类会给容器装配很多组件
+ 定制化配置
  + 用户自定义@Bean替换底层的组件
  + 用户可以查看组件获取配置文件的值并进行相应地修改

==xxxAutoConfiguration --- > 组件 --- > xxxProperties中取值 ---> application.properties==



### 4.3 实践

+ 引入场景依赖
  + https://docs.spring.io/spring-boot/docs/2.4.7/reference/html/using-spring-boot.html#using-boot-starter
+ 查看自动配置了哪些自动配置类(可以不关心)
  + 在application.properties中写入```debug = true```开启自动配置报告
+ 是否需要修改
  + 参照官方文档修改配置项
    + https://docs.spring.io/spring-boot/docs/2.4.7/reference/html/appendix-application-properties.html#common-application-properties
  + 自主分析：xxxProperties绑定了配置文件的哪些前缀==prefix==
  + 自定义加入或替换组件
    + @Bean、@Component
  + 自定义器 xxxCustomizer



## 05 开发技巧



### 5.1 Lombok

+ 使用Lombok可以让实体类用注解代替实现Set、Get、toString及无参有参构造器方法

```xml
<dependencies>
	<dependency>
        <groupId>org.projectlombok</groupId> 
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

+ 以Car类为例：

```java
@ToString //toString方法
@Data //set和get方法
@AllArgsConstructor //全参构造器
@NoArgsConstructor //无参构造器
@ConfigurationProperties(prefix="mycar")
public class Car{
    private String brand;
    private Integer price;
}
```



### 5.2 Spring Initializr

+ 创建项目时可以使用Spring Initializr 勾选所需要的组件，自动加载（初始化向导）

<img src="image-20210629122330526.png" alt="image-20210629122330526"  />

+ 生成的项目：resources/static存放CSS、JS等静态资源；resources/templates存放页面

<img src="image-20210629122507804.png" alt="image-20210629122507804" style="zoom:80%;" />



# 二、核心功能



## 06 配置文件



### 6.1 properties

与Spring中的properties用法一致



### 6.2 yaml

非常适合用来做以==数据==为中心的配置文件，比xml更简洁、更节省资源

**基本语法**

+ key：value ; kv之间有空格
+ 大小写敏感
+ 使用==缩进==表示层级关系
+ 缩进不允许使用tab，只能用空格
+ ‘#’为注释
+ 字符串无需加引号，加了引号' '与" "表示字符串内容，会被转义/不转义



**数据类型**

+ 字面量：单个的、不可再分的值。date、boolean、string、number、null

```yaml
k: v
```

+ 对象：键值对的集合；map、hash、set、object

```yaml
# 行内写法；类似于Json
k: {k1:v1,k2:v2,k3:v3}
# 或yaml格式
k:
   k1: v1
   k2: v2
   k3: v3
```

+ 数组：一组按次序排列的值。array、list、queue

```yaml
# 行内写法
k: {v1,v2,v3}
# yaml格式
k:
   - v1
   - v2
   - v3
```



**实例**

+ Person.class

```java
@ConfigurationProperties(prefix = "person")
@Component
@ToString
@Data
public class Person {
    private String username;
    private Boolean boss;
    private Date birth;
    private Integer age;
    private Pet pet;
    private String[] interests;
    private List<String> animal;
    private Map<String,Object> score;
    private Set<Double> salary;
    private Map<String,List<Pet>> allPets;
}
```

+ Pet.class

```java
@ToString
@Data
public class Pet {
    private String name;
    private Double weight;
}
```

+ application.yaml

```yaml
person:
  username: zhangsan
  boss: true
  birth: 2019/12/9
  age: 18
  interests: [basketball,badminton,football]
  animal: [阿猫,阿狗]
  score:
    english: 90
    math: 80
  salary: [9999.97,9999.98]
  pet:
    name: 阿狗
    weight: 99.99
  allPets:
    sick:
      - {name: 阿狗,weight: 99.99}
      - name: 阿猫
        weight: 88.88
      - name: 阿虫
        weight: 77.77
    health:
      - {name: 阿花,weight: 66.66}
      - {name: 阿明,weight: 55.55}
```

+ Result

![image-20210629145636088](image-20210629145636088.png)



## 07 web开发



### 7.1 SpringMVC自动配置

SpringBoot provides ==auto-configuration== for SpringMVC that **works well with most applications**.

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.
  - ==内容协商视图解析器==和==BeanName视图解析器==
- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.4.8/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content))).
  - ==静态资源==
- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.
  - ==自动注册==```Conberter```,```GenericConverter```,```Formatter```
- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.4.8/reference/html/spring-boot-features.html#boot-features-spring-mvc-message-converters)).
- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.4.8/reference/html/spring-boot-features.html#boot-features-spring-message-codes)).
- Static `index.html` support.
- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.4.8/reference/html/spring-boot-features.html#boot-features-spring-mvc-web-binding-initializer)).
  - ==DataBinder负责将请求数据绑定到JavaBean上==

> If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.3.8/reference/html/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`
>
> **使用```@Configuration```和```@WebMvcConfigurer```自定义规则**



> If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.
>
> **声明`WebMvcRegistrations`改变默认底层组件**



> If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.
>
> **使用`@EnableWebMvc`+`@Configuration`+`DelegatingWebMvcConfiguration`全面接管Spring-MVC**



### 7.2 简单功能分析

#### 7.2.1 静态资源访问

**1、静态资源目录**

Spring Boot serves static content from a directory called `/static` (or `/public` or `/resources` or `/META-INF/resources`) in the classpath

访问：当前项目根路径 / 静态资源名

原理：静态映射请求为`/**`，即拦截所有请求；请求进来先找Controller看能不能处理。不能处理的所有请求就都交给静态资源处理器；若静态资源也找不到则404

改变默认静态资源目录：

```yaml
spring:
  web:
    resources:
      static-locations: classpath:/ 路径名
```



**2、静态资源访问前缀**

默认是无前缀

通过yaml修改static-path-pattern可以更改静态资源访问前缀

```yaml
spring:
  mvc:
    static-path-pattern: /res/**
```



### 7.2.2 Welcome Page

+ 静态资源路径下 index.html
  + 可以配置静态资源路径
  + 不可以配置静态资源访问前缀，会导致index.html不能被默认访问

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/** 会导致welcome page失效
#
  web:
    resources:
      static-locations: classpath:/haha
```

+ controller处理`/index`请求

