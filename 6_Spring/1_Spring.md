# Spring

# 1、Spring基本概念

一般说的**Spring框架**指的都是**Spring Framework,** 它是很多模块的集合，**Spring支持Ioc和Aop。** 是一款开源的轻量级Java开发框架，旨在提高开发人员的开发效率以及系统的可维护性。可以方便的对数据库进行访问，集成第三方组件。

**Spring各个模块的依赖如下：**

![image.png](Spring%20101e74500164808292f0d49a46528c08/image.png)

- **Spring Core 核心模块**

Spring 框架的基础，提供了**依赖注入（DI）和 控制反转（IoC）** 的功能。它通过 BeanFactory 和 ApplicationContext 管理对象的生命周期和依赖关系。

- **Spring Context（上下文模块）**

**扩展了 BeanFactory 的功能，** 提供了更高级的容器功能，如国际化支持、事件传播、资源加载等。

- **Spring AOP（面向切面编程模块）**

提供了面向切面编程（AOP）的支持，允许开发者将横切关注点（如日志、事务管理、安全等）与业务逻辑分离。

- **Spring MVC（Web 模块）**

提供了构建 Web 应用程序的框架，支持请求映射、视图解析、数据绑定等功能。

- **Spring Boot（启动模块）**

用于简化 Spring 应用程序的初始搭建和开发过程。

## 1.1 Spring、Spring MVC、Spring Boot 之间什么关系？

Spring包含了多个功能模块，最重要的是Spring-Core(主要提供IoC依赖注入功能)模块，Spring其他模块（SpringMVC）都依赖与Spring-Core。使用依赖注入实现控制反转，可以很方便的整合各种框架。

**Spring MVC是Spring中很重要的一个模块，** 主要赋予 Spring **快速构建 MVC 架构的 Web 程序**的能力。**MVC 是模型(Model)、视图(View)、控制器(Controller)的简写**，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

![image.png](Spring%20101e74500164808292f0d49a46528c08/image%201.png)

Spring Boot：简化Spring开发 **（减少配置文件，开箱即用！！！）只是简化了配置。** 如果需要构建MVC架构的Web程序，还是需要Spring MVC框架。SpringBoot只能简化配置。整合了一系列解决方案（starter机制）。

# 2. Spring IOC

**Inversion Of Control**，即 **控制反转**，**是一种设计思想，而不是一个具体的技术实现。将原本在程序中手动创建（实例化、管理）的权力【控制】，交给Spring框架来管理【反转】**

在传统的 Java SE 程序设计中，我们直接在对象内部通过 new 的方式来创建对象，是程序主动创建依赖对象；

![Untitled](Spring%20101e74500164808292f0d49a46528c08/Untitled.png)

而在Spring程序设计中，**IOC 是有专门的容器去控制对象。（Ioc容器实际上就是个Map,Map中存放的是各种对象）**

![Untitled](Spring%20101e74500164808292f0d49a46528c08/Untitled%201.png)

**所谓控制**就是对象的**创建、初始化、销毁。**

- 创建对象：原来是new 一个，现在是由 Spring 容器创建。
- 初始化对象：原来是对象自己通过构造器或者 setter 方法给依赖的对象赋值，现在是由 Spring 容器自动注入。
- 销毁对象：原来是直接给对象赋值 null 或做一些销毁操作，现在是 Spring 容器管理生命周期负责销毁对象。

**所谓反转**：其实是反转的控制权，前面提到是由 Spring 来控制对象的生命周期，那么对象的控制就完全脱离了我们的控制，控制权交给了 Spring 。这个反转是指：我们由对象的控制者变成了 IOC 的被动控制者。

**总结：IOC 解决了繁琐的对象生命周期的操作，解耦了我们的代码。**

## 2.1 Bean对象

Bean 代指的就是那些 **被 IoC 容器所管理的对象**。告诉 IoC 容器帮助我们管理哪些对象，这个是通过配置元数据来定义的。Bean 是 Spring 应用程序的基本构建块，几乎所有核心功能都依赖于 Bean。

配置元数据可以是 **XML 文件**、**注解（最常用！！！）** 或者 **Java 配置类。**

### 2.1.1一个类声明为Bean的注解

1、 **`@Component`**：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。

2、 **`@Repository`** : 对应持久层即 Dao 层，主要用于数据库相关操作。

3、 **`@Service`** : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。

4、 **`@Controller`** : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 `Service` 层返回数据给前端页面。

声明为Bean的注解不同，主要是为了语义化区分：通过不同的注解，明确标识类的角色和职责（如数据访问层、服务层、控制层等）。

- **@Component 和@Bean的区别？**

