# Spring Boot

# 1  SpringBoot基础知识

## 1.1 为什么使用SpringBoot，SpringBoot比Spring好在哪里？

1、**简化开发**：Spring Boot通过提供一系列的**开箱即用的组件和自动配置**，简化了项目配置和开发过程。

2、**快速启动**：Spring Boot提供了快速的应用程序启动方式，可通过内嵌的Tomcat、Jetty或Undertow等容器快速启动应用程序，无需额外的部署步骤。

3、**自动化配置**：Spring Boot通过自动配置功能，**根据项目中的依赖关系和约定俗成的规则来配置应用程序。**

## 1.2 如何理解SpringBoot的自动配置？

1、自动扫描类路径 : Spring Boot在应用程序启动时会自动扫描类路径（classpath）上的组件和配置，根据发现的组件和类来自动装配应用程序的各种部分，如数据源、Web容器等。

2、条件化配置：可以配置条件，配置仅在特定条件满足时才会生效。

3、自定义配置：尽管Spring Boot提供了许多自动配置，但依然可以根据需要进行自定义。可以通过提供自己的配置来覆盖默认的自动配置，或者通过使用属性配置文件来修改自动配置的行为。

4、启动器（Starters）：Spring Boot启动器是一种依赖关系管理的机制，它可以简化项目的依赖管理，同时自动配置了特定类型的应用程序。通过引入适当的启动器，可以一次性地引入一组相关依赖和自动配置，从而快速搭建特定类型的应用程序。

# 2  SpringBoot的启动

## 2.1 @SpringBootApplication

**这个注解是 Spring Boot 项目的基石，创建 SpringBoot 项目之后会默认在主类加上。**

```java
@SpringBootApplication
public class SpringSecurityJwtGuideApplication {
      public static void main(java.lang.String[] args) {
        SpringApplication.run(SpringSecurityJwtGuideApplication.class, args);
    }
}
```

`@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。

`@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制。
`@ComponentScan`：扫描被`@Component` (`@Repository`,`@Service`,`@Controller`) 注解的 bean，注解默认会扫描该类所在的包下所有的类。
 `@Configuration`：允许在 Spring 上下文中注册额外的 bean 或导入其他配置类。

## 2.2 SpringBoot的启动流程

**Step1、创建一个 ApplicationContext 实例，即我们常说的 IoC 容器。**

**Step2、将主类（primaryClass）注册到 IoC 容器中（简单但重要的第一步）**

源配置类：通常是main方法所在的类，而且会被注解 @SpringBootApplication 所修饰，我们又称之为主类。

**Step3、递归加载并处理所有的配置类** 

![image.png](Spring%20Boot%20101e74500164805ba349ddf3457fdfa4/image.png)

SpringBoot加载配置类的方式：

**@ComponentScan**，它的作用是对指定的package进行扫描，找到其中符合条件的类，默认是搜索被注解@Component修饰的配置类 。

**@Import**，是**提供了一种显示地从其他地方加载配置类的方式**，这样可以避免使用性能较差的组件扫描（Component Scan）。

**Step4、实例化所有的单例 Bean（Singleton Bean）**

实例化所有的单例Bean,”依赖注入”和“自动装配@Autowired”就属于其中的环节

**Step5、如果是 Web 应用，则启动 Web 服务器（例如Tomcat）**

## 2.3 SpringBoot自动配置原理

Auto-Configuration:基于引入的依赖jar包，对 SpringBoot 应用进行自动配置。为SpringBoot **“开箱即用**”提供了基础支撑。

注：这里很容易将自动装配与自动配置搞混，自动配置：Auto-Configuration  自动装配：Autowire

**SpringBoot框架最终会导入 AutoConfigurationImportSelector 来实现自动配置，如何实现AutoConfigurationImportSelector？使用SpringFactories机制。**

### **自动配置原理（考过）**

**1、自动配置的入口：**@EnableAutoConfiguration 注解启动，因为 **@SpringBootApplication** 是一个复合注解，包含了 @EnableAutoConfiguration。

**2、自动配置类的加载：** Spring Boot 在启动时会去 **依赖的 Starter 包中** 寻找 resources/META-INF **/spring.factories** 文件 **（文件中定义了自动配置类的全限定名，Spring Boot 启动时会加载这些类。）**，然后根据 **文件中配置的 Jar 包**去扫描项目所依赖的 Jar 包。

---

根据 **spring.factories** 配置加载 AutoConfigure 类。

spring.factories是SpringBoot SPI机制实现的核心，SPI机制表示扩展机制，spring.factories文件的作用就是来对SpringBoot进行扩展的。我们可以直接向文件中添加，而不需要额外的去写代码。

---

**3、条件化配置：** 根据 **@Conditional 注解**的条件，**过滤掉不必要的自动配置类**，进行自动配置并将 Bean 注入 Spring Context 。

---

条件注解：通过检查一组预定义的条件来判断是否满足某个条件，如果条件成立，则相应的组件或配置会被启用，否则会被忽略。

常见的：

@ConditionalOnClass : 指定的类位于路径上时，才会启用被注解的组件或配置

@ConditionalOnMissingClass :  指定的类不在路径上时，才会启用被注解的组件或配置

@ConditionalOnBean : 当指定的Bean在应用程序上下文中存在时，才会启用被注解的组件或配置

---

**4、配置属性绑定：** Spring Boot 使用 @ConfigurationProperties 注解将配置文件中的属性绑定到 Java 类中。

