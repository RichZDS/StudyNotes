## 什么是循环依赖

多个Bean 相互注入 相互依赖 那么就到底先加载谁？？？ 就卡住了 报错了

## 如何解决循环依赖

关键**`提前暴露还未创建完成的Bean`**

 方法:**三级缓存**

1. 一级缓存	
   - 用于存储**完全初始化**的完整单例化的Bean
2. 二级缓存
   - **尚未*完全初始化*** 但是**已经实例化**的Bean ，用于**提前暴露对象**，**避免循环依赖问题**
3. 三级缓存
   - 用于存储对象工厂 当需要时 可以通过工厂创建早期的Bean（特别是为了支持AOP代理对象的创建）



解决步骤：

- Spring先**创建Bean的实例** 将其***加入三级缓存***中
- 当一个Bean需要依赖另一个未初始化的Bean的时候 Spring会从**三级缓存中获取Bean工厂**，并**生成该对象**
- 代理对象**存入二级缓存**，解决依赖
- 一旦所有依赖Bean被完**全初始化**，Bean将**转移到一级缓存**中



### 扩展

### 只有是依赖的**Bean是单例**

什么是单例模式 就是一个Bean他只能有一个实例化

那么因此Bean1需要2的时候 就创建B B创建的时候需要A 那么正好

但是如果是原型模式的话 那么A需要B B需要A的时候就会创建A2 但是A2也需要B 因此就会创建B2 .....循环往复 周而复始



<img src="F:\记录\八股文\image-20251024154932330.png" alt="image-20251024154932330" style="zoom:50%;" />

### 并且 必须`不全是`构造器注入  && beanName字母序在前的不能是构造器注入

背景：Spring 创建Bean分为三部

1. 实例化 --》createBeanInstance 就是new了一个对象
2. 属性注入 --》populateBean 就是set一些属性
3. 初始化 --》 initializeBean 执行一些aware接口中的方法 initMethod AOP代理



所以 当我们全部都使用构造器注入的时候 newA(B b) 就回去执行 new B(A a) 但是tmdA没有构造完啊 那他妈怎么办 

还能怎么办 换一种思路 那就是set注入

<img src="F:\记录\八股文\image-20251024160334418.png" alt="image-20251024160334418" style="zoom:50%;" />

###  那么我们再次回到创建Bean的三级缓存中 重新回顾一下

1. 首先获取单例的Bean 的时候会通过BeanName去找一级缓存 查找完整的Bean 如果找到就返回 找不到就去2
2. 看看对应的Bean是否在创建中，如果不在直接返回找不到NULL 如果找到 则去二级缓存中找BEAN 找到就返回 没有就3
3.  去三级缓存中 通过BeanName找到对应的工厂 如果有工厂则创建Bean 且放到earlySingletonObjects中
4. 三级缓存都没有找到 那么就返回null



我们发现在2中如果Bean没有创建 则是返回null 那么返回了null 以后呢？？就是去创建这个Bean了 就调用 createBean 创建Bean 调用的方法就是docreateBean 那么这个docreateBean 是干什么的呢 就是`` 实例化 初始化 属性注入``



### 在实例化了一个Bean以后 

就会在**addSingletonFactory**中塞入一个工厂  然后这个工厂调用一个方法addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean)); 我们就可以得到这个Bean

要注意，此时 Spring 是不知道会不会有循环依赖发生的，但是它不管，反正往 singletonFactories 塞这个工厂，这里就是`提前暴露`。 重点就是在对象实例化之后，都会在三级缓存里加入一个工厂，提前对外暴露还未完整的 Bean，这样如果被循环依赖了，对方就可以利用这个工厂得到一个不完整的 Bean，破坏了循环的条件。

### Tip:

![image-20251024161935634](F:\记录\八股文\image-20251024161935634.png)





### Spring 二级缓存行不行 不要三级了

先说原因：AOP代理和Bean早期的引用问题

如果只是解决循环依赖 那么很简单 二级就可以 但是在AOP的时候 直接使用二级而不做任何处理 会导致我们拿到的Bean 是 **未代理的原始对象** 如果二级都存的是这个 就违反了Bean的生命周期 

破化了**Bean 初始化流程中 “代理对象生成” 与 “依赖注入” 的衔接环节**

