**Servlet**  
```java
@Bean
public ServletRegistrationBean mServlet(){
	ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet() ,"/myServlet");
	return registrationBean;
}
```

**Filter**  
```java
@Bean
public FilterRegistrationBean myFilter(){
	FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
	filterRegistrationBean.setFilter(new MyFilter());
	filterRegistrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
	return filterRegistrationBean;
}
```

**Listener**  
```java
@Bean
public ServletListenerRegistrationBean myListener(){
	ServletListenerRegistrationBean<MyListener> myListenerServletListenerRegistrationBean =
			new ServletListenerRegistrationBean<>(new MyListener());
	return myListenerServletListenerRegistrationBean;
}
```