**`@Component` 注解作用于类，而`@Bean`注解作用于方法。** `@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。
`@Bean`注解使用示例。

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

### 2.1.2 注入Bean的注解

Spring 内置的 `@Autowired` 以及 JDK 内置的 `@Resource` 和 `@Inject` 都可以用于注入 Bean。
`Autowired` 属于 Spring 内置的注解，默认的注入方式为**`byType`（根据类型进行匹配）**，也就是说会优先根据**接口类型**去匹配并注入 Bean （接口的实现类）。

**存在问题**

 当一个接口 **存在多个实现类的话，** `byType` 这种方式就 **无法正确注入对象了**，因为这个时候 Spring 会同时找到多个满足条件的选择，默认情况下它自己不知道选择哪一个。**这种情况下，注入方式会变为 `byName`（根据名称进行匹配），这个名称通常就是类名（首字母小写）**

### @Autowired 和 @Resource 的区别

 @Resource属于JDK提供的注解，@Autowired是Spring提供的注解

```java
// smsService 就是我们上面所说的名称
@Autowired
private SmsService smsService;
```

例子，`SmsService` 接口有两个实现类: `SmsServiceImpl1`和 `SmsServiceImpl2`，且它们都已经被 Spring 容器所管理。

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Autowired
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Autowired
private SmsService smsServiceImpl1;
// 正确注入  SmsServiceImpl1 对象对应的 bean
// smsServiceImpl1 就是我们上面所说的名称
@Autowired
@Qualifier(value = "smsServiceImpl1")//Autowired 可以通过 @Qualifier 注解来显式指定名称
private SmsService smsService;
```

`@Resource`属于 JDK 提供的注解，默认注入方式为 `byName`。**如果无法通过名称匹配到对应的 Bean 的话，注入方式会变为`byType`。**

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Resource
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Resource
private SmsService smsServiceImpl1;
// 正确注入 SmsServiceImpl1 对象对应的 bean（比较推荐这种方式）
@Resource(name = "smsServiceImpl1")//@Resource可以通过 name 属性来显式指定名称。
private SmsService smsService;
```

注：@Autowired用的多是因为：@Autowired 是 Spring 框架的原生注解，与 Spring 生态高度集成，

### 2.1.3 注入Bean的方式有哪些？

依赖注入 (Dependency Injection, DI) 的常见方式：

- **构造函数注入：** 通过类的构造函数来注入依赖项。

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    //...
}
```

- **Setter 注入：** 通过 **类的 Setter 方法(构造方法)** 来注入依赖项。

```java
@Service
public class UserService {

    private UserRepository userRepository;

    // 在 Spring 4.3 及以后的版本，特定情况下 @Autowired 可以省略不写
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    //...
}
```

- **Field（字段） 注入：** 直接在类的字段上使用注解（如 `@Autowired` 或 `@Resource`）来注入依赖项。

### 2.1.4 Bean是线程安全的吗？（考过）

Spring 框架中的 Bean 是否线程安全，**取决于其作用域和状态。**
大部分 Bean 实际都是 **无状态（没有定义可变的成员变量）** 的（比如 Dao、Service），这种情况下， **Bean 是线程安全的。**

**有状态的Bean存在线程安全问题**

即Bean的成员变量被多个线程共享时，举个例子：一个简单的Java Bean类 `Counter`，它包含一个计数器变量 `count`，并且提供了一个方法来增加计数器的值：

```java
public class Counter {
    private int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```

解决方式：加锁、使用原子类、使用ThreadLocal等

### 2.1.5 Bean的生命周期

**step1、实例化 —> Bean 容器首先会找到配置文件中的Bean定义，然后使用Java反射API来创建Bean的实例。**

**step2、属性赋值/填充 —> 为Bean设置相关的属性和依赖（比如@Autowired等注解注入的对象）**  

**step3、初始化 —> 传入对应实现接口的对象实例**

**step4、销毁 —>  销毁并不是说立马把Bean给销毁掉，而是把Bean的销毁方法先记录下来，将来需要销毁Bean或者销毁容器的时候，就调用这些方法去释放Bean所持有的资源。**

---

**追问：Bean的实例化和初始化有什么区别？**

Spring 在创建一个Bean对象时，会先创建出来一个Java对象，会通过反射来执行类的构造方法从而得到一个Java对象，而这个过程就是Bean实例化。**核心的方法是 doCreateBean(), 这个方法里面有createBeanInstance()方法来 实例化** 。

得到Java对象后，会进行依赖注入，依赖注入后就会进行初始化了；Bean的初始化就是调用前面创建出来的Java对象中特定的方法。**注入后，实现接口的对象实例，调用实现类中的方法，即初始化。**

