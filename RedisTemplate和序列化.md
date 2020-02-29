spring-boot-redis中有两个对象操作redis
1. StringRedisTemplat			--kv操作字符串
2. RedisTemplate				--kv操作对象

```java
// 操作字符串
@Autowired
StringRedisTemplate stringRedisTemplate;

// 操作对象
@Autowired
RedisTemplate redisTemplate;
@Test
void contextLoads() {
	//stringRedisTemplate.opsForValue().set("java","hello world");
	//String java = stringRedisTemplate.opsForValue().get("java");
	//System.out.println(java);
	//stringRedisTemplate.opsForList().leftPush("mylist", "1 2 3 4");
	//System.out.println(stringRedisTemplate.opsForList().leftPop("mylist"));

	Employee employee = employeeRepository.findById(1).get();
	//默认保存对象是使用jdk序列化器，序列化后保存到redis
	redisTemplate.opsForValue().set("emp",employee);
	//自己将对象转为Json
	//redisTemplate默认的序列化规则
}
```

自定义序列化规则的RedisTemplate
```java
@Configuration
public class MyRedisConfig {

    @Bean
    public RedisTemplate<Object, Employee> redisTemplate(RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setDefaultSerializer(new Jackson2JsonRedisSerializer<Employee>(Employee.class));
        return template;
    }
}
```

  
