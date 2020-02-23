## 整合Mybatis
```xml
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>2.1.1</version>
</dependency>
```

 
注解版
```java
@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id=#{id}")
    public Department queryDepartmentById(Integer id);

    @Delete("delete from department where id=#{id}")
    public int deleteDepartmentById(Integer id);

    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{departmentName})")
    public int insertDepartment(Department department);

    @Update("Update department set departmentName=#{departmentName} where id=#{id}")
    public int updateDepartmentById(Department department);
}
```

**自定义Mybatis配置规则**
开启驼峰命名
```java
// 创建一个配置类 添加一个ConfigurationCustomizer组件
@Configuration
public class MyBatisConfig {
	@Bean
    public ConfigurationCustomizer configurationCustomizer(){
       return new ConfigurationCustomizer() {
            @Override
            public void customize(org.apache.ibatis.session.Configuration configuration) {
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```


```java
@MapperScan("springboot.springbootmybatis.mapper")

// 批量扫描 加载SpingBoot启动器上
```

配置版
```yml
mybatis:
  # 指定全局配置文件位置
  config-location: classpath:mybatis/mybatis-config.xml
  # 指定sql映射文件位置
  mapper-locations: classpath:mybatis/mapper/*.xml
```

 