---

### 2.1.6 Bean的作用域

Spring Ioc容器中，支持多种其他作用域的Bean：

- **Singleton（单例）：默认作用域，单例Bean 。** 容器中只有一个 Bean 实例。
- **Prototype（原型）：** 每次请求时都会创建一个新的 Bean 实例。
- Request：每个 HTTP 请求创建一个 Bean 实例（仅适用于 Web 应用）。
- Session：每个 HTTP 会话创建一个 Bean 实例（仅适用于 Web 应用）。
- Application：每个 ServletContext 生命周期内创建一个 Bean 实例（仅适用于 Web 应用）。
- WebSocket：每个 WebSocket 会话内创建一个 Bean 实例（仅适用于 Web 应用）。

### 2.1.7 单例Bean是单例模式吗？

是的

单例模式是指在一个JVM中，**一个类只能构造出一个对象**。Spring中的单例Bean也是一种单例模式 **，只不过范围比较小，范围是beanName**,一个beanName对应同一个Bean对象，不同的beanName可以对应不同的Bean对象。

## 2.2 依赖注入 DI

### 定义

依赖：**一个对象 需要 另一个对象来完成其功能。** 例如，UserService 依赖于 UserRepository 来访问数据库。

注入：**将依赖对象（如 UserRepository）通过某种方式传递给需要它的对象（如 UserService）**，而不是由需要依赖的对象自己创建或查找依赖。

依赖注入的核心思想是：**将对象的依赖关系交给外部容器（如 Spring 容器）来管理，而不是在对象内部硬编码依赖关系。**

## 2.3 Ioc的工作流程

**阶段一：Ioc容器的初始化阶段**

Spring 容器启动时，首先会**加载配置文件或扫描注解（根据程序中定义的XML或者注解等Bean的声明方式，）**，获取 Bean 的定义信息。

实例化 Bean：Spring 容器根据 Bean 的定义信息，通过反射机制创建 Bean 的实例。

依赖注入：容器根据 Bean 的依赖关系，将所需的依赖注入到 Bean 中。

**阶段二：完成Bean初始化、使用与销毁**

初始化 Bean

Bean的使用：**通常会使用@Autowired注解或者通过BeanFactory.getBean()从Ioc容器里面去获取一个指定Bean的一个实例。**

销毁 Bean：当容器关闭时，会销毁所有的单例 Bean。

ApplicationContext就是Ioc容器，它继承了BeanFactory

![image.png](Spring%20101e74500164808292f0d49a46528c08/image%202.png)

## 2.4 BeanFactory

**BeanFactory 是 Spring 框架中最核心的接口之一，它是 Spring IoC（控制反转）容器的基础，负责管理应用中 Bean 的生命周期、依赖注入和配置。**

- **核心作用**

**1、Bean 的管理**：负责创建、配置和管理 Bean 实例。

2、**依赖注入（DI）**：自动解决 Bean 之间的依赖关系。

**3、延迟加载**：默认情况下，Bean 是按需创建的（在首次请求时初始化）。

**4、配置灵活性**：支持多种配置方式（XML、注解、Java 配置等）。

- **BeanFactory接口定义的关键方法**

BeanFactory 接口提供了一系列方法用于操作 Bean，核心方法包括：
Object getBean(String name)：根据名称获取 Bean 实例。
<T> T getBean(Class<T> requiredType)：根据类型获取 Bean 实例。

![image.png](Spring%20101e74500164808292f0d49a46528c08/image%203.png)

# 3. Spring AOP

Spring AOP是Spring框架中的一个重要模块，用于 **实现面向切面编程。**

Java 就是一门面向对象编程的语言，在 OOP 中最小的单元就是“Class 对象”，但是在 AOP 中最小的单元是“切面”。一个“切面”可以包含很多种类型和对象，对它们进行模块化管理，例如事务管理。

在面向切面编程的思想里面，把功能分为两种： 

- **核心业务**：登陆、注册、增、删、改、查都叫核心业务
- **周边功能**：**日志、事务管理**这些次要的为周边业务

AOP能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度，并有利于未来的可拓展性和可维护性**。

Spring AOP 就是 **基于动态代理** 的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 **JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了。

注：JDK动态代理：基于接口实现，通过java.lang.reflect.Proxy动态代理生成代理类。CGLIB动态代理：基于类继承，通过字节码技术生成目标类的子类，来实现对目标方法的代理。

### 具体实现过程（考过）

Bean的创建过程中有一个步骤可以对Bean进行扩展实现，AOP本身就是一个扩展功能，所以在**BeanPostProcessor的后置处理方法**中来进行实现：