**若 Bean 需要 AOP 代理，最终暴露给容器和其他依赖 Bean 的必须是 “代理对象”，而非原始对象**

## Bean的生命周期

`实例化` --》 `属性注入`--》 初始化前的扩展机制--》 初始化前--》 `初始化`--》 初始化后--》 `使用`--》`销毁`

实例化： SPring容器 根据配置文件或注解实例化Bean对象

属性注入： Spring将依赖（setter/字段注入） 注入到 Bean实例中

初始化前的扩展机制： if(Bean对象实现了BeanNameAware... aware接口)则执行aware注入 

初始化前： 可以通过BeanPostProcessor接口对Bean进行一些额外的处理 比如可以获取Bean的名字 工厂 ApplicationContext等容器资源

初始化： - `InitializingBean` 接口提供了一个 `afterPropertiesSet` 方法，用于在 Bean 的所有属性设置完成后执行一些自定义初始化逻辑。开发者也可以通过 `@PostConstruct` 注解或者 XML/Java 配置中的 `init-method` 属性，来指定初始化方法。

初始化后：

使用Bean：

销毁：调用DisposiableBean  接口提供了destory方法 	

<img src="F:\记录\八股文\image-20251024164026531.png" alt="image-20251024164026531" style="zoom:67%;" />

## SPring 核心特性

Spring框架核心特性包括：

- **IoC容器**：Spring通过控制反转实现了对象的创建和对象间的依赖关系管理。开发者只需要定义好Bean及其依赖关系，Spring容器负责创建和组装这些对象。
- **AOP**：面向切面编程，允许开发者定义横切关注点，例如事务管理、安全控制等，独立于业务逻辑的代码。通过AOP，可以将这些关注点模块化，提高代码的可维护性和可重用性。
- **事务管理**：Spring提供了一致的事务管理接口，支持声明式和编程式事务。开发者可以轻松地进行事务管理，而无需关心具体的事务API。
- **MVC框架**：Spring MVC是一个基于Servlet API构建的Web框架，采用了模型-视图-控制器（MVC）架构。它支持灵活的URL到页面控制器的映射，以及多种视图技术。

但是也可以这么分

`核心容器`

- SpringCore 包含了`DI 依赖注入 & Ioc 控制反转`
- SpringBean 管理Bean的定义和生命周期
- SpringContext 基于Core 和Bean的高级容器
- SpringExpressionLanguage 表达式语言

`Aop(切面编程)`

- SPringAop 

`数据访问`

- SPringJDBC
- SpringORM :（mybatis就是这个管着）
- SpringTransaction` 事务管理`

`Web层`

- SpringWeb 提供基础的Web开发支持
- SpringMVC 实现了MCV model-controller-view 用于构建Http请求的Web应用
- SpringWebFlux



## Spring 的核心特征

DI IOC AOP

<img src="F:\记录\八股文\image-20251024183530002.png" alt="image-20251024183530002" style="zoom:50%;" />

## 什么是SPring Ioc

**def**:Inversion of Control 控制反转 ioc是通过DI实现的 IOC让对象的**创建与管理职责由容器负责**，而不是由对象自身控制

`核心思想`： 对象的创建和依赖不是由对象本身实现的 而是由Spring容器管理的

好处： `灵活性 解耦`

DI （依赖注入dependency injection ）通过`构造器、setter方法、接口注入` 将对象的依赖项传递给他 而不是对象自行创建依赖.



扩展：IOC 只是一个思想

控制的是什么？？ 控制的是` 对象的创建 `

 反转的是什么？？ 反转的是 `创建对象且注入依赖对象的这个动作` 

反转之前 是我们在一个类里面new一个类 这个类的创建是我们创建的

而现在类的生命周期是由Spring控制的了

ex：Spring容器使用了工厂模式为我们创建了所需要的对象，我们使用时不需要自己去创建，直接调用Spring为我们提供的对象
即可，这就是控制反转的思想。



**好处**

例如，对象 A 需要依赖一个实现 B，但是对象都由 IOC 控制之后，我们不需要明确地在对象 A 的代码里写死依赖的实现 B，只需要写明依赖一个接口，这样我们的代码就能顺序的编写下去。 然后，我们可以在配置文件里定义 A 依赖的具体的实现 B，根据配置文件，在创建 A 的时候，IOC 容器就知晓 A 依赖的 B，这时候注入这个依赖即可。 如果之后你有新的实现需要替换，那 A 的代码不需要任何改动，你只需要将配置文件 A 依赖 B 改成 B1，这样重启之后，IOC 容器会为 A 注入 B1。



