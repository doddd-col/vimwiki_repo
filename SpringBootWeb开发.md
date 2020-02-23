# SpringBootWeb开发
使用SpringBoot：
1. 创建SpringBoot应用，选用我们需要的模块
2. SpringBoot默认配置好，只需要在配置文件中指定少量的配置即可
3. 业务开发

**自动配置**
> xxxAutoConfiguration
> 帮我们给容器自动配置组件<br/>
> xxxProperties 配置类封装配置文件的内容 

## SpringBoot对静态资源的映射规则

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
```

```java
@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}
```
1. 所有/web/jars/**,都在classpath:/META-INF/resources/webjars/找资源
	- [webjars](https://www.webjars.org/):以jar包的方式引入静态资源

2. "/**"访问当前项目路的任何资源（静态资源文件夹）
	```java
	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { 
	"classpath:/META-INF/resources/",
	"classpath:/resources/",
	"classpath:/static/", 
	"classpath:/public/" }; 

	"/"当前根路径
	``` 
3. index 静态资源文件夹下的所有index.html页面，被“/**”映射
```java
		@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			return welcomePageHandlerMapping;
		}
```

4. 修改网页图标
将图标改为favicon.ico放到静态资源下

5. 重新指定静态资源文件夹(默认路径失效)
```yml
spring.resources.static-location=classpath:/website/,classpath:/html/
```