1、定义切面：创建一个切面类，使用 **@Aspect 注解**标记。

2、定义切点：使用 **@Pointcut 注解**定义切点表达式，**匹配需要拦截的连接点。**

3、创建代理对象：Spring 容器在启动时，会扫描所有 Bean，**并根据切点表达式匹配目标对象。如果目标对象符合切点表达式，Spring会为其创建代理对象。** 通过JDK或者CGLIB的方式来生成代理对象在执行方法调用的时候，会调用到生成的字节码文件中，直接回找到DynamicAdvisored-Interceptor类中的intercept方法，从此方法开始执行

4、**执行通知：** 根据之前定义好的**通知**来 **生成拦截器链，** 从拦截器链中依次获取每一个通知开始进行执行，在执行过程中，为了方便找到下一个通知是哪个，会有一个InvocationInterceptor的对象，找的时候是从-1的位置一次开始查找并且执行的。

![image.png](Spring%20101e74500164808292f0d49a46528c08/image%204.png)

**切入点一定是连接点，连接点不一定是切入点**

### 通知类型

- **Before（前置通知）**：目标对象的方法调用之前触发
- **After （后置通知）**：目标对象的方法调用之后触发
- **AfterReturning（返回通知）**：目标对象的方法**调用完成，在返回结果值**之后触发
- **AfterThrowing（异常通知）**：目标对象的方法运行中抛出 / 触发异常后触发。AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值。
- **Around（环绕通知）**：编程式控制目标对象的方法调用。环绕通知是所有通知类型中可操作范 围最大的一种，因为它可以直接拿到目标对象，以及要执行的方法，所以环绕通知可以任意的在目标对象的方法调用前后搞事，甚至不调用目标对象的方法。

### 多个切面的执行顺序如何确定？

**通常使用`@Order` 注解直接定义切面顺序。**

### 示例

**1、定义业务组件目标**

```java
@Service
public class UserService {
    
    public String getUserById(Long id) {
        System.out.println("执行 getUserById 方法");
        return "User-" + id;
    }
    
    public void deleteUser(Long id) {
        System.out.println("执行 deleteUser 方法");
        if (id < 0) {
            throw new IllegalArgumentException("ID不能为负数");
        }
    }
}

@Service
public class OrderService {
    
    public String createOrder(String product) {
        System.out.println("执行 createOrder 方法");
        return "Order-" + product;
    }
}
```

**2、定义切入点和通知**

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {

    // ========== 定义切入点 ==========
    
    // 1. 匹配 UserService 所有方法
    @Pointcut("execution(* com.example.demo.service.UserService.*(..))")
    public void userServiceMethods() {}
    
    // 2. 匹配所有 Service 类的 public 方法
    @Pointcut("execution(public * com.example.demo.service.*.*(..))")
    public void servicePublicMethods() {}
    
    // 3. 匹配所有包含 @Transactional 注解的方法
    @Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")
    public void transactionalMethods() {}

    // ========== 定义通知 ==========
    
    // 1. 前置通知
    @Before("userServiceMethods()")
    public void logBeforeUserServiceMethod(JoinPoint jp) {
        String methodName = jp.getSignature().getName();
        System.out.println("[前置通知] 准备执行 UserService 方法: " + methodName);
    }
    
    // 2. 后置通知（无论是否抛出异常都会执行）
    @After("servicePublicMethods()")
    public void logAfterServiceMethod(JoinPoint jp) {
        System.out.println("[后置通知] 方法执行完成: " + jp.getSignature().getName());
    }
    
    // 3. 返回通知（方法正常返回时执行）
    @AfterReturning(
        pointcut = "servicePublicMethods()",
        returning = "result"
    )
    public void logAfterReturning(JoinPoint jp, Object result) {
        System.out.println("[返回通知] 方法 " + jp.getSignature().getName() 
            + " 返回结果: " + result);
    }
    
    // 4. 异常通知
    @AfterThrowing(
        pointcut = "servicePublicMethods()",
        throwing = "ex"
    )
    public void logAfterThrowing(JoinPoint jp, Exception ex) {
        System.out.println("[异常通知] 方法 " + jp.getSignature().getName() 
            + " 抛出异常: " + ex.getMessage());
    }
    
    // 5. 环绕通知（最强大的通知类型）
    @Around("transactionalMethods()")
    public Object logAround(ProceedingJoinPoint pjp) throws Throwable {
        String methodName = pjp.getSignature().getName();
        System.out.println("[环绕通知-前] 准备执行事务方法: " + methodName);
        
        try {
            Object result = pjp.proceed(); // 执行目标方法
            System.out.println("[环绕通知-后] 事务方法执行成功: " + methodName);
            return result;
        } catch (Exception e) {
            System.out.println("[环绕通知-异常] 事务方法执行失败: " + methodName);
            throw e;
        }
    }
}
```

**3、测试类**

```java
@SpringBootTest
public class AopTest {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private OrderService orderService;
    
