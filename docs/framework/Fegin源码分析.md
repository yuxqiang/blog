# open-fegin 工作流程
OpenFeign是一个远程客户端请求代理，它的基本作用是让开发者能够以面向接口的方式来实现远程调用，从而屏蔽底层通信的复杂性，它的具体原理如下图所示。
## OpenFeign注解扫描与解析
思考， 一个被声明了@FeignClient注解的接口，使用@Autowired进行依赖注入，而最终这个接口能够正常被注入实例。
1.被@FeignClient声明的接口，在Spring容器启动时，会被解析。
2.由于被Spring容器加载的是接口，而接口又没有实现类，因此Spring容器解析时，会生成一个动态代理类。
### EnableFeignClient原理
@FeignClient注解是在什么时候被解析的呢？基于我们之前所有积累的知识，无非就以下这几种

ImportSelector，批量导入bean
ImportBeanDefinitionRegistrar，导入bean声明并进行注册
BeanFactoryPostProcessor ， 一个bean被装载的前后处理器
在这几个选项中，似乎ImportBeanDefinitionRegistrar更合适，因为第一个是批量导入一个bean的string集合，不适合做动态Bean的声明。 而BeanFactoryPostProcessor是一个Bean初始化之前和之后被调用的处理器。

而在我们的FeignClient声明中，并没有Spring相关的注解，所以自然也不会被Spring容器加载和触发。
在集成FeignClient时，我们在SpringBoot的main方法中，声明了一个注解@EnableFeignClients(basePackages = "com.gupaoedu.ms.api")。这个注解需要填写一个指定的包名。

嗯，看到这里，基本上就能猜测出，这个注解必然和@FeignClient注解的解析有莫大的关系。

下面这段代码是@EnableFeignClients注解的声明，果然看到了一个很熟悉的面孔FeignClientsRegistrar。
``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {

}
```
#### FeignClientsRegistrar
FeignClientRegistrar，主要功能就是针对声明@FeignClient注解的接口进行扫描和注入到IOC容器
``` java
class FeignClientsRegistrar
		implements ImportBeanDefinitionRegistrar, ResourceLoaderAware, EnvironmentAware {
		}
```
果然，这个类实现了ImportBeanDefinitionRegistrar接口
```
public interface ImportBeanDefinitionRegistrar {
    default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry, BeanNameGenerator importBeanNameGenerator) {
        this.registerBeanDefinitions(importingClassMetadata, registry);
    }

    default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
    }
}
```
这个接口有两个重载的方法，用来实现Bean的声明和注册。
#### FeignClientsRegistrar.registerBeanDefinitions
registerDefaultConfiguration 方法内部从 SpringBoot 启动类上检查是否有 @EnableFeignClients, 有该注解的话， 则完成Feign框架相关的一些配置内容注册。
registerFeignClients 方法内部从 classpath 中， 扫描获得 @FeignClient 修饰的类， 将类的内容解析为 BeanDefinition , 最终通过调用 Spring 框架中的 BeanDefinitionReaderUtils.resgisterBeanDefinition 将解析处理过的 FeignClient BeanDeifinition 添加到 spring 容器中

```java
//注册@EnableFeignClients中定义defaultConfiguration属性下的类，包装成FeignClientSpecification，注册到Spring容器。
//在@FeignClient中有一个属性：configuration，这个属性是表示各个FeignClient自定义的配置类，后面也会通过调用registerClientConfiguration方法来注册成FeignClientSpecification到容器。
//所以，这里可以完全理解在@EnableFeignClients中配置的是做为兜底的配置，在各个@FeignClient配置的就是自定义的情况。
	public void registerBeanDefinitions(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
		//有这个注解才开始注册registerFeignClients到springbean容器
		//注册缺省配置到Spring容器
		registerDefaultConfiguration(metadata, registry);
		//注册所发现的各个客户端FeignClient到Spring容器
		registerFeignClients(metadata, registry);
	}
