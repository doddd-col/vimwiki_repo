## SpringBoot启动配置原理
**重要回调实例**  
// 配置在META-INF/spring.facotries
1. ApplicationContextInitializer
2. SpringApplicationRunListener
// 放在IOC容器里的
3. ApplicationRunner
4. CommandLineRunner

**启动流程**
1. 创建SpringApplicaton对象
```java
// 先创建一个SpringApplication对象在执行run方法
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
	return new SpringApplication(primarySources).run(args);
}
// run 返回一个SpringApplication实例
public SpringApplication(Class<?>... primarySources) {
	this(null, primarySources);
}


// SpringApplication构造器
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	this.resourceLoader = resourceLoader;
	Assert.notNull(primarySources, "PrimarySources must not be null");
	this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
	this.webApplicationType = WebApplicationType.deduceFromClasspath();
	setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
	setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
	this.mainApplicationClass = deduceMainApplicationClass();
}


// 扫描自动配置容器
public void setInitializers(Collection<? extends ApplicationContextInitializer<?>> initializers) {
	this.initializers = new ArrayList<>(initializers);
}

// 扫描监听器
public void setListeners(Collection<? extends ApplicationListener<?>> listeners) {
	this.listeners = new ArrayList<>(listeners);
}

// 扫描所有包 找到并返回主程序
private Class<?> deduceMainApplicationClass() {
	try {
		StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
		for (StackTraceElement stackTraceElement : stackTrace) {
			if ("main".equals(stackTraceElement.getMethodName())) {
				return Class.forName(stackTraceElement.getClassName());
			}
		}
	}
	catch (ClassNotFoundException ex) {
		// Swallow and continue
	}
	return null;
}
```

2. 运行run方法
```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
	return new SpringApplication(primarySources).run(args);
}
 

//Run the Spring application, creating and refreshing a new
public ConfigurableApplicationContext run(String... args) {
	StopWatch stopWatch = new StopWatch();
	stopWatch.start();  // 开始监听
	ConfigurableApplicationContext context = null;
	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
	configureHeadlessProperty();	// awt有关的

	// 获取SpringApplicationRunListeners  从类路径下的META-INF/spring.facotories获取
	SpringApplicationRunListeners listeners = getRunListeners(args);
	//回调所有的获取SpringApplicationRunListener.starting()方法
	listeners.starting();
	try {
		// 封装命令行参数
		ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
		// 准备环境 --创建环境完成后回调所有的获取SpringApplicationRunListener.enviromentPrepared();表示环境准备完成
		ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			
		configureIgnoreBeanInfo(environment);
		//打印控制台图标
		Banner printedBanner = printBanner(environment);
		//根据情况选择创建webIOC或是普通IOC容器	
		context = createApplicationContext();
		
		// 获取工厂实例 加载自定义
		exceptionReporters = getSpringFactoriesInstances(SpringBootEceptionReporter.class,
				new Class[] { ConfigurableApplicationContext.class }, context);
				
		// 准备上下文环境  保存刚才获取的环境到IOC里 
		// 并且里面有一个applyInitializers(context)方法-
		// -将之前创建SpringApplication获取到的所有ApplicationContextInitializer里面的Initialize方法进行回调
		// 还有listeners.contextPrepared(context);回调所有SpringApplicationRunListener的contextPrepared()方法
		prepareContext(context, environment, listeners, applicationArguments, printedBanner);
		//prepareContext运行到最后会回调所有SpringApplicationRunListener的contextLoaded()方法
		
		// 刷新容器 IOC容器初始化 (如果是web应用还会创建嵌入式的tomcat)
		// 扫描 创建 加载所有组件的地方(配置类，组件，自动配置)
		refreshContext(context);
		
		afterRefresh(context, applicationArguments);
		stopWatch.stop();
		if (this.logStartupInfo) {
			new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
		}
		// 调用所有SpringApplicationRunListener的start()方法
		
	/**
	 * The context has been refreshed and the application has started but
	 * {@link CommandLineRunner CommandLineRunners} and {@link ApplicationRunner
	 * ApplicationRunners} have not been called.
	 * @param context the application context.
	 * @since 2.0.0
	 */
		// 官方文档描述 尚未调用CommandLineRunner CommandLineRunners and  ApplicationRunner
		listeners.started(context);
		
		// 从IOC容器中获取所有的ApplicationRunner和CommandLineRunner, 
		// ApplicationRuner先回调 CommandLineRunner后回调
		callRunners(context, applicationArguments);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, listeners);
		throw new IllegalStateException(ex);
	}

	try {
		// 调用所有ApplicationRunners CommandLineRunners
	/**
	 * Called immediately before the run method finishes, when the application context has
	 * been refreshed and all {@link CommandLineRunner CommandLineRunners} and
	 * {@link ApplicationRunner ApplicationRunners} have been called.
	 * @param context the application context.
	 * @since 2.0.0
	 */
	 
		listeners.running(context);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, null);
		throw new IllegalStateException(ex);
	}
	// 整个SpringBoot应用启动完成以后返回启动的IOC容器
	return context;
}
```

creatApplicationContext
```java
protected ConfigurableApplicationContext createApplicationContext() {
	Class<?> contextClass = this.applicationContextClass;
	if (contextClass == null) {
		try {
			switch (this.webApplicationType) {
			case SERVLET:
				contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
				break;
			case REACTIVE:
				contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
				break;
			default:
				contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
			}
		}
		catch (ClassNotFoundException ex) {
			throw new IllegalStateException(
					"Unable create a default ApplicationContext, please specify an ApplicationContextClass", ex);
		}
	}	
	// 利用BeanUtils的反射创建context
	return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
}
```

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
	ClassLoader classLoader = getClassLoader();
	// Use names and ensure unique to protect against duplicates
	Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
	List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
	AnnotationAwareOrderComparator.sort(instances);
	return instances;
}
```

```java
private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment,
			SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
		context.setEnvironment(environment);
		postProcessApplicationContext(context);
		applyInitializers(context);
		listeners.contextPrepared(context);
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}
		// Add boot specific singleton beans
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
		if (printedBanner != null) {
			beanFactory.registerSingleton("springBootBanner", printedBanner);
		}
		if (beanFactory instanceof DefaultListableBeanFactory) {
			((DefaultListableBeanFactory) beanFactory)
					.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		if (this.lazyInitialization) {
			context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
		}
		// Load the sources
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
		load(context, sources.toArray(new Object[0]));
		listeners.contextLoaded(context);
	}
```

 

```java
protected void applyInitializers(ConfigurableApplicationContext context) {
	for (ApplicationContextInitializer initializer : getInitializers()) {
		Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(initializer.getClass(),
				ApplicationContextInitializer.class);
		Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
		initializer.initialize(context);
	}
}
```

```java
private void callRunners(ApplicationContext context, ApplicationArguments args) {
	List<Object> runners = new ArrayList<>();
	runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
	runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
	AnnotationAwareOrderComparator.sort(runners);
	for (Object runner : new LinkedHashSet<>(runners)) {
		if (runner instanceof ApplicationRunner) {
			callRunner((ApplicationRunner) runner, args);
		}
		if (runner instanceof CommandLineRunner) {
			callRunner((CommandLineRunner) runner, args);
		}
	}
}
```

  
 

