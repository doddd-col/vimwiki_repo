1.x: 通过实现 org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer 的 customize 方法来实现自定义

2.x: 在 2.x 版本改为实现 org.springframework.boot.web.server.WebServerFactoryCustomizer 接口的 customize 方法

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


### 1.x相关类
- org.springframework.boot.context.embedded.ConfigurableEmbeddedServletContainer
- org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer
- org.springframework.boot.context.embedded.tomcat.TomcatConnectorCustomizer
- org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory


### 2.x相关类
- org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory
- org.springframework.boot.web.server.WebServerFactoryCustomizer
- org.springframework.boot.web.embedded.tomcat.TomcatConnectorCustomizer
- org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory
