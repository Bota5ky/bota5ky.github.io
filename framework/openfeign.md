### 1. 注册 FeignClient 的入口

@Import 的三种使用方式

**1.1 Import \<JavaObject\>**

```java
@Import(ImportJavaObject.class)
```

几种引入 bean 的验证方式

```java
//1.通过AnnotationConfigApplicationContext
var context = new AnnotationConfigApplicationContext(ImportBeanSample.class);
var importJavaObject = context.getBean(ImportJavaObject.class);
//2.通过BeanDefinition
var beanDefinition = context.getBeanDefinition(ImportJavaObject.class.getCanonicalName());
importJavaObject = context.getBean(ImportJavaObject.class.getCanonicalName(),ImportJavaObject.class);
//3.获取报错不存在
importJavaObject = context.getBean("importJavaObject",ImportJavaObject.class);
```

**1.2 实现 ImportSelector 接口**

```java
@ComponentScan(basePackages = "cn.chenhy.microsoft.service.consumer.importbean")
```

实现 selectImports 方法

```java
@Override
public String[] selectImports(AnnotationMetadata importingclassMetadata) {
	Map<String,Object> attributes = importingclassMetadata.getAnnotationAttributes(ComponentScan.class.getName());
    // 1.直接返回实现类的全路径
	// return new String[]{AService.class.getCanonicalName(),BService.class.getCanonicalName()};
    String[] basePackages = (String[]) attributes.get("basePackages");
    TypeFilter typeFilter = new AssignableTypeFilter(IServiceclass);
    //根据类路径扫描获取类信息
    var provider = new ClassPathScanningCandidateComponentProvider(false);
	provider.addIncludeFilter(typeFilter);
	Set<String> classes = new HashSet<>();
	for(String basePackage:basePackages){
		Set<BeanDefinition> beanDefinitions = provider.findCandidateComponents(basePackage);
        for(BeanDefinition beanDefinition:beanDefinitions){
			classes.add(beanDefinition.getBeanClassName());
        }
    }
	return classes.toArray(new String[0]);
}
```

**1.3 实现 ImportBeanDefinitionRegistrar 接口**

```java
@Override
public void registerBeanDefinitions(AnnotationMetadata importingclassletadata, BeanDefinitionRegistry registry) {
    RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(ImportJavaObiect.class);
	registry.registerBeanDefinition("myImportJavaObject",rootBeanDefinition);
}
```

```java
@Override
public void registerBeanDefinitions(AnnotationMetadata importingclassletadata, BeanDefinitionRegistry registry) {
	BeanDefinitionBuilder definition = BeanDefinitionBuilder.genericBeanDefinition(ImportJava0bject.class,
		new Supplier<ImportJavaObject>() {
			@Override
			public ImportJavaObject get() {return new ImportJavaobject();}
        });
	definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);
    definition.setLazyInit(true);
	var beanDefinition = definition.getBeanDefinition();
    var holder = new BeanDefinitionHolder(beanDefinition,ImportJavabject.class.getName(),new String[{"AImportJavaObject","BImportJavaObject"});
	BeanDefinitionReaderUtils.registerBeanDefinition(holder,registry);
}
```

### 2. 扫描收集 FeignClient

**2.1 获取扫描 FeignClient 的类路径**

2.1.1 通过 AnnotationMetadata

- 通过 EnableFeignClients 的 clients 获取
- 获取扫描 FeignClient 的类路径
  - 通过 EnableFeignClients 注解的属性 value 获取
  - 通过 EnableFeignClients 注解的属性 basePackages 获取
  - 通过 EnableFeignClients 注解的 basePackageClasses 获取
  - 通过 EnableFeignClients 注解作用的类的包路径作为扫描路径

2.1.2 通过 ClassPathScanningCandidateComponentProvider

```java
public void registerFeignClients(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
	LinkedHashSet<BeanDefinition> candidateComponents = new LinkedHashSet<>();
    //获取@EnableFeignCLients注解的注解属性信息
	Map<String, Object> attrs = metadata.getAnnotationAttributes(EnableFeignClients.class.getName());
	//从@EnabLeFeignCLients注解里获取cLients质性的值
    final Class<?>[] clients = attrs == null ? null : (Class<?>[]) attrs.get("clients");
	//如cLients值为空，即@EnablecLients注解没有赋值clients，获取使用了@Feignclient注解的对象，将这些对象注册到IOC容器
	//怎么将注解对象通过AnnotationBeanFactory注册到beanDefinitionMap?
    if (clients == null || clients.length == 0) {
        //获取提供扫描信息对象CLassPathScanningCandidateComponentProvider
		ClassPathScanningCandidateComponentProvider scanner = getScanner();
		scanner.setResourceLoader(this.resourceLoader);
        //scanner添加过滤器,在scanner里拦截获取使用了@Feignclient的对象
		scanner.addIncludeFilter(new AnnotationTypeFilter(FeignClient.class));
        //获取@EnabLeFeignClients里配置的basePackages
		Set<String> basePackages = getBasePackages(metadata);
		for (String basePackage : basePackages) {
            //根basePackage我到使用了@FeignClient的注解的元数据对象，添加到candidateComponents
			candidateComponents.addAll(scanner.findCandidateComponents(basePackage));
		}
	} else {//如果clients不为空，将clients里所有的client装入candidateComponents
		for (Class<?> clazz : clients) {
			candidateComponents.add(new AnnotatedGenericBeanDefinition(clazz));
		}
	}
	//遍历所有使用了@FeigncLient的注解元数据对象
    for (BeanDefinition candidateComponent : candidateComponents) {
		if (candidateComponent instanceof AnnotatedBeanDefinition beanDefinition) {
			// verify annotated class is an interface
			AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
			Assert.isTrue(annotationMetadata.isInterface(), "@FeignClient can only be specified on an interface");
            //java.lang.Class类的getCanonicalName()
            //方法用于获取该类的规范名称，该名称是Java语言规范定义的规范名称
            //该方法以String的形式返回此类的规范名称
			Map<String, Object> attributes = annotationMetadata.getAnnotationAttributes(FeignClient.class.getCanonicalName());
			String name = getClientName(attributes);
            //如果Feignclient注解里使用到configuration，要再次注册configuration,此时注册的feign client name的cofiguration
			String className = annotationMetadata.getClassName();
			registerClientConfiguration(registry, name, className, attributes.get("configuration"));
            //注册FeignClient
			registerFeignClient(registry, annotationMetadata, attributes);
		}
	}
}
```

**2.2 使用 ScannerProvider 扫描包路径获取 FeignClient 接口的 BeanDefinition**
