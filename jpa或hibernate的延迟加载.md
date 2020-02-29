org.hibernate.lazyinitializationexception could not initialize proxy - no session
解决:

五个解决方案

1、关闭 LazyInitialization, 将 fetch 设成 eager, 可以在配置文件，也可注解

2、在 spring boot 的配置文件 application.properties 添加 spring.jpa.open-in-view=true，yml 同理

3、用 spring 的 OpenSessionInViewFilter

4、在 spring boot 的配置文件 application.properties 添加 spring.jpa.properties.hibernate.enable_lazy_load_no_trans=true

5、在出问题的实体类上加 @Proxy (lazy = false) 
