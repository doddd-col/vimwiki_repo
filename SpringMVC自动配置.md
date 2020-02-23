## SpringMVC自动配置   
- 包含ContentNegotiatingViewResolver 和 BeanNameViewResolver beans。
	- 自动配置ViewResolver(视图解析器)
	- ContenNegotiantingViewResolve组合所有视图解析器
- 支持提供静态资源，包括对 WebJars 的支持（ 本文档稍后介绍））。
- 自动注册 Converter，GenericConverter 和 Formatter beans。
	- Converter自动转换器
	- Formatter格式化器
	```java	
	 	@Bean
		@Override
		public FormattingConversionService mvcConversionService() {
			WebConversionService conversionService = new WebConversionService(this.mvcProperties.getDateFormat());
			addFormatters(conversionService);
			return conversionService;
		}
	```
- 支持 HttpMessageConverters（ 本文档稍后部分）。
	- HttpMessageConverters消息转换器
- 自动注册 MessageCodesResolver（ 本文档后面部分）。
- 静态 index.html 支持。
- 自定义 Favicon 支持（本文档稍后介绍）。
- 自动使用 ConfigurableWebBindingInitializer bean（本文 后面会介绍）。
	- 初始化WebDateeBinder
	- 请求数据====javabean

## 全面接管SpringMVC
 @EnableWevMvc
 ```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({DelegatingWebMvcConfiguration.class})
public @interface EnableWebMvc {
}

 ```
 ```java
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
 ```

```java
 @Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
// 判断容器中是否有这个组件，没有的时候才会生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```

 @EnableWebMvc会WebMvcConfigurationSupport导入
 
 SpringBoot对SpringMVC的自动全部失效，只有用户自己配置的生效
 
## 如何修改SpringBoot自动配置
 1. SpringBoot
	先看容器中有没有用户的配置,有就使用用户的，没有就默认配置,有一些配置会共同生效
 2. 在SpringBoot会有很多xxxConfiguration