## Spring IOC容器初始化过程

### 启动阶段

- 配置加载：加载配置文件或者配置类 
- 创建容器：Spring 创建IOC容器（Beanfactory ApplicationContext）准备加载和管理Bean文件

### Bean注册阶段

- 解析和注册：BeanDefinitionReader读取配置中的Bean定义，并将其注册到容器中，形成BeanDefinition对象

### 实例化和依赖注入

- 实例化： 根据BeanDefinition创建Bean实例
- 依赖注入：根据BeanDefinitio中的依赖关系，可以通过 构造注入 setter注入 或者字段注入 将依赖注入到Bean里面

### 初始化

- BeanPostProcessor处理 这些处理器会在Bean初始化生命周期中加入定义的处理逻辑
- Aware接口调用 如果Bean实现了Aware接口（BeanNameAware BeanFactoryAware） Spring会回调这些接口，传递容器相关的信息
- 初始化方法调用：调用Bean的初始化方法



## 什么是Bean

任何通过Spring容器实例化的 组装和管理的JAVA对象都可以称为bean

**Bean定义方式**

1. XML ->就是

   ```xml
   <bean id="mobian3" class="pers.mobian.springseventh.MoBian3XML">
       id就是你给他定义的名字 class就是路径名字
   ```

   

2. 注解

   ```java
   @C@Component
   @Controlle
   @Service
   @Repository
   ```

   

3. java配置类

```java
@Configuration
@ComponentScan("pers.mobian.firstTest")
public class ContextConfig {
}


@Configuration
public class Config {

	@Bean(name = "mobian666")
	public MoBian moBian(){
		return new MoBian();
	}
}
```



## 什么是BeanFactory

Bean工厂 创建和管理Bean的 底层就是Ioc容器 

负责从配置源 XML java配置类 注解之类的读取Bean的定义 负责她的生命周期

**延迟加载** -》延迟初始化  只有在Bean首次请求的时候才会实例化这个Bean 而不是程序已启动就开始



## FactoryBean

FactoryBean 可以自定义任何所需的初始化逻辑，生产出一些定制化的 bean，简单来说就是 可以封装Bean

**FactoryBean 与普通 Bean 的区别** 

- **创建逻辑不同**：普通的 Bean 直接由 Spring 容器管理，而 FactoryBean 通过自定义的 `getObject()` 方法创建实际的对象。
- **动态代理和复杂对象**：FactoryBean 适用于创建动态代理或复杂的 Bean，而普通 Bean 通常只处理简单的对象创建。



想要成为FactoryBean需要实现

```java
public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory> {
    @Override
    public SqlSessionFactory getObject() throws Exception {
        // 创建并配置 SqlSessionFactory（如加载 MyBatis 配置、数据源等）
        return new SqlSessionFactoryBuilder().build(config);
    }
}

```





## Bean的作用域

**singleton**:默认是单例，含义不用解释了吧，一个IoC容器内部仅此一个 

**prototype**:原型，多实例 

**request**:每个请求都会创建一个属于自己的Bean实例，这种作用域仅存在Spring Web应用中 

**session**:一个Http Session 中有一个bean的实例，这种作用域仅存在Spring Web应用中 

**application**:整个ServletContext生命周期里，只有一个bean,这种作用域仅存在Spring Web应用中

**websocket**: 一个WebSocket生命周期内一个bean实例，这种作用域仅存在Spring Web应用中

## 什么是AOP(切面编程)

是什么？： 其实就是将与业务无关的一些东西 比如（log 权限校验 安全检查 之类的）与业务逻辑分开 

**AOP（面向切面编程）的核心**是将日志、事务、权限等与业务逻辑无关的**横切关注点**从业务代码中抽取出来，通过声明式的方式动态应用到业务方法中，避免这些逻辑直接嵌入业务代码导致的冗余与耦合。

**主要组成部分** 切面 连接点 通知 切入点 织入



**AOP是通过动态代理来实现的**

动态代理：运行时 将切面的逻辑放入

静态代理：编译期间/类加载期间



Aop的核心概念

- **Aspect**：切面，只是一个概念，没有具体的接口或类与之对应，是 Join point，Advice 和 Pointcut 的一个统称。

  - ```java
    @Aspect
    public class LoggingAspect {
      @Before("execution(* com.example.service.*.*(..))")
      public void logBefore() {
          System.out.println("Logging before method execution");
      }
    }
    
    ```

    

- **Join point**：连接点，指程序执行过程中的一个点，例如方法调用、异常处理等。在 Spring AOP 中，仅支持方法级别的连接点。

- **Advice**：通知，即我们定义的一个切面中的横切逻辑，有“around”，“before”和“after”三种类型。在很多的 AOP 实现框架中，Advice 通常作为一个拦截器，也可以包含许多个拦截器作为一条链路围绕着 Join point 进行处理。

- **Pointcut**：切点，用于匹配连接点，一个 AspectJ 中包含哪些 Join point 需要由 Pointcut 进行筛选。

  - ```java
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}
    ```

- **Introduc**

- **tion**：引介，让一个切面可以声明被通知的对象实现任何他们没有真正实现的额外的接口。例如可以让一个代理对象代理两个目标类。

- **Weaving**：织入，在有了连接点、切点、通知以及切面，如何将它们应用到程序中呢？没错，就是织入，在切点的引导下，将通知逻辑插入到目标方法上，使得我们的通知逻辑在方法调用时得以执行。



## SpringAOP 默认是什么动态代理

Spring Framework 默认的是 JDK动态代理 SB 2x了以后 就是CGLIB

区别

**代理方式**

- JDK动态代理 基于**接口实现** 通过Proxy动态生成代理类
- CFLIB动态代理**基于类继承**，通过字节码技术生成目标类的子类

**使用场景**

- JDK动态代理 推荐用于`代理接口`的场景 适合代理的类实现了接口 
- CGLIB动态代理 适合没有接口的类。`基于继承实现的`



## Spring 拦截连的实现

什么是拦截链 指一系列拦截器以此作用于请求或方法调用 实现横切关注点的处理

**主要方面**

- `HandlerInterceptor `（MVC拦截器） 用于拦截HTTP 请求并经行处理 实现`preHandle postHandle afterCompletion` 来实现拦截前中后 的方法
- `Filter` 基于servlet API 的过滤器 用于处理 跨域 安全验证 编码过滤 通过*`doFilter`*方法拦截请求
- `Aop切面` Spring AOP 提供的方法级别的拦截，通过定义切面（Aspect）可以实现方法的前后处理。切面中的` @Before、@After、@Around` 等注解用于控制拦截的执行顺序。

这个底层的源码就是

有一个chain集合 这个集合里面存放着所有的interceptor的方法  然后每次通过index++来调取下一个方法 这个方法 被cglibmethodinvocation封装起来在proceed方法





## AOP与AspectJ有什么区别

- Aop 基于**动态代理** 在**运行时**的代理机制

  - 轻量级 的SpringBean容器
  - 适合大部分的业务场景 尤其是简单的AOP功能的Spring应用

  AspectJ 更加强大

  - 支持编译时 类加载时 运行时的AOP功能
  - 支持更加灵活的切点和增强操作，提供编译期和加载期的织入方式，性能较高。
  - 适合对性能要求较高或需要复杂切点匹配的场景，如日志、监控等。

 

## SpringMVC

M model 模型

V view 试图

C controller 控制

核心就是**DispatchServlet** 前端控制器。它通过注解、配置等方式 将Http的请求 映射到控制器方法中去 然后由Controller处理请求逻辑 并将数据返还给View层去渲染



Spring MVC 的工作流程可以分为以下几个关键步骤:

**客户端请求**：浏览器向服务器发送 HTTP 请求。 

**DispatcherServlet**：所有的请求首先由 Spring MVC 的核心前端控制器 `DispatcherServlet` 接收，它充当整个流程的调度中心。

- ```java
  @Controller --> 标记控制类
  public class MyController {
    @RequestMapping("/login") --> 方法和url的映射关系
    public String mianshiya() {
        return "哈哈哈";
    }
  }
  
  ```

  

**处理器映射（Handler Mapping）**：`DispatcherServlet` 根据请求的 URL 使用处理器映射器找到对应的控制器（Controller）。

