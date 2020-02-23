1. 引入依赖
```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid</artifactId>
	<version>1.1.21</version>
</dependency>
```

 
2. 写application.yml
```yml
spring:
  datasource:
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://182.92.225.30:3307/mybatis
    type: com.alibaba.druid.pool.DruidDataSource

    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

3. 创建配置类绑定yml数据并配置监控和拦截器
```java
/**
 * @program: springboot-mybatis
 * @description: Druid配置类
 * @author: doddd
 * @create: 2020-02-22 16:44
 **/
@Configuration
public class DruidConfig {

    /**
     * @Author doddd
     * @Description 配置数据源
     * @Date 4:47 下午 2020/2/22
     * @Param []
     * @return javax.sql.DataSource
     **/
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return new DruidDataSource();
    }


    /**
     * @Author doddd
     * @Description 配置Druid监控
     * @Date 4:55 下午 2020/2/22
     * @Param []
     * @return org.springframework.boot.web.servlet.ServletRegistrationBean
     **/
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new StatViewServlet(), "/druid" +
                "/*");
        Map<String,String> initParams = new HashMap<>();

        initParams.put("loginUsername", "admin");
        initParams.put("loginPassword", "123456");
        initParams.put("allow", "");
        initParams.put("deny", "182.92.225.30");

        servletRegistrationBean.setInitParameters(initParams);
        return servletRegistrationBean;
    }

    /**
     * @Author doddd
     * @Description 配置拦截器
     * @Date 5:03 下午 2020/2/22
     * @Param []
     * @return org.springframework.boot.web.servlet.FilterRegistrationBean
     **/
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions", "*。js, *.css, /druid/*");

        filterRegistrationBean.setInitParameters(initParams);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/*"));

        return filterRegistrationBean;
    }
}
```

 

