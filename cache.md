## cache使用
1. 开启基于注解的缓存
	- @EnableCaching
2. 标注注解缓存
	- @Cacheable


>@Cacheable
将方法运行的结果进行缓存，以后再要相同的数据，直接从缓存中获取，
CacheManager管理多个Cache组件，对缓存的真正CRUD操作在Cache组件中，每一个组件有自己唯一的名称
几个属性:
	- cacheNames/value:指定缓存组件的名字,以数组的形式,可以放到多个缓存中  --value={"emp","dept"}
		* @Cacheable(cacheNames = "emp", condition = "#id>0", unless = "#result == null", key = "#root.methodName+'['+#id+']'")
```java
// 通过keyGenerator生成key
@Configuration
public class MyCacheConfig {

    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){
        return new KeyGenerator(){

            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName()+"["+ Arrays.asList(params)+"]";
            }
        };
    }

}

@Cacheable(cacheNames = "emp", condition = "#id>0", unless = "#result == null", keyGenerator = "myKeyGenerator")
```

		 
	- key:缓存数据所用的key，可以用它来指定，默认是方法使用的参数的值，1-方法的返回值
		* SpEL:#id 参数id的值   #a0 #p0 #root.args[0]
	- keyGenarator:key的生成器，可以自己指定key的生成器的组件id 
		* key/keyGenerator 二选一
	- cacheManager:指定缓存管理器
		* cacheManager/cacheResolver 二选一
	- condition:指定符合条件的情况下才能缓存
	- unless:否定缓存，当unless指定的条件为true，方法返回值就不会被缓存	
	- sync:是否使用同步模式，默认false
	

>@CachePut
**既调用方法，又更新数据**  
修改数据库的某个数据，同时更新缓存
运行时机：
	- 先调用目标方法
	- 再将目标方法的结果缓存起来



>@CacheEvict:缓存清除
按照指定key删除
	- allEntries=true:清除所有的数据
	- beforeInvocation=true:在方法执行之前清除(有可能方法异常仍然清除缓存)
以上两个属性默认是false


>@Caching
复杂的组合注解
```java
 @Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Caching {

	Cacheable[] cacheable() default {};

	CachePut[] put() default {};

	CacheEvict[] evict() default {};

}
```

 
```java

	@Caching(
			cacheable = {
					// 每次通过名字查询, 都会调用该注解下的方法，因为@CachePut一定会调用方法进行更新
					@Cacheable(key="#result.lastName")
			},
			put = {
					// 通过名字查询后将结果分别以id和email作为key缓存起来, 这样直接查询id可能在缓存中查到
					@CachePut(key = "#result.id"),
					@CachePut(key = "#result.email")
			}

	)
```

>@CacheConfig
给一个类抽取公共部分
```java
/**
 * {@code @CacheConfig} provides a mechanism for sharing common cache-related
 * settings at the class level.
 *
 * <p>When this annotation is present on a given class, it provides a set
 * of default settings for any cache operation defined in that class.
 *
 * @author Stephane Nicoll
 * @author Sam Brannen
 * @since 4.1
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CacheConfig {
```

 
实例
```java
@CacheConfig(cacheNames = "emp")
@Service
public class EmployeeService {

        @Autowired
        EmployeeRepository employeeRepository;

        /**
         * @Author doddd
         * @Description 查询员工
         * @Date 6:24 下午 2020/2/24
         * @Param [id]
         * @return com.doddd.cache.bean.Employee
         **/
        @Cacheable(condition = "#id>0", unless = "#result == null")
        public Employee queryEmpById(@NonNull Integer id){

                System.out.println("查询了id: "+id+" 的员工");
                Employee employee = employeeRepository.findById(id).get();
                return employee;
        }

        @CachePut(key = "#result.id")
        public Employee updateEmpById(Employee employee){
                Employee save = employeeRepository.save(employee);
                return save;
        }

        @CacheEvict(allEntries = true)
        public void deleteEmpById(Integer id ){
                System.out.println("delete:"+id);
        }

        @Caching(
                cacheable = {
                        @Cacheable()
                },
                put = {
                        @CachePut(key = "#result.id"),
                        @CachePut(key = "#result.email")
                }

        )
        public Employee queryEmpByLastName(){
                return null;
        }
}
```

 
