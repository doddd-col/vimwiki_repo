## Redis事务

可以一次执行多个命令，本质是一组命令的集合，一个事务中的所有命令都会序列化，按顺序的串行化执行而不会被其他命令插入，不许加塞

>一个队列中，一次性、顺序性、排他性的执行一系列命令


**Redis 事务命令**  
>- discard		--取消事务
>
>- exec			--执行事务
>
>- multi		--开启一个事务
>
>- unwatch		--取消watch对所有key的监控
>
>- watch key [key...]		--监视key，如果在事务执行之前被更改，则事务被打断


#一旦执行了exec之前加的监控都会被取消


**部分保证原子性**:
- 命令编译时写错全部失效
- 命令执行时出错只有执行出错的失败


**Redis事务三阶段**  
```
1. 开启：MULTI
2. 入队：输入命令返回QUEUED
3. 执行：EXEC
```