    @Test
    public void testAop() {
        // 测试 UserService 方法（会触发 userServiceMethods 切入点）
        userService.getUserById(1L);
        
        System.out.println("==========");
        
        // 测试 OrderService 方法（会触发 servicePublicMethods 切入点）
        orderService.createOrder("iPhone");
        
        System.out.println("==========");
        
        // 测试异常情况
        try {
            userService.deleteUser(-1L);
        } catch (Exception e) {
            System.out.println("捕获到异常: " + e.getMessage());
        }
    }
}
```

### 与注解区别

通过注解和AOP都可以实现函数耗时统计功能，但两者在实现方式和适用场景上有显著差异。

**AOP实现通常是更优的选择，因为它提供了更好的可维护性和扩展性。** 而对于小型工具类或性能敏感的场景，简单的注解实现可能更为合适。

![image.png](Spring%20101e74500164808292f0d49a46528c08/image%205.png)

# 4. Spring MVC

MVC 是一种设计模式，Spring MVC 是一款很优秀的 MVC 框架。Spring MVC 可以帮助我们进行更简洁的 Web 层的开发，并且它天生与 Spring 框架集成。

Spring MVC 下我们一般把后端项目分为 Service 层（处理业务）、Dao 层（数据库操作）、Entity 层（实体类）、Controller 层(控制层，返回数据给前台页面)。**它提供了一种松耦合的方式将用户请求、业务逻辑和视图渲染分离开来。**

## 4.1 统一异常处理

推荐**使用注解**的方式统一异常处理，具体会使用到 `@ControllerAdvice` + `@ExceptionHandler` 这两个注解。

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(BaseException.class)
    public ResponseEntity<?> handleAppException(BaseException ex, HttpServletRequest request) {
      //......
    }

    @ExceptionHandler(value = ResourceNotFoundException.class)
    public ResponseEntity<ErrorReponse> handleResourceNotFoundException(ResourceNotFoundException ex, HttpServletRequest request) {
      //......
    }
}
```

这种异常处理方式下，会给所有或者指定的 `Controller` 织入异常处理的逻辑（AOP），当 `Controller` 中的方法抛出异常的时候，**由被`@ExceptionHandler` 注解修饰的方法进行处理。**

## 4.2 Spring如何解决循环依赖问题

**循环依赖指的是两个类中的属性相互依赖对方：例如 A 类中有 B 属性，B 类中有 A属性，从而形成了一个依赖闭环**，**无法确定加载或初始化的顺序**

如下图。

![Untitled](Spring%20101e74500164808292f0d49a46528c08/Untitled%202.png)

### 三级缓存

Spring 框架通过使用**三级缓存**来解决这个问题，确保即使**在循环依赖的情况下也能正确创建 Bean**

Spring 中的三级缓存其实就是三个 Map，如下：

```java
// 一级缓存
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

// 二级缓存
/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

// 三级缓存
/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

**一级缓存（singletonObjects）**：**存放最终形态的 Bean（已经实例化、属性填充、初始化）**，单例池，为“Spring 的单例属性”⽽⽣。一般情况我们获取 Bean 都是从这里获取的，但是并不是所有的 Bean 都在单例池里面，例如原型 Bean 就不在里面。

**二级缓存（earlySingletonObjects）**：**存放过渡 Bean（半成品，尚未属性填充）**，也就是三级缓存中`ObjectFactory`产生的对象，与三级缓存配合使用的，可以防止 AOP 的情况下，每次调用`ObjectFactory#getObject()`都是会产生新的代理对象的。

**三级缓存（singletonFactories）**：**存放`ObjectFactory` 对象工厂**， `ObjectFactory`的`getObject()`方法（最终调用的是`getEarlyBeanReference()`方法）可以生成原始 Bean 对象或者代理对象（如果 Bean 被 AOP 切面代理）。**三级缓存只会对单例 Bean 生效。**

![image.png](Spring%20101e74500164808292f0d49a46528c08/image%206.png)

---

**Spring 创建 Bean 的流程:**
1.  先去 **一级缓存 `singletonObjects`** 中获取，存在就返回；

2.  如果不存在或者对象正在创建中，于是去 **二级缓存 `earlySingletonObjects`** 中获取；