**5、自动配置的加载顺序：** Spring Boot 会按照 **spring.factories 或 AutoConfiguration.imports 文件中定义的顺序加载配置类**，加载顺序会影响 Bean 的优先级

其实就是 Spring Boot 在启动的时候，按照约定去**读取 Spring Boot Starter 的配置信息**，再根据配置信息**对资源进行初始化**，并注入到 Spring 容器中。这样 Spring Boot 启动完毕后，就已经准备好了一切资源，使用过程中直接注入对应 Bean 资源即可。

### 总结

![image.png](Spring%20Boot%20101e74500164805ba349ddf3457fdfa4/image%201.png)

@Import 是 Spring Boot 自动配置机制的一个**关键环节，它通过引入 AutoConfiguration-ImportSelector，启动了自动配置的加载和筛选过程。**

虽然 @Import 是触发自动配置的一种常见方式，但 **Spring Boot 提供了多种机制来启动自动配置**，

- **直接使用 @EnableAutoConfiguration（@SpringBootApplication 包含其中）：通过内部的 @Import 触发自动配置。**
- 使用 @SpringBootApplication：这是一个更高级的注解，内部已经包含了 @EnableAuto-Configuration。
- 通过 SpringApplication 的构造器或方法：显式启用自动配置。

## 2.4 Springboot接口开发的常用注解有哪些？

@Controller 标记此类是一个**控制器** 可以返回视图解析器指定的html页面，通过@ResponseBody可以将结果返回json、xml数据。

@RestController **相当于@ResponseBody + @Controller**,实现rest接口开发，返回json数据，**不能返回html页面！**

@RequestMapping 定义接口地址，**可以标记在类上也可以标记在方法上**，支持http的pos、put、get等方法。

**@PostMapping、@GetMapping、@PutMapping**只能标记在方法上。

**@ResponseBody** ： 将数据以 **json的格式** 响应给 **前端（侧重返回，定义在类上）**

**@RequestBody** ：将 **json数据** 转化为 **java对象（侧重接收，定义在方法上）**

---

**@RequestParam** : 将请求参数绑定到控制器的方法参数上（是SpringMVC中接收普通参数的注解）

1. 语法：**@RequestParam(value=”参数名”,required=”true/false”,defaultValue=””)**

2. required：是否包含该参数，默认为true，表示该请求路径中必须包含该参数，如果不包含就报错。

3. defaultValue：默认参数值，如果设置了该值，required=true将失效，自动为false,如果没有传该参数，就使用默认值。

**@PathVariable ：** 接收请求路径中占位符的值。

![Untitled](Spring%20Boot%20101e74500164805ba349ddf3457fdfa4/Untitled.png)

---

**@Autowired：是基于类型的注入。**

**@Resource：基于名称注入。名称注入失败则自动转换为基于类型注入。**

## **2.5 常见的启动器（starter）**

SpringBoot的Starter机制：Starter其实没东西，就是一组 **【预配置的依赖项集合】**，用于启用特定类型的功能。构建项目时，只需要引入这个Starter，而无需单独处理每个依赖项的版本和配置。

### spring-boot-starter-web

最常用的起步依赖之一，包含了Spring MVC和Tomcat嵌入式服务器，用于快速构建Web应用程序

### spring-boot-starter-security

提供了Spring Security的基本配置，帮助开发者快速实现应用的安全性，包括认证和授权功能。

### mybatis-spring-boot-starter

用于简化在Spring Boot应用中集成MyBatis的过程。

# 3 SpringBoot 用到哪些设计模式？

### 1、代理模式

Spring的AOP通过**动态代理**实现方法级别的切面增强

SpringBoot 默认使用CGLib动态代理。主要考虑到：1、性能和速度  2、无需接口  3、无侵入性。

### 2、策略模式

Spring AOP支持JDK和CGLib两种动态代理实现方式，**通过策略接口实现不同的策略类，** 运行时动态选择，其创建一般通过工厂方法实现。

### 3、装饰器模式

Spring用TransactionAwareCacheDecorator解决缓存与数据库事务问题增加对事务的支持。

### 4、单例模式

Spring Bean默认是单例模式，通过单例注册表（如HashMap）实现。

### 5、简单工厂模式、工厂方法模式

Spring中的FactoryBean体现工厂方法模式，为不同产品提供不同的工厂。

### 6、观察者模式

Spring观察者模式包含Event事件、Listener监听者、Publisher发送者，通过定义事件、监听器和发送者实现，观察者注册在ApplicationContext中，消息发送由ApplicationEventMulticaster完成。

### 7、适配器模式

Spring MVC中**针对不同方式定义的Controller，利用适配器模式统一函数定义**，定义了统一接口HandlerAdapter及对应适配器类。

# 4  Spring Boot中“约定大于配置”

通过约定好的方式来提供默认行为 **（通过减少配置和提供合理的默认值，使得开发者可以快速地构建和部署应用程序）**，减少开发者需要做出的决策。

**自动化配置**：Spring Boot根据项目的依赖和环境自动配置应用程序，无需手动配置大量的XML和Java配置文件。

**起步依赖**：Spring Boot提供了一系列起步依赖，这些依赖包含了常用的框架和功能，可以帮助开发者快速搭建项目。

# 5  Spring Boot的项目结构是怎么样的？

![image.png](Spring%20Boot%20101e74500164805ba349ddf3457fdfa4/image%202.png)

对应代码目录：

![image.png](Spring%20Boot%20101e74500164805ba349ddf3457fdfa4/image%203.png)