```
这里面需要重点分析的就是registerFeignClients方法，这个方法主要是扫描类路径下所有的@FeignClient注解，然后进行动态Bean的注入。它最终会调用registerFeignClient方法。
```java
public void registerFeignClients(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
    registerFeignClient(registry, annotationMetadata, attributes);
}
```
##### FeignClientsRegistrar.registerFeignClients
```java
public void registerFeignClients(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
		ClassPathScanningCandidateComponentProvider scanner = getScanner();
		scanner.setResourceLoader(this.resourceLoader);

		Set<String> basePackages;
        ///获取@EnableFeignClients注解的元数据
		Map<String, Object> attrs = metadata
				.getAnnotationAttributes(EnableFeignClients.class.getName());
		AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(
				FeignClient.class);
//获取@EnableFeignClients注解中的clients属性，可以配置@FeignClient声明的类，如果配置了，则需要扫描并加载
		final Class<?>[] clients = attrs == null ? null
				: (Class<?>[]) attrs.get("clients");
		if (clients == null || clients.length == 0) {
        //默认TypeFilter生效，这种模式会查询出许多不符合你要求的class名
        //添加包含过滤的属性@FeignClient。
			scanner.addIncludeFilter(annotationTypeFilter);
			basePackages = getBasePackages(metadata);
		}
		else {
			final Set<String> clientClasses = new HashSet<>();
			basePackages = new HashSet<>();
			for (Class<?> clazz : clients) {
				basePackages.add(ClassUtils.getPackageName(clazz));
				clientClasses.add(clazz.getCanonicalName());
			}
			AbstractClassTestingTypeFilter filter = new AbstractClassTestingTypeFilter() {
				@Override
				protected boolean match(ClassMetadata metadata) {
					String cleaned = metadata.getClassName().replaceAll("\\$", ".");
					return clientClasses.contains(cleaned);
				}
			};
			scanner.addIncludeFilter(
					new AllTypeFilter(Arrays.asList(filter, annotationTypeFilter)));
		}

		for (String basePackage : basePackages) {
			Set<BeanDefinition> candidateComponents = scanner
					.findCandidateComponents(basePackage);
			for (BeanDefinition candidateComponent : candidateComponents) {
				if (candidateComponent instanceof AnnotatedBeanDefinition) {
					// verify annotated class is an interface
					AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
					AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
					Assert.isTrue(annotationMetadata.isInterface(),
							"@FeignClient can only be specified on an interface");

					// 获取所定义的feign客户端接口上的注解@FeignClient属性
					Map<String, Object> attributes = annotationMetadata
							.getAnnotationAttributes(
									FeignClient.class.getCanonicalName());
                  //获取@FeignClient中配置的服务名称。
					String name = getClientName(attributes);
					registerClientConfiguration(registry, name,
							attributes.get("configuration"));

					registerFeignClient(registry, annotationMetadata, attributes);
				}
			}
		}
	}
```
##### FeignClient Bean的注册
这个方法就是把FeignClient接口注册到Spring IOC容器，
```java
private void registerFeignClient(BeanDefinitionRegistry registry,
			AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
    //获取FeignClient接口的类全路径
		String className = annotationMetadata.getClassName();
		BeanDefinitionBuilder definition = BeanDefinitionBuilder
				.genericBeanDefinition(FeignClientFactoryBean.class);
		validate(attributes);
		definition.addPropertyValue("url", getUrl(attributes));
		definition.addPropertyValue("path", getPath(attributes));
		String name = getName(attributes);
		definition.addPropertyValue("name", name);
		String contextId = getContextId(attributes);
		definition.addPropertyValue("contextId", contextId);
		definition.addPropertyValue("type", className);
		definition.addPropertyValue("decode404", attributes.get("decode404"));
		definition.addPropertyValue("fallback", attributes.get("fallback"));
		definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
		definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);

		String alias = contextId + "FeignClient";
		AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();
		beanDefinition.setAttribute(FactoryBean.OBJECT_TYPE_ATTRIBUTE, className);

		// has a default, won't be null
		boolean primary = (Boolean) attributes.get("primary");

		beanDefinition.setPrimary(primary);

		String qualifier = getQualifier(attributes);
		if (StringUtils.hasText(qualifier)) {
			alias = qualifier;
		}

		BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className,
				new String[] { alias });
		BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
	}
```
综上代码分析，其实实现逻辑很简单。
创建一个BeanDefinitionBuilder。
创建一个工厂Bean，并把从@FeignClient注解中解析的属性设置到这个FactoryBean中
调用registerBeanDefinition注册到IOC容器中

#### FeignClientFactoryBean的详解
```java
public Object getObject() {
   return getTarget();
}
```
构建对象Bean的实现代码如下，这个代码的实现较长，我们分为几个步骤来看
Feign上下文的构建#
先来看上下文的构建逻辑，代码部分如下。
```java
<T> T getTarget() {
        FeignContext context = this.applicationContext.getBean(FeignContext.class);
        Feign.Builder builder = feign(context);

        if (!StringUtils.hasText(this.url)) {
        if (!this.name.startsWith("http")) {
        this.url = "http://" + this.name;
        }
        else {
        this.url = this.name;
        }
        this.url += cleanPath();
        return (T) loadBalance(builder, context,
        new HardCodedTarget<>(this.type, this.name, this.url));
        }
        if (StringUtils.hasText(this.url) && !this.url.startsWith("http")) {
        this.url = "http://" + this.url;
        }
        String url = this.url + cleanPath();
        Client client = getOptional(context, Client.class);
        if (client != null) {
        if (client instanceof LoadBalancerFeignClient) {
        // not load balancing because we have a url,
        // but ribbon is on the classpath, so unwrap
        client = ((LoadBalancerFeignClient) client).getDelegate();
        }
        if (client instanceof FeignBlockingLoadBalancerClient) {
        // not load balancing because we have a url,
        // but Spring Cloud LoadBalancer is on the classpath, so unwrap
        client = ((FeignBlockingLoadBalancerClient) client).getDelegate();
        }
        builder.client(client);
        }
        Targeter targeter = get(context, Targeter.class);
        return (T) targeter.target(this, builder, context,
        new HardCodedTarget<>(this.type, this.name, url));
        }
```


