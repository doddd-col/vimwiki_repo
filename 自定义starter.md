## 自定义starter
1. 这个场景需要什么依赖
2. 如何编写自动配置
```java
@Configuration  // 指定这是一个配置类
@ConditionalOnxxx	// 在指定的条件成立的情况下自动配置类生效
@AutoConfigureAfter		// 指定自动配置类序顺


@Bean	// 添加组件

@ConfigurationProperties  // 结合相关xxxProperties类来绑定相关的配置   
// 如@ConfingurationProperties(prefix = "doddd.hello") 在application.properties中可以根据这个修改配置
@EnableConfigurationProperties		//让xxxProperties类生效加入到容器中


自动配置类要加载
将需要启动就加载的自动配置类，配置在META-INF/spring.factories

org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener
```

3. 模式

启动器只用来做依赖导入
专门写一个自动配置模块
启动器依赖自动配置，使用时只用引入启动器(starter)

命名:xxx-spring-boot-stater

----

**实战**

启动器只需要添加自动配置依赖以及其他所需依赖
```xml
<!--    启动器-->
<dependencies>
	<!--    引入自动配置-->
	<dependency>
		<groupId>com.doddd.starter</groupId>
		<artifactId>doddd-spring-boot-starter-autoconfigurer</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</dependency>
</dependencies>
```

aotuconfigurer实例

HelloService
```java
public class HelloService {

    HelloProperties helloProperties;

    public HelloProperties getHelloProperties() {
        return helloProperties;
    }

    public void setHelloProperties(HelloProperties helloProperties) {
        this.helloProperties = helloProperties;
    }

    public String sayHelloToYou(String name){
        return helloProperties.getPrefix()+" "+name+" "+helloProperties.getSuffix();
    }
}
```

HelloProperties(通过@ConfigurationProperties实现从配置文件对属性进行配置)
```java
@ConfigurationProperties(prefix = "doddd.hello")
public class HelloProperties {
    private String prefix;
    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }

}
```

HelloServiceAutoConfiguration(自动配置类，将HelloProperties与HelloService绑定，并将HelloService对象加入到IOC容器中)
```java
@Configuration
@ConditionalOnWebApplication
// 通过这个注解开启对使用了@ConfigurationProperties这个注解类的支持 可以直接通过注入拿到对象
@EnableConfigurationProperties(HelloProperties.class)
public class HelloServiceAutoConfiguration {

    @Autowired
    HelloProperties helloProperties;
    @Bean
    public HelloService helloService(){
        HelloService service = new HelloService();
        // 将HelloProperties 与HelloService绑定 (这样才可以传值)
        service.setHelloProperties(helloProperties);
        return service;
    }
}
```

pom,xml
```xml
<!--	引入spring-boot-starter:所有starter的基本配置-->
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter</artifactId>
	</dependency>

</dependencies>
```

 
spring.factories(加入到自动启动)
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.doddd.starter.HelloServiceAutoConfiguration
```


