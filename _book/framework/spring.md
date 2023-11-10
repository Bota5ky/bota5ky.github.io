### 1. ApplicationContext 和 BeanFactory 的区别

BeanFactory 是 Spring 中非常核心的组件，表示 Bean 工厂可以生成 Bean，维护 Bean，而 ApplicationContext 继承了 BeanFactory，所以 ApplicationContext 拥有 BeanFactory 所有的特点，也是一个 Bean工厂，但是 ApplicationContext 除开继承了 BeanFactoy 之外，还继承了诸如 EnvironmentCapable（获取环境变量、properties等）、MesageSoure（国际化）、ApplicationEventPublisher 等接口，从而 ApplicationContext 还有获取系统环境变量、国际化、事件发布等功能，这是 BeanFactory 所不具备的。

### 2. Spring Boot 是如何启动 Tomcat 的？

- 首先，SpringBoot 在启动时会先创建一个 Spring 容器
- 在创建 Spring 容器过程中，会利用`@Conditional0nClass`技术来判断当前 classpath 中是否存在 Tomct 依赖，如果存在则会生成一个启动 Tomcat 的 Bean
- Spring 容器创建完之后，就会获取启动 Tomcat 的 Bean，并创建 Tomcat 对象，并绑定端口等，然后启动 Tomcat

### 3. @SpringBootApplication 注解

@SpringBootApplication 注解：这个注解标识了一个 SpringBoot 工程，它实际上是另外三个注解的组合，这三个注解是

- @SpringBootConfiguration：这个注解实际就是一个 @Configuration，表示启动类也是一个配置类
- @EnableAutoConiquration：向Spring容器中导入了一个 Seletor，用来加载 ClassPath 下Springfactories 中所定的自动配置类，将这些白动加载为配置Bean
- @ComponentScan：标识扫描路径，因为默认是没有配置实际扫描路径，所以SpringBoot扫描的路径是启动类所在的当前目录

### 4. Spring 容器启动流程

- 在创建 Spring 容器，也就是启动 Spring 时：
- 首先会进行扫描，扫描得到所有的 BeanDefinition 对象，并存在一个 Map 中
- 然后筛选出非懒载的单例 BeanDefinition 进行创建 Bean，对于多例 Bean 不需要在启动过程中去进行创建，对于多例 Bean 会在每次获取 Bean 时利用 BeanDefinition 去创建
- 利用 BeanDefinition 创建 Bean 就是 Bean 的创建生命周期，这期间包括了合并 BeanDefinition、推断构造方法、实例化、属性填充、初始化前、初始化、初始化后等步骤，其中 AOP 就是发生在**初始化后**这一步骤中
- 单例 Bean 创建完了之后，Spring 会发布一个容器启动事件
- Spring 启动结束
- 在源码中会更复杂，比次如源码中会提供一些模板方法，让子类来实现，比如源码中还涉及到一些 BeanfactoryPostProcesser 和 BeanPostProcessor 的注册，Spring 的扫描就是通过 BeanFactoryPostProcessor 来实现的，依赖注入就是通过 BeanPostProcessor 来实现的
- 在 Spring 启动过程中还会去处理 @Import 等注解

### 5. Spring 事务传播机制

多个事务方法相互调用时，事务如何在这些方法间传播，方法 A 是一个事务的方法，方法 A 执行过程中调用了方法 B，那么方法 B 有无事务以及方法 B 对事务的要求不同都会对方法 A 的事务具体执行造成影响，同时方法 A 的事务对方法 B 的事务执行也有影响，这种影响具体是什么就由两个方法所定义的事务传播类型所决定。

- REQUIRED（Spring 默认的事务传播类）：如果当前没有事务，则自己新建一个事务，如果当前存在事务，则加入这个事务
- SUPPORTS：当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行
- MANDATORY：当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常
- REQUIRES_NEW：创建一个新事务，如果存在当前事务，则挂起该事务
- NOT_SUPPORTED：以非事务方式执行如果当前存在事务，则挂起当前事务
- NEVER：不使用事务，如果当前事务存在，则抛出异常
- NESTED：如果当前事务存在，则在嵌套事务中执行，否则 REQUIRED 的操作一样（开启一个事务）

