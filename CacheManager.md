## CacheManager
CacheManager是缓存组件中实际给缓存存储数据的组件

1. 引入redis的starter，容器中保存的是RedisCacheManager
2. RedisCacheManager创建RedisCache作为缓存组件
3. 默认保存数据k-v 都是Object，利用序列化保存
		- 引入了redis的starter, cachemanager->RedisCahceManager
		- 默认创建的RedisCacheManager 操作redis的时候使用的是RedisTemplate<Object,Object>
		- RedisTemplate<Object, Object>是默认使用jbk的序列化机制


**引入Redis后创建RedisCache**
```java
/**
 * Configuration hook for creating {@link RedisCache} with given name and {@code cacheConfig}.
 *
 * @param name must not be {@literal null}.
 * @param cacheConfig can be {@literal null}.
 * @return never {@literal null}.
 */
protected RedisCache createRedisCache(String name, @Nullable RedisCacheConfiguration cacheConfig) {
	return new RedisCache(name, cacheWriter, cacheConfig != null ? cacheConfig : defaultCacheConfig);
}

```

**自定义RedisCacheManager修改RedisConfiguration默认序列化机制(jdk)**
```java
//RedisCacheManager实现了CacheManager接口所以可以直接返回CacheManager
@Bean
public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
  // 创建新的RedisCacheConfiguration,通过获取原始的默认配置进行函数式修改
  RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
		// 配置缓存生效时间
		.entryTtl(Duration.ofHours(1))
		// 禁止缓存null
		.disableCachingNullValues()
		// 用Jackson2序列化value
		.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
  //用builder方法传入工厂重新创建RedisCacheManager，调用cahceDefaults传入配置好的RedisConfiguration
  return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(redisCacheConfiguration).build();
}


# fastjson
@Bean
public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
	RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
			.entryTtl(Duration.ofHours(1))
			.disableCachingNullValues()
			.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
			.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new FastJsonRedisSerializer<Employee>(Employee.class)));
	return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(redisCacheConfiguration).build();
}
```

**cacheDefaults**

```java
/**
 * Define a default {@link RedisCacheConfiguration} applied to dynamically created {@link RedisCache}s.
 *
 * @param defaultCacheConfiguration must not be {@literal null}.
 * @return this {@link RedisCacheManagerBuilder}.
 */
public RedisCacheManagerBuilder cacheDefaults(RedisCacheConfiguration defaultCacheConfiguration) {

	Assert.notNull(defaultCacheConfiguration, "DefaultCacheConfiguration must not be null!");

	this.defaultCacheConfiguration = defaultCacheConfiguration;

	return this;
}
```

<++>