3. 如果还没有获取到，就去 **三级缓存 `singletonFactories`** 中获取，通过执行 `Object-Facotry` 的 `getObject()` 就可以获取该对象，获取成功之后，从三级缓存移除，并将该对象加入到二级缓存中。

---

解决循环依赖的流程如下：
1、 当 Spring 创建 A 之后，发现 A 依赖了 B ，又去创建 B，B 依赖了 A ，又去创建 A；

2、 在 B 创建 A 的时候，那么此时 A 就发生了循环依赖，由于 A 此时还没有初始化完成，因此在 **一二级缓存** 中肯定没有 A；

![创建B时，B依赖于A，而A对象又没创建完成](Spring%20101e74500164808292f0d49a46528c08/image%207.png)

创建B时，B依赖于A，而A对象又没创建完成

![B只能从第三级缓存中获取A](Spring%20101e74500164808292f0d49a46528c08/image%208.png)

B只能从第三级缓存中获取A

3、那么此时就去三级缓存中调用 `getObject()` 方法去获取 A 的 **前期暴露的对象** ,也就是调用上边加入的 `getEarlyBeanReference()` 方法，生成一个 A 的 **前期暴露对象**；

4、然后就将这个 `ObjectFactory` 从三级缓存中移除，**并且将前期暴露对象放入到二级缓存中，那么 B 就将这个前期暴露对象注入到依赖，来支持循环依赖。**

![其实这里调用了AOP，二级缓存中生成A对象的代理，注入到B中，然后将三级缓存中的对象移除](Spring%20101e74500164808292f0d49a46528c08/image%209.png)

其实这里调用了AOP，二级缓存中生成A对象的代理，注入到B中，然后将三级缓存中的对象移除

![结B的创建过程，此时B已经被创建，放于一级缓存中](Spring%20101e74500164808292f0d49a46528c08/image%2010.png)

此时B已经被创建，放于一级缓存中

总结：如果发生循环依赖的话，就去 **三级缓存 `singletonFactories`** 中拿到三级缓存中存储的 `ObjectFactory` 并调用它的 `getObject()` 方法来获取这个循环依赖对象的**前期暴露对象（虽然还没初始化完成，但是可以拿到该对象在堆中的存储地址了）**，并且将这个前期暴露对象放到二级缓存中，这样在循环依赖时，就不会重复初始化了！

循环依赖问题在Spring中主要有三种情况：

- 第一种：通过构造方法进行依赖注入时产生的循环依赖问题。
- 第二种：通过setter方法进行依赖注入且是在多例（原型）模式下产生的循环依赖问题。
- **第三种：通过setter方法进行依赖注入且是在单例模式下产生的循环依赖问题。**

只有【第三种方式】的循环依赖问题被 Spring 解决了，其他两种方式在遇到循环依赖问题时，Spring都会产生异常。

Spring 解决**单例模式（一定要是单例模式！！！）** 下的setter循环依赖问题的主要方式是通过三级缓存解决循环依赖。三级缓存指的是 Spring 在创建 Bean 的过程中，通过三级缓存来缓存正在创建的 Bean，以及已经创建完成的 Bean 实例。具体步骤如下：

1. **实例化 Bean**：Spring 在实例化 Bean 时，会先创建一个空的 Bean 对象，并将其放入一级缓存中。

2. **属性赋值**：Spring 开始对 Bean 进行属性赋值，如果发现循环依赖，会将当前 Bean 对象提前暴露给后续需要依赖的 Bean（通过提前暴露的方式解决循环依赖）。

3. **初始化 Bean**：完成属性赋值后，Spring 将 Bean 进行初始化，并将其放入二级缓存中。

4. **注入依赖**：Spring 继续对 Bean 进行依赖注入，如果发现循环依赖，会从二级缓存中获取已经完成初始化的 Bean 实例。

通过三级缓存的机制，Spring 能够在处理循环依赖时，确保及时暴露正在创建的 Bean 对象，并能够正确地注入已经初始化的 Bean 实例，从而解决循环依赖问题，保证应用程序的正常运行。

## 4.3 SpringMVC的工作流程（也是考过的）

![Untitled](Spring%20101e74500164808292f0d49a46528c08/Untitled.jpeg)

1. 用户发送请求至前端控制器**DispatcherServlet。**

2. **DispatcherServlet**收到请求调用处理器映射器HandlerMapping。

3. 处理器映射器**根据请求url找到具体的处理器**，生成处理器执行链HandlerExecutionChain(包括处理器对象和处理器拦截器)一并返回给**DispatcherServlet**。

4. DispatcherServlet根据处理器Handler获取处理器适配器**HandlerAdapter执行HandlerAdapter处理一系列的操作，如：参数封装，数据格式转换，数据验证等操作**

