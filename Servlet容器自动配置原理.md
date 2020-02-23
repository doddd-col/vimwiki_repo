## SpringBoot容器自动配置原理
| Servlet 容器类型 | WebServer 模型接口 | WebServer 工厂实现类            |
|------------------|--------------------|---------------------------------|
| Tomcat           | TomcatWebServer    | TomcatServletWebServerFactory   |
| Jetty            | JettyWebServer     | JettyServletWebServerFactory    |
| Undertow         | UndertowWebServer  | UndertowServletWebServerFactory |

BeanPostProcessor 接口作用是：如果我们需要在 Spring 容器完成 Bean 的实例化、配置和其他的初始化前后添加一些自己的逻辑处理，我们就可以定义一个或者多个 BeanPostProcessor 接口的实现，然后注册到容器中。
自动配置类

ImportBeanDefinitionRegistrar: 动态注册Bean

>ServletWebServerFactoryAutoConfiguration满足条件 ->引入的容器配置类存放在ServletWebServerFactoryConfiguration
(三个都是内部类)

>它们会分别检测 classpath 上存在的类，从而判断当前应用使用的是 Tomcat/Jetty/Undertow 中的哪一个 Servlet Web 服务器，从而决定定义相应的工厂组件 bean : TomcatServletWebServerFactory/JettyServletWebServerFactory/UndertowServletWebServerFactory。


```java
@Configuration(proxyBeanMethods = false)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
// 导入 ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar 以注册 BeanPostProcessor : WebServerFactoryCustomizerBeanPostProcessor 和 ErrorPageRegistrarBeanPostProcessor
// 当以上条件满足引入以下嵌入式容器配置类
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
		ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
		ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
		ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {
	// 定义 bean ServletWebServerFactoryCustomizer，绑定properties与serverProperties配置，解释了为什么配置文件也可修改servlet容器配置
	@Bean
	public ServletWebServerFactoryCustomizer servletWebServerFactoryCustomizer(ServerProperties serverProperties) {
		return new ServletWebServerFactoryCustomizer(serverProperties);
	}
    // 针对当前Servlet容器是Tomcat时定义该 bean，用于定制化 TomcatServletWebServerFactory
	@Bean
	// 仅在类 org.apache.catalina.startup.Tomcat 存在于 classpath 上时才生效
	@ConditionalOnClass(name = "org.apache.catalina.startup.Tomcat")
	public TomcatServletWebServerFactoryCustomizer tomcatServletWebServerFactoryCustomizer(
			ServerProperties serverProperties) {
		return new TomcatServletWebServerFactoryCustomizer(serverProperties);
	}

	/**
     * Registers a WebServerFactoryCustomizerBeanPostProcessor. Registered via
     * ImportBeanDefinitionRegistrar for early registration.
     * 这是一个 ImportBeanDefinitionRegistrar， 它会向容器注入两个 BeanPostProcessor :
     * 1. WebServerFactoryCustomizerBeanPostProcessor
     * 该 BeanPostProcessor 会搜集容器中所有的 WebServerFactoryCustomizer，对当前应用所采用的
     * WebServerFactory 被初始化前进行定制
     * 2. ErrorPageRegistrarBeanPostProcessor
     * 该 BeanPostProcessor 会搜集容器中所有的 ErrorPageRegistrar，添加到当前应用所采用的
     * ErrorPageRegistry 中,实际上，这里的 ErrorPageRegistry 会是 ConfigurableWebServerFactory,
     * 具体实现上来讲，是一个 ConfigurableTomcatWebServerFactory,ConfigurableJettyWebServerFactory
     * 或者 ConfigurableUndertowWebServerFactory,分别对应 Tomcat, Jetty, Undertow 这三种
     * Servlet Web 容器的工厂类
     */

	public static class BeanPostProcessorsRegistrar implements ImportBeanDefinitionRegistrar, BeanFactoryAware {

	private ConfigurableListableBeanFactory beanFactory;

	@Override
	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
		if (beanFactory instanceof ConfigurableListableBeanFactory) {
			this.beanFactory = (ConfigurableListableBeanFactory) beanFactory;
		}
	}

	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
			BeanDefinitionRegistry registry) {
		if (this.beanFactory == null) {
			return;
		}
		registerSyntheticBeanIfMissing(registry, "webServerFactoryCustomizerBeanPostProcessor",
				WebServerFactoryCustomizerBeanPostProcessor.class);
		registerSyntheticBeanIfMissing(registry, "errorPageRegistrarBeanPostProcessor",
				ErrorPageRegistrarBeanPostProcessor.class);
	}

	private void registerSyntheticBeanIfMissing(BeanDefinitionRegistry registry, String name, Class<?> beanClass) {
		if (ObjectUtils.isEmpty(this.beanFactory.getBeanNamesForType(beanClass, true, false))) {
			RootBeanDefinition beanDefinition = new RootBeanDefinition(beanClass);
			beanDefinition.setSynthetic(true);
			registry.registerBeanDefinition(name, beanDefinition);
		}
	}

}
```

ServletWebServerFactoryConfiguration
ServletWebServerFactoryConfiguration 是一个针对 ServletWebServerFactory 进行配置的配置类。它通过检测应用 classpath 存在的类，从而判断当前应用要使用哪个 Servlet 容器：Tomcat,Jetty 还是 Undertow。检测出来之后，定义相应的 Servlet Web 服务器工厂组件 bean

Servlet 容器类型 	WebServerFactory 实现类
Tomcat 	TomcatServletWebServerFactory
Jetty    	JettyServletWebServerFactory
Undertow    	UndertowServletWebServerFactory
注意，以上三个实现类都继承自抽象基类 AbstractServletWebServerFactory, 实现了接口 WebServerFactory,ErrorPageRegistry。该特征会被 WebServerFactoryCustomizerBeanPostProcessor 和 ErrorPageRegistrarBeanPostProcessor 使用，用于对 WebServerFactory 进行定制化，以及设置相应的错误页面。
```java
@Configuration(proxyBeanMethods = false)
class ServletWebServerFactoryConfiguration {
	@Configuration(proxyBeanMethods = false)
	@ConditionalOnClass({ Servlet.class, Tomcat.class, UpgradeProtocol.class }) // 判断是否引入了Tomcat
	// 判断用户有没有自定义ServletWebServerFactory 容器工厂
	@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT) 
	static class EmbeddedTomcat {

		@Bean
		TomcatServletWebServerFactory tomcatServletWebServerFactory(
				ObjectProvider<TomcatConnectorCustomizer> connectorCustomizers,
				ObjectProvider<TomcatContextCustomizer> contextCustomizers,
				ObjectProvider<TomcatProtocolHandlerCustomizer<?>> protocolHandlerCustomizers) {
			TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
			factory.getTomcatConnectorCustomizers()
					.addAll(connectorCustomizers.orderedStream().collect(Collectors.toList()));
			factory.getTomcatContextCustomizers()
					.addAll(contextCustomizers.orderedStream().collect(Collectors.toList()));
			factory.getTomcatProtocolHandlerCustomizers()
					.addAll(protocolHandlerCustomizers.orderedStream().collect(Collectors.toList()));
			return factory;
		}
	}

	/**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration(proxyBeanMethods = false)
	@ConditionalOnClass({ Servlet.class, Server.class, Loader.class, WebAppContext.class })
	@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT)
	static class EmbeddedJetty {

		@Bean
		JettyServletWebServerFactory JettyServletWebServerFactory(
				ObjectProvider<JettyServerCustomizer> serverCustomizers) {
			JettyServletWebServerFactory factory = new JettyServletWebServerFactory();
			factory.getServerCustomizers().addAll(serverCustomizers.orderedStream().collect(Collectors.toList()));
			return factory;
		}

	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	@Configuration(proxyBeanMethods = false)
	@ConditionalOnClass({ Servlet.class, Undertow.class, SslClientAuthMode.class })
	@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT)
	static class EmbeddedUndertow {

		@Bean
		UndertowServletWebServerFactory undertowServletWebServerFactory(
				ObjectProvider<UndertowDeploymentInfoCustomizer> deploymentInfoCustomizers,
				ObjectProvider<UndertowBuilderCustomizer> builderCustomizers) {
			UndertowServletWebServerFactory factory = new UndertowServletWebServerFactory();
			factory.getDeploymentInfoCustomizers()
					.addAll(deploymentInfoCustomizers.orderedStream().collect(Collectors.toList()));
			factory.getBuilderCustomizers().addAll(builderCustomizers.orderedStream().collect(Collectors.toList()));
			return factory;
		}

	}

}
```

ServletWebServerFactoryConfiguration 

- TomcatServletWebServerFactory
- JettyServletWebServerFactory
- UndertowServletWebServerFactory

TomcatServletWebServerFactory分析
```java
只有当端口号大于0才会启动
protected TomcatWebServer getTomcatWebServer(Tomcat tomcat) {
	return new TomcatWebServer(tomcat, this.getPort() >= 0);
}
```

进入TomcatWebServer
```java
public TomcatWebServer(Tomcat tomcat, boolean autoStart) {
	Assert.notNull(tomcat, "Tomcat Server must not be null");
	this.tomcat = tomcat;
	this.autoStart = autoStart;
	initialize();
}

// Tomcat初始化
private void initialize() throws WebServerException {
	logger.info("Tomcat initialized with port(s): " + getPortsDescription(false));
	synchronized (this.monitor) {
		try {
			addInstanceIdToEngineName();

			Context context = findContext();
			context.addLifecycleListener((event) -> {
				if (context.equals(event.getSource()) && Lifecycle.START_EVENT.equals(event.getType())) {
					// Remove service connectors so that protocol binding doesn't
					// happen when the service is started.
					removeServiceConnectors();
				}
			});

			// Start the server to trigger initialization listeners
			this.tomcat.start();

			// We can re-throw failure exception directly in the main thread
			rethrowDeferredStartupExceptions();

			try {
				ContextBindings.bindClassLoader(context, context.getNamingToken(), getClass().getClassLoader());
			}
			catch (NamingException ex) {
				// Naming is not enabled. Continue
			}

			// Unlike Jetty, all Tomcat threads are daemon threads. We create a
			// blocking non-daemon to stop immediate shutdown
			startDaemonAwaitThread();
		}
		catch (Exception ex) {
			stopSilently();
			destroySilently();
			throw new WebServerException("Unable to start embedded Tomcat", ex);
		}
	}
}
```




 

