### SpringBoot默认是嵌入的servlet容器(tomcat)
1. 如何修改servlet容器配置

- 修改serrver有关的配置(ServerProperties)
```properties
server.servlet.context-path=/crud

# SpringBoot2.0默认false 开启自定义异常
server.error.include-exception=true

server.tomcat.uri-encoding=utf-8

// 通用设置
server.xxx

// Tomcat配置
server.tomcat.xxx
```

- 编写一个WebServerFactoryCustomizer:嵌入式Servlet容器的定制器


```java
public WebServerFactoryCustomizer<ConfigurableWebServerFactory> webServerFactoryCustomizer(){
	return new WebServerFactoryCustomizer<ConfigurableWebServerFactory>() {
		@Override
		public void customize(ConfigurableWebServerFactory factory) {
			factory.setPort(8081);
		}
	};

}
```

 