5. 执行处理器Handler(Controller，也叫页面控制器)。

6. Handler执行完成返回 ModelAndView。

7. HandlerAdapter 将 Handler 执行结果 ModelAndView 返回到 DispatcherServlet。

8. DispatcherServlet 将 ModelAndView   传给ViewReslover**视图解析器。**

9. ViewReslover解析后**返回具体View。**

10. **DispatcherServlet**对**View进行渲染视图（即将模型数据model填充至视图中）**。

11. **DispatcherServlet**响应用户。

---

### SpringMVC 的核心 — DispatcherServlet

中央调度器：作为所有HTTP请求的统一入口，负责协调各组件完成请求处理的全生命周期。

核心职责：接受请求 —> 委托组件处理 —>返回响应，实现**控制反转（这也有控制反转的思想？）**，将开发者的关注点聚焦于业务逻辑。

```sql
HTTP 请求 -> DispatcherServlet -> HandlerMapping -> HandlerAdapter -> Controller
                            |
                            +-> ModelAndView -> ViewResolver -> View -> 渲染响应
```

### HandlerMapping

处理器映射器，负责将请求映射到合适的处理器（Controller）

### HandlerAdaptor

处理器适配器，负责调用处理器（Controller）并执行其中的方法。

### Controller

控制器，处理具体的业务逻辑，返回ModelAndView对象。在Spring MVC中，控制器常通过@Controller注解标记。

### ModelAndView

视图模型，封装视图逻辑名和模型数据。控制器处理完请求后，返回一个ModelAndView 

### ViewResolver

视图解析器，将视图逻辑名解析为具体的View实现。

### View

视图，负责渲染视图，并将模型数据转换为响应内容

---

## 4.4 拦截器链

Spring 拦截器链（Interceptor Chain） 是 Spring MVC 框架中用于处理请求的一种机制。它通过一组拦截器（HandlerInterceptor）在请求处理的不同阶段执行特定的逻辑，例如权限验证、日志记录、性能监控等。

### 拦截器 Interceptor

拦截器是一种动态拦截方法调用的机制，类似于过滤器。Spring框架中提供的，用来动态拦截控制器方法执行。

拦截器是一个接口，定义了三个方法：

preHandle：在请求处理之前执行。

postHandle：在请求处理之后、视图渲染之前执行。

afterCompletion：在请求完成之后执行（视图渲染之后）。

请求会依次通过每个拦截器的 preHandle 方法，然后由处理器（Controller）处理请求，最后再依次通过每个拦截器的 postHandle 和 afterCompletion 方法。

### 过滤器

过滤器可以把对资源的请求拦截下来，从而实现一些特殊的功能。

过滤器一般完成一些通用的操作：登录校验、统一编码处理。

### 拦截器与过滤器区别（重点！）

**1、归属层级不同：** 过滤器（Filter）是Servlet规范的一部分，拦截器(Interceptor)是Spring框架中的一部分。

**2、作用范围：** 过滤器：在请求进入Servlet之前和响应返回客户端之前，对所有请求生效（包括静态资源）。

拦截器：在进**入Controller之前和视图渲染之前，只对Spring MVC处理的请求生效。**

总的来说：过滤器更底层，适用于所有请求，拦截器更上层，适用于Spring MVC处理请求。

# 5. Spring 事务

Spring支持事务管理—-实际是**通过 AOP** 实现（基于**`@Transactional`** 的全注解方式使用最多）

**具体实现过程：** 如果加了@Transactional注解，则会利用**事务管理器**创建一个数据库连接。并且修改数据库连接的autocommit属性为false。禁止此连接的自动提交。

然后执行当前方法，方法中会执行SQL，执行完当前方法后，如果没有出现异常就直接提交事务，如果出现了异常，并且这个异常是需要回滚的就会回滚事务，否则仍然提交事务。

## 5.1 事务传播行为

**事务传播行为是为了解决业务层方法之间互相调用的事务问题**。当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。

### **1、`TransactionDefinition.PROPAGATION_REQUIRED`**

使用的最多的一个事务传播行为，我们平时经常使用的`@Transactional`注解默认使用就是这个事务传播行为。**如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。**

`aMethod()`和`bMethod()`使用的都是`PROPAGATION_REQUIRED`传播行为的话，两者使用的就是同一个事务，**只要其中一个方法回滚，整个事务均回滚。**

```java
@Service
Class A {
    @Autowired
    B b;
    @Transactional(propagation = Propagation.REQUIRED)
    public void aMethod {
        //do something
        b.bMethod();
    }
}
@Service
Class B {
    @Transactional(propagation = Propagation.REQUIRED)
    public void bMethod {
       //do something
    }
}
```

