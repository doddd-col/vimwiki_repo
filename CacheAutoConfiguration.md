## Cache自动配置原理
Cache的自动配置类:CacheAutoConfiguration


缓存的配置类:
org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration
org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration

>默认配置：SimpleCacheConfiguration 生效了

给容器中注册了一个CacheManager,ConcurrentMapCacheManager
可以获取和创建ConcurrentMapCache类型的缓存组件，它的作用将数据保存在ConcurrentMap中



**运行流程**:
@Cahcheable
1. 方法运行之前先去查询啊Cache（缓存组件), 按照cacheNames指定的名字获取 --CacheManager先获取相应的缓存, -
		-第一次获取缓存, 如果没有Cache组件会自动创建
```java
	@Override
	@Nullable
	public Cache getCache(String name) {
		Cache cache = this.cacheMap.get(name);
		if (cache == null && this.dynamic) {
			synchronized (this.cacheMap) {
				cache = this.cacheMap.get(name);
				if (cache == null) {
					// 第一次自动创建组件
					cache = createConcurrentMapCache(name);
					this.cacheMap.put(name, cache);
				}
			}
		}
		return cache;
	}
```
 
2. 去Cache中查找缓存的内容, 使用一个key默认是方法的参数
		--key是按照某种策略生成的，默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key
```java
	public static Object generateKey(Object... params) {
		// 没有参数返回一个空的key:public static final SimpleKey EMPTY = new SimpleKey();
		if (params.length == 0) {
			return SimpleKey.EMPTY;
		}
		if (params.length == 1) {
			Object param = params[0];
			if (param != null && !param.getClass().isArray()) {
				return param;
			}
		}
		// 多个参数 包装到SimpleKey
		return new SimpleKey(params);
	}
```

   
3. 没有查找到缓存就调用目标方法
4. 将目标方法结果放到缓存中
```java
	@Override
	public void put(Object key, @Nullable Object value) {
		this.store.put(key, toStoreValue(value));
	}
```

@Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存,如果没有就运行方法并将结果放入缓存

**核心**  
- 使用CacheManager[ConcurrentMapCacheManager]按照名字得到Cache[ConcurrentmapCache]组件
- key使用keyGenerator生成的，默认是SimpleKeyGenerator