**控制器（Controller）**：控制器接收请求并处理业务逻辑，通常通过注解 `@Controller` 和 `@RequestMapping` 定义请求的映射方法。

**模型和视图（ModelAndView）**：控制器处理完业务逻辑后，将数据封装到模型对象中，并指定返回的视图名称。 

**视图解析器（ViewResolver）**：`DispatcherServlet` 调用视图解析器，将逻辑视图名称解析为实际的视图，如 JSP、Thymeleaf 等模板引擎。 

- ```java
  @Bean
  public InternalResourceViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    return resolver;
  }
  //InternalResourceViewResolver等等就是一个视图解析器
  
  
  @RequestMapping("/user")
  public String getUser(@RequestParam("id") String userId) {
    // 使用请求参数
  }
  
  @RequestMapping("/user/{id}")
  public String getUserById(@PathVariable("id") String userId) {
    // 使用路径变量
  }
  //@RequestParam获取请求参数
  //@PathVariable 获取url的路径的变量
  ```

  

**视图渲染**：视图渲染引擎根据模型中的数据生成 HTML 页面并返回给客户端。



### 表单处理 & 数据绑定

数据绑定 就是通过`@ModelAttribute` 自动的将表单数据绑定到数据模型上面 

```java
@RequestMapping("/submitForm")
public String submitForm(@ModelAttribute("user") User user) {
  // 处理表单提交的数据
  return "result";
}

```

表单校验

通过 `@Valid` 和`BindingResult` 可以处理表单之前就进行数据数据处理

```java
@RequestMapping("/submitForm")
public String submitForm(@Valid @ModelAttribute("user") User user, BindingResult result) {
  if (result.hasErrors()) {
      return "form";
  }
  return "result";
}

```

### 全局异常处理

通过` @ControllerAdvice `和 ` @ExceptionHandler(Exception.class)` 处理全局的异常

```java
@ControllerAdvice
public class GlobalExceptionHandler {
  @ExceptionHandler(Exception.class)
  public String handleException(Exception ex) {
      // 处理异常
      return "error";
  }
}
```

<img src="F:\记录\八股文\image-20251025211652863.png" alt="image-20251025211652863" style="zoom:50%;" />



## Spring中的设计模式

<img src="F:\记录\八股文\image-20251025213007518.png" alt="image-20251025213007518" style="zoom: 50%;" />

<img src="F:\记录\八股文\image-20251025213031184.png" alt="image-20251025213031184" style="zoom: 50%;" />

## 脏读 不可重读 幻读

脏读（Dirty Read）：一个事务**读取**了**另一个尚未提交**的事务的数据，如果该事务回滚，则数据是不一致的。

不可重复读（Non-repeatable Read）：因为**其他事务修改了该数据并提交**，所以 在同一事务内的多次读取，前后数据不一致，。 

幻读（Phantom Read）：在一个事务内的多次查询，查询结果集不同，因为其他事务插入或删除了数据。



不可重复读 是在读的事务中，其他事务update提交了。

幻读 是在读的事务中，其他事务进行了insert或者delete。

## 事务隔离级别

### DEFAULT(默认)

 默认通常为默认

### READ_UNCOMMITTED(读未提交)

- 最低的隔离级别
- 允许事务**读取**`尚未提交`的数据
- 可能会导致脏读、不可重读、幻读

### READ_COMMITTED(读已提交)

- 仅次于上面那位
- 仅允许读取已经提交的数据
- `避免了脏读` 但是还是**不可重读、幻读**有可能实现

### PEREATABLE_RAED(可重复读)

- 仅次于上面那位
- 确保在同一个事务内的``多次读取结果一致``
- `避免脏读和不可重复读`，但可能会有**幻读问题**



### SERIALIZABLE(可串行化)

- 最屌的那位
- 通过强事务按顺序执行 
- 避免了所有的问题

<img src="F:\记录\八股文\image-20251027170227931.png" alt="image-20251027170227931" style="zoom: 50%;" />







```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void processTransaction() {
    // 事务逻辑
}
```





## 事务的传播级别

前言：什么是传播？

就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行。比如说，A事务方法调用了B事务方法，B是继续在调用者A的事务中运行呢？还是为自己另开一个新事物运行？ 这就是由B的事务传播行为决定的。

正常来说有几种解决方案:

1. 融入事务：直接去掉 serviceB 中关于开启事务和提交事务的 begin 和 commit，融入到 serviceA 的事务中 (直接用 ServiceA 的事务)。问题: B 事务的错误会引起 A 事务的回滚。
2. 挂起事务：如果不想 B 事务的错误引起 A 事务的回滚，可以开启两个连接，一个执行 A 一个执行 B，互不影响，执行到 B 的时候把 A 挂起新起连接去执行 B，B 执行完了再唤醒 A 执行。
3. 嵌套事务: MySQL 中可以通过给 B 事务加 savepoint 和 rollback 去模拟嵌套事务，把 B 设置成伪事务。

spring 中的事务传播行为:

1. PROPAGATION_REQUIRED (需要):
   -  需要事务。
   - 如果存在一个事务，则直接使用当前事务。如果则开启一个新的事务。(A 如果存在事务，则 B 融入 A 事务，如果没有则新起一个事务)
2. SUPPORTS (支持): 
   - 支持 A 的事务。
   - 如果存在一个事务，支持当前事务。如果则非事务的执行。(A 有，则 B 融入，A 没有，则非事务执行)
3. MANDATORY (强制性):
   -  强制什么？
   - 强制必须有事务。如果已经当前存在一个事务，使用当前事务。没有则抛出异常。(A 有，则 B 融入，A 没有，则抛异常)
4. REQUIRES_NEW (需要新的): 
   1. 如果一个事务已经存在，则先将这个存在的事务挂起。
   2. 如果没有，则新起一个事务执行。(A 有，则将 A 事务挂起，新开一个事务来执行 B，执行完后，唤醒 A，继续执行；A 没有事务则新起一个事务执行 B)
5. NOT_SUPPORTED (不支持): 
   1. 不支持什么？不支持事务。
   2. 总是非事务地执行，并挂起任何存在的事务。(A 有，则挂起 A，B 非事务执行，执行完之后，唤醒 A，A 继续执行，A 还是以事务形式执行的)
6. NEVER (从不): 
   1. 总是非事务地执行，
   2. 如果存在一个活动事务，则抛出异常。(A 有，则抛异常)
7. NESTED (嵌套的): 
   1. 如果一个活动的事务存在，则运行在一个嵌套的事务中。
   2. 如果没有活动事务，则按 TransactionDefinition.PROPAGATION_REQUIRED 属性执行。(A 有，则 B 用 savepoint 方式嵌套执行与 A)

## 事务的传播有什么用

1. 控制事嵌套和传播
2. 确保独立操作的事务隔离
3. 控制事务的边界的一致性



<img src="F:\记录\八股文\image-20251027180538746.png" alt="image-20251027180538746" style="zoom: 50%;" />



## SpringAOP技术的相关术语

### 切面

切面是将通知应用到切入点的过程。切面通常由通知和切入点组成，用于定义增强逻辑的执行范围。

```java
@Aspect
public class LoggingAspect {

    @Before("addPointcut()")
    public void logBeforeAdd() {
        System.out.println("前置通知：方法执行前");
    }

    @After("addPointcut()")
    public void logAfterAdd() {
        System.out.println("后置通知：方法执行后");
    }
}
```



### 通知

通知是指在连接点上执行的增强逻辑。

- **前置通知**：在方法执行之前执行。

- **后置通知**：在方法执行之后执行。

- **环绕通知**：在方法执行前后都可以执行。

- **异常通知**：在方法抛出异常时执行。

- **最终通知**：在方法正常返回时执行。

- ```java
  @Around("addPointcut()")
  public Object aroundAdd(ProceedingJoinPoint joinPoint) throws Throwable {
      System.out.println("环绕通知：方法执行前");
      Object result = joinPoint.proceed(); // 执行目标方法
      System.out.println("环绕通知：方法执行后");
      return result;
  }
  ```

  

### 切点

切入点是指我们实际增强的连接点。在Spring AOP中，切入点是通过表达式定义的，用于指定哪些连接点需要被增强。

```java
@Pointcut("execution(* com.example.User.add(..))")
public void addPointcut() {}

@Pointcut("execution(* com.example.User.delete(..))")
public void deletePointcut() {}
```



### 连接点

连接点是指程序执行过程中可以被增强的点。在Spring AOP中，连接点通常是指方法的执行点。