### **2、 `TransactionDefinition.PROPAGATION_REQUIRES_NEW`**

如果我们上面的`bMethod()`使用`PROPAGATION_REQUIRES_NEW`事务传播行为修饰，`aMethod`还是用`PROPAGATION_REQUIRED`修饰的话，如果`aMethod()`发生异常回滚，`bMethod()`不会跟着回滚，因为 `bMethod()`开启了独立的事务。

### **3、 `TransactionDefinition.PROPAGATION_NESTED`**

如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`TransactionDefinition.PROPAGATION_REQUIRED`。
如果 `bMethod()` 回滚的话，`aMethod()`不会回滚。如果 `aMethod()` 回滚的话，`bMethod()`会回滚。**（就像父与子的关系）**

### **4、 `TransactionDefinition.PROPAGATION_MANDATORY`**

mandatory:强制性，最不常用的一个。如果当前存在事务，则加入该事务；如果当前没有事务，则**抛出异常。**

## 5.2 事务隔离级别

Spring 提供了以下五种事务隔离级别：

- DEFAULT：大多数数据库的默认隔离级别是 READ_COMMITTED。
- READ_UNCOMMITTED（读未提交）：允许读取未提交的数据。
- READ_COMMITTED（读已提交）
- REPEATABLE_READ（可重复读）
- SERIALIZABLE（串行化）

## 5.3 事务失效（这个非常重要）

- **方法的自调用**

Spring是基于AOP的，只要使用 **代理对象** 调用某个方法时，Spring事务才能生效，而在一个方法中调用使用 this.xxx() 调用方法时，**this并不是代理对象，所以会导致事务失效。**

解决方法：自己注入自己。

- **事务方法被非 public 修饰/事务方法被final修饰**

Spring 事务基于动态代理，如果是CGLIB动态代理的话，被修饰后，子类就没有办法重写它。

- **事务方法未被 Spring 代理**

Spring 事务是基于动态代理实现的，只有通过 Spring 代理调用的方法才会被事务管理。

- **异常未被正确捕获或抛出**

Spring 默认只对 **运行时异常（RuntimeException）和 Error 进**行回滚，对 **检查异常（Checked Exception）不会回滚。**

## 5.4 为什么加了@Transational注解后，MySQL就能实现自动回滚呢？

Spring事务管理机制与数据库的事务支持相结合。

### Step1 、事务管理器的介入

当你在方法上添加 @Transactional 注解时，**Spring 的事务管理器（如 DataSourceTransactionManager）会接管事务的创建、提交和回滚。** Spring 通过 AOP（面向切面编程）在方法调用前后织入事务管理逻辑

### Step2、 **事务的开启**

关闭自提交：默认情况下，MySQL 的 JDBC 驱动会为每个 SQL 语句开启自动提交（auto-commit=true）。

当Spring 开启事务时，**会先将数据库连接的自动提交模式关闭（auto-commit=false），从而将多个 SQL 操作绑定到同一个事务中。**

### Step3、事务的提交与回滚

**正常执行：提交事务**
如果方法执行成功（没有抛出异常），Spring 会在方法结束时调用 commit()，将事务提交到数据库。

**异常触发：回滚事务**
如果方法执行过程中抛出了未捕获的异常，**Spring 会检测到异常，并调用 rollback() 回滚事务。**
默认情况下，Spring 只对 非检查型异常（RuntimeException 及其子类）和 Error 执行回滚。
如需对检查型异常（Checked Exception）回滚，需通过 @Transactional(rollbackFor=Exception.class) 显式配置。

![image.png](Spring%20101e74500164808292f0d49a46528c08/image%2011.png)

# 6.Spring 用到了哪些设计模式？

1、工厂模式：BeanFactory

2、原型模式：原型Bean

3、单例模式：单例Bean

4、代理模式：AOP

# 7.其他

## 7.1 @Data

@Data 是 Lombok 库提供的一个非常常用的注解，**它作用于 编译阶段。**

这个注解是一个复合注解：它相当于同时添加了以下注解的功能：@Getter - 为所有字段生成 getter 方法。@Setter - 为所有非 final 字段生成 setter 方法。@ToString - 生成 toString() 方法。@EqualsAndHashCode - 生成 equals() 和 hashCode() 方法。@RequiredArgsConstructor - 生成包含 final 字段的构造方法

使用：

```java
import lombok.Data;

@Data
public class User {
    private Long id;
    private String username;
    private String password;
    private final String role;  // 只有 final 字段会包含在构造方法中
}
```

**@Data 注解极大的简化了Java实体类的编写。**
