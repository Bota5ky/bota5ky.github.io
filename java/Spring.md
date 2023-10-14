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
- 在源妈中会更复杂，比次如源码中会提供一些模板方法，让子类来实现，比如源码中还涉及到一些 BeanfactoryPostProcesser 和 BeanPostProcessor 的注册，Spring 的扫描就是通过 BeanFactoryPostProcessor 来实现的，依赖注入就是通过 BeanPostProcessor 来实现的
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
- 模板方法模式：AbstractApplicationContext：postProcessBeanFactory()——子类可以继续处理 BeanFactory
  、onRefresh()——子类可以做一些额外的初始化
- 责任链模式：DefaultAdvisorChainFactory——负责构造一条 AdvisorChain，代理对象执行某个方法时会依次经过 AdvisorChain 中的每个 Advisor、QualifierAnnotationAutowireCandidateResolver——判断某个Bean能不能用来进行依赖注入勉强可认为也是责任链

### 8. Spring 中 Bean 是线程安全的吗？

Spring 本身并没有针对 Bean 做线程安全的处理，所以

- 如果 Bean 是无状态的，那么 Bean 则是线程安全的
- 如果 Bean 是有状态的，那么 Bean 则不是线程安全的

另外，Bean 是不是线程安全，跟 Bean 的作用域没有关系，Bean 的作用域只是表示 Bean 的生命周期范围，对于任何生命周期的 Bean 都是一个对象，这个对象是不是线程安全的，还是得看这个 Bean 对象本身。

### 9. Spring中的Bean创建的生命周期有哪些步骤

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