### 6. Spring 事务什么时候会失效？

[Spring@Transactional 注解的 12 种失效场景](https://blog.51cto.com/u_14787961/4833414)

### 7. Spring 中的设计模式

- 工厂模式：BeanFactory、FactoryBean、ProxyFactory
- 原型模式：原型 Bean、PrototypeTargetSource、PrototypeAspectInstanceFactory
- 单例模式：单例 Bean、SingletonTargetSource、DefaultBeanNameGenerator、SimpleAutowireCandidateResolver、AnnotationAwareOrderComparator
- 构造器模式：BeanDefinitionBuilder——BeanDefinition 构造器、BeanFactoryAspectJAdvisorsBuilder - 解析并构造 @AspectJ 注解的 Bean 中所定义的 Advisor、StringBuilder
- 适配器模式：ApplicationListenerMethodAdapter——将@EventListener 注解的方法适配成 ApplicationListener、AdvisorAdapter 把 Advisor 适配成 MethodInterceptor
- 访问者模式：PropertyAccessor——属性访问器，用来访问和设置某个对象的某个属性、MessageSourceAccessor——国际化资源访问器
- 装饰器模式：BeanWrapper——比单纯的Bean对象功能更加强大、HttpRequestWrapper
- 代理模式：方式生成了代理对象的地方就用到了代理模式、AOP、@Configuration、@Lazy
- 观察者模式：ApplicationListener——事件监听机制、AdvisedSupportListener——ProxyFactory 可以提交此监听器，用来监听 ProxyFactory 创建代理对象完成事件、添加 Advisor 事件等
- 策略模式：InstantiationStrategy——Spring需要根据BeanDefinition来实例化Bean，但是员体可以选择不同的策略来进行实例化、BeanNameGenerator——beanName 生成器
- 模板方法模式：AbstractApplicationContext：postProcessBeanFactory()——子类可以继续处理 BeanFactory、onRefresh()——子类可以做一些额外的初始化
- 责任链模式：DefaultAdvisorChainFactory——负责构造一条 AdvisorChain，代理对象执行某个方法时会依次经过 AdvisorChain 中的每个 Advisor、QualifierAnnotationAutowireCandidateResolver——判断某个Bean能不能用来进行依赖注入勉强可认为也是责任链

### 8. Spring 中 Bean 是线程安全的吗？

Spring 本身并没有针对 Bean 做线程安全的处理，所以

- 如果 Bean 是无状态的，那么 Bean 则是线程安全的
- 如果 Bean 是有状态的，那么 Bean 则不是线程安全的

另外，Bean 是不是线程安全，跟 Bean 的作用域没有关系，Bean 的作用域只是表示 Bean 的生命周期范围，对于任何生命周期的 Bean 都是一个对象，这个对象是不是线程安全的，还是得看这个 Bean 对象本身。

### 9. Spring中的Bean创建的生命周期有哪些步骤

https://juejin.cn/post/6844904065457979405

Spring 中一个 Bean 的创建大概分为以下几个步骤：

- 推断构造方法
- 实例化
- 填充属性，也就是依赖注入
- 处理 Aware 回调
- 初始化前，处理 @PostConstruct 注解
- 初始化，处理 InitializingBean 接口
- 初始化后，进行 AOP

### 10. Spring 中的事务是如何实现的

- Spring事务底层是基于数据库事务和 AOP 机制的
- 首先对于使用了 @Transactional 注解的 Bean，Spring 会创建一个代理对象作为 Bean
- 当调用代理对象的方法时，会先判断该方法上是否加了 @Transactional 注解
- 如果加了，那么则利用事务管理器创建一个数据库连接
- 并且修改数据库连接的 autocommit 属性为 false，禁止此连接的自动提交，这是实现 Spring 事务非常重要的一步
- 然后执行当前方法，方法中会执行 sql
- 执行完当前方法后，如果没有出现异常就直接提交事务
- 如果出现了异常，并且这个异常是需要回滚的就会回滚事务，否则仍然提交事务
- Spring 事务的隔离级别对应的就是数据库的隔离级别
- Spring 事务的传播机制是 Spring 事务自己实现的，也是 Spring 事务中最复杂的
- Spring 事务的传播机制是基于数据库连接来做的，一个数据车连接一个事务，如果传播机制图置为需要新开一个事务，那么实际上就是先建立一个数据车连接，在此新数据库连接上执行 sql

### 11. SpringBoot 启动原理

参考自：https://www.cnblogs.com/zyly/p/13194186.html，https://www.bilibili.com/video/BV1hv4y1z7PQ/

**1. 服务构建 new SpringApplication()**

- 传入资源加载器 resourceLoader、主方法类 primarySources
- 逐一判断对应的服务类是否存在，来确定 Web 服务类型：Servlet、Reactive、None
- 加载初始化类，读取所有`META-INF/spring.factories`文件中的“注册初始化”、“上下文初始化”、“监听器”这三类配置，spring-boot和 spring-boot-autoconfigure 这两个工程中配置了 7 个“上下文初始化”和 8 个“监听器”
  - 注册初始化 BootstrapRegistryInitializer，默认为空
  - 设置上下文初始化 ApplicationContextInitializer
  - 设置监听器 ApplicationListener
- 通过“运行栈” stackTrace 推断出 main 方法所在的类

**2. 环境准备 SpringApplication.run()**

- 新建 BootstrapContext “启动上下文”，逐一调用上面加载的“启动注册初始化器” BootstrapRegistryInitializer 中的 initialize 方法
- 将“java.awt.headless”这个设置改为 true，表示缺少显示器、键盘等输入设备也可以正常启动
- 通过 prepareEnvironment 方法“组装启动参数”
  - 构造一个“可配置环境” ConfigurableEnvironment，根据不同的 Web 服务器类型会构造不同的环境
  - 加载系统环境变量、JVM 系统属性等到 propertySources 中
  - 通过 ConfigurableEnvironment 将传入的环境参数 args 进行设置
  - 在 propertySources 集合首位添加 configurationProperties 空配置
  - 发布“环境准备完成”事件
  - 刚加载的 8 个 Listener 会监听到这个事件，其中的部分监听器会进行相应（串行）处理，例如EnvironmentPostProcessorApplicationListener 会去加载 spring.factories 配置文件中“环境配置后处理器
    “ EnvironmentPostProcessor
  - 考虑到刚创建的“可配置环境”在一系列过程中可能会有变化做的补偿，通过二次更新保证匹配
- 将”spring.beaninfo.ignore“设为 true，表示不加载 Bean 的元数据信息，打印 banner

**3. 容器创建 ApplicationContext**

- createApplicationContext()，根据服务类型创建”容器“ ConfigurableApplicationContext，对应 SERVLET 服务在过程会构造以下内容：
  - Bean 工厂
  - 用来解析@Component、@ComponentScan 等注解的“配置类后处理器 ConfigurationClassPostProcessor
  - 用来解析@Autowired、@Value、@Inject 等注解的“自动注解 Bean 后处理器 AutowiredAnnotationBeanPostProcessor 等在内的属性对象

- 通过 prepareContext 方法对容器中的部分属性进行初始化
  - 用 postProcessApplicationContext 方法设置”Bean名称生成器 beanNameGenerator“、”资源加载器 resourceLoader"、”类型转换器 ConversionService“等
  - 执行之前加载的 ApplicationContextInitializer，容器 ID、警告日志处理、日志监听都是在这里实现的
  - 发布“容器准备完成”事件
  - 为容器注册“启动参数”、“Banner”、“Bean 引用策略”和“懒加载策略”等等
  - 通过 Bean 定义加载器将“启动类”在内的资源加载到 BeanDefinitionMap 中
  - 发布“资源加载完成”事件


**4. 填充容器**

这个过程也被称为“自动装配”，包含“Bean 生命周期”管理和 Tomcat 启动：
- prepareRefresh：准备 servlet 相关的环境
- obtainFreshBeanFactory：Springboot 无实际逻辑，Spring 会重新构造 beanfactory，加载 BeanDefinition
- prepareBeanFactory：
  - 准备“类加载器” BeanClassLoader、“表达式解析器” BeanExpressionResolver、“配置文件处理器” PropertyEditorRegistrar 等系统级处理器
  - 以及两个“Bean 后置处理器”：用来解析 Aware 接口的 ApplicationContextAwareProcessor、用来处理自定义监听器注册和销毁的 ApplicationListenerDetector
  - 同时会注册一些”特殊 Bean“和“系统级 Bean“：比如容器本身 BeanFactory 和 ApplicationContext、系统环境 environment（特殊对象池）、系统属性 systemProperties（单例池） 等
- postProcessBeanFactory：对 BeanFactory 进行额外设置或修改
  - 主要定义了包括 request、session 在内的 Servlet 相关作用域 Scopes
  - 注册与 Servlet 相关的特殊 Bean：包括 ServletRequest、ServletResponse、HttpSession 等
- invokeBeanFactoryPostProcessors
  - 执行各种 BeanFactory 后置处理器，其中最主要的就是用来加载所有”Bean定义“的”配置处理器“ ConfigurationClassPostProcessor，通过它加载所有 @Configuration 配置类
  - 通过 Bean 扫描器将所有扫描出来的”Bean 定义“（包括加了 @Bean、@Import 等注解的类和方法）都放到 BeanDefinitionMap 中
- registerBeanPostProcessors：检索所有 Bean 后置处理器，根据指定 order 进行排序，放入后置处理器池 beanPostProcessors 中，每一个后置处理器都会在 bean 初始化之前和之后分别执行对应的逻辑
- 会通过 initMessageSource 和 initApplicationEventMulticaster 方法，从单例池中获取两个非常实用的 Bean放在 ApplicationContext 中，一个是用于国际化，名为“messageSource”的 Bean，另一个是用于自定义广播事件，名为“applicationEventMulticaster”的 Bean，有了它就可以通过 publishEvent 方法进行事件的发布了
- 通过 onRefresh 构造并后动 Web 服务器，先查找实现了 ServletWebServerFactory 这个接口的应用服务器Bean，接下来通过 getWebServer 方法构造一个 Tomcat 对象并启动
- 通过 registerListeners 方法，在 bean 中查找所有的“监听器 Bean”，将它们注册到第八步构造的“消息广播器” applicationEventMulticaster 中
- 通过 finishBeanFactorylnitialization 来生产我们所有的 Bean，整体分为“构造对象、填充属性、机始化实例、注册销毁”四个步骤
- 通过 finishRefresh 方法构造并注册”生命周期管理器 lifecycleProcessor，同时会调用所有实现了“生命周期接口”Lifecycle 的 Bean 中的 start 方法，当然在容器关闭时也会自动调用对应的 stop 方法，发布“容器刷新完成”事件

### 12. 如何理解 SpringBoot 的 Starter 机制

- 提供预配置的依赖：Spring Boot Starters 是一组预配置的依赖项集合，用于启用特定类型的功能。例如，你可以使用 spring-boot-starter-web 启用 Web 应用程序相关的功能，包括内嵌的 Web 服务器、Spring MVC、Jackson 等。这样，你只需引入这个Starter，而无单独处理每个依赖项的版本和配置。

- 简化依赖管理：Spring Boot Starters 简化了项目的依赖管理。通过引入适当的 Starter 你不需要手动指定每个相关的依赖项，Spring Boot 会自动处理它们的版本兼容性和配置。这有助于避免版本冲突和配置错误。

- 自动配置：Spring Boot Starters 还包含了与功能相关的自动配置类。这些自动配置类根据应用程序的依赖和配置，自动配置了必要的组件和设置。这使得你可以快速地启用和使用功能，而无需手动配置每个组件。
  
- 遵循约定：Spring Boot Starters 遵循约定大于配置的原则，它们约定了如何将依敕和配置组合在一起，使得应用程序开发更加一致和高效。

- 自定义扩展：尽管 Spring Boot Starters 提供了默认的依赖和配置，你仍然可以根需要进行自定义扩展。你可以在自己的项目中覆盖默认的配置，添加额外的依赖项，或者通过属性配置文件来修改 Starter 的行为。

总之，Spring Boot Starter 机制是 Spring Boot 框架的一项重要功能，它通过提供预配置的依赖和自动配置，大大简化了项目的依赖管理和配置过程，使开发人员能够更专注于业务逻辑的实现。