假设我们有一个`User`类，其中包含四个方法：`add()`、`update()`、`select()`和`delete()`。这些方法都可以被增强，因此它们都是连接点。

```java
public class User {
    public void add() {
        System.out.println("执行添加方法");
    }

    public void update() {
        System.out.println("执行更新方法");
    }

    public void select() {
        System.out.println("执行查询方法");
    }

    public void delete() {
        System.out.println("执行删除方法");
    }
}
```

### 目标对象

给谁加 就谁是目标对象 很好理解

### 代理

- JDK动态代理
- CGLIB代理

### 织入

织入是将切面应用到目标对象的过程。织入可以发生在编译时、类加载时或运行时。在 Spring 中，AOP 的织入通常发生在运行时。

- 使用 `@Aspect` 注解标记的类，会在运行时通过代理方式将切面织入目标类。





| 问题                           | 答案                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| Q1: 什么是连接点？             | A1: 连接点是指程序执行过程中可以被增强的点，通常是指方法的执行点。 |
| Q2: 切入点和连接点有什么区别？ | A2: 连接点是程序中所有可以被增强的点，而切入点是指实际增强的连接点。 |
| Q3: 通知有哪些类型？           | A3: Spring AOP支持五种类型的通知：前置通知、后置通知、环绕通知、异常通知、最终通知。 |
| Q4: 切面的作用是什么？         | A4: 切面是将通知应用到切入点的过程，用于定义增强逻辑的执行范围。 |
| Q5: 如何定义切入点？           | A5: 通过Spring AOP的表达式语言（如`execution()`）定义切入点。 |







## SPringBean 注册到容器的方式

- XML

- @Compoenet

  - @Service@Controller@Repository是@Component行生注解

- @Configuration 和 @Bean

  - ```java
    @Configuration//声明配置类
    public class AppConfig {
    
        @Bean
        public MyBean myBean() {
            return new MyBean(myDependency());
        }
    
        @Bean
        public MyDependency myDependency() {
            return new MyDependency();
        }
    }
    
    ```

    

- @Import注解

- ```java
  @Configuration
  @Import(MyComponent.class)
  public class MainConfig {
      // 当前配置类逻辑
  }
  
  public class MyComponent {
      // 普通类逻辑
  }
  
  ```

  



## 自动装配

### no 默认装配

默认方式 不进行自动装配 依赖关系必须显式定义

```xml
<bean id="myBean" class="com.example.MyBean">
    <property name="myDependency" ref="myDependency"/>
</bean>
<bean id="myDependency" class="com.example.MyDependency"/>

```



### byName

根据名字定义

```xml
<bean id="myBean" class="com.example.MyBean" autowire="byName"/>
<bean id="myDependency" class="com.example.MyDependency"/>

```



### byType

要求容器中有且仅有一个符合类型的Bean

```xml
<bean id="myBean" class="com.example.MyBean" autowire="byType"/>
<bean id="myDependency" class="com.example.MyDependency"/>

```



### constructor

```xml
<bean id="myBean" class="com.example.MyBean" autowire="constructor"/>
<bean id="myDependency" class="com.example.MyDependency"/>

```



## @Conponent @Controller @Service @Repository

 @Controller @Service @Repository都是@Conponent的衍生注解

@component:这是一个通用的注解，用于将任意类注册为 Spring 容器中的 Bean。它没有特定的语义，适用于任何需要 Spring 管理的类

@contro11er:这是一个专门用于 Spring MVC 中的控制器(Controler)层的注解。用于处理 HTTP 请求，并将结果返回给客户端。

@service:用于标识业务逻辑层的类。它具有明确的语义，表明该类承担业务操作，主要用于编写服务类
@Repository:用于数据访问层(DAO)的类，与数据库交互。

```java
@Repository
public class UserRepository {

    public User findUserById(Long id) {
        // 从数据库中查询用户
        return new User("John", "Doe");
    }
}

```



## Spring启动过程

1. 加载配置文件 初始化文件
2. 实例化文件
   - 根据配置中的信息创建文件 ApplicationContext 
   - 实例化BeanFactory 并且加载容器中的BeanDefination
3. 解析BeanDefination
4. 实例化Bean
5. 注入依赖
6. 处理Bean的生命周期
7. 处理BeanPostProcessor
8. 发布事件
9. 完成启动





