## WebMvcConfigurerAdapter 的替换接口或类
 ```java
/** 解决跨域问题 **/
public void addCorsMappings(CorsRegistry registry) ;
/** 添加拦截器 **/
void addInterceptors(InterceptorRegistry registry);
/** 这里配置视图解析器 **/
void configureViewResolvers(ViewResolverRegistry registry);
/** 配置内容裁决的一些选项 **/
void configureContentNegotiation(ContentNegotiationConfigurer configurer);
/** 视图跳转控制器 **/
void addViewControllers(ViewControllerRegistry registry);
/** 静态资源处理 **/
void addResourceHandlers(ResourceHandlerRegistry registry);
/** 默认静态资源处理器 **/
void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer);

 ```

方案 1 直接实现 WebMvcConfigurer
```java
 
@Configuration
public class WebMvcConfg implements WebMvcConfigurer {
 
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/index").setViewName("index");
        }
 
}
```

 方案 2 直接继承 WebMvcConfigurationSupport
 ```java
@Configuration
public class WebMvcConfg extends WebMvcConfigurationSupport {
 
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/index").setViewName("index");
        }
 
}

 ```
>其实，源码下 WebMvcConfigurerAdapter 是实现 WebMvcConfigurer 接口，所以直接实现 WebMvcConfigurer 接口也可以；WebMvcConfigurationSupport 与 WebMvcConfigurerAdapter、接口 WebMvcConfigurer 处于同一个目录下 WebMvcConfigurationSupport 包含 WebMvcConfigurer 里面的方法，由此看来版本中应该是推荐使用 WebMvcConfigurationSupport 类的，WebMvcConfigurationSupport 应该是新版本中对 WebMvcConfigurerAdapter 的替换和扩展

继承 WebMvcConfigurationSupport会使自动配置失效

ps:WebMvcConfigurationSupport 类并不是用来扩展的，是禁止 SpringBoot 对 mvc 的自动配置，完全由用户自己实现配置
