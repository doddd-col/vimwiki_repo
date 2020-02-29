## Redis持久化
#### RDB(Redis DataBase)

 >在指定时间间隔内将内存中的数据集快照写入磁盘
>
>也就是说Snapshot快照，它恢复时是将快照文件直接读到内存
```
Redis会创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，等持久化结束，
在用这个临时文件替换上次持久化好的文件，整个过程中，主进程不进行任何IO操作，
确保了极高的性能，如果需要进行大规模的数据恢复，且对于数据恢复的完整性不是非常敏感，
那RDB方式要比AOF方式更加高效，RDB的缺点是最后一次持久化后的数据可能丢失

#RDB保存的是dump.rdb文件(在redis.conf文件中可以在dbfilename修改持久化文件名字)
#重启是会快速从dump.rdb文件中恢复
#shutdown/flushall会触发持久化

```
**总的说，对精度要求不高最好的就是RDB**  

```
触发快照(Snapshot)
redis.conf 里面 save设置
默认是:
	- save 900 1		#900秒(15分钟)内key改变过一次
	- save 300 10		#5分钟
	- save 60 10000		#1分钟
禁用: save ""
手动直接备份: save/bgsave
	- save : 只管保存，全部堵塞
	- bgsave : 后台异步进行快照操作，同时响应客户端请求，通过lastsave命令获取最后一次成功执行快找时间

动态停止所有RDB备份: redis-cli config set save ""


```
#### AOF(Append Only File)

```
以日志的形式来记录每个写操作,将Redis执行过程的所有写指令记录下来（读操作不记录），
只需追加文件不可以改写文件，redis启动之初会读取该文件重新构建数据，也就是说，
redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

#flushall也会被记录
```

>appendonly.aof与dump.rdb

**AOF and RDB persistence can be enabled at the same time without problems**  
**If the AOF is enabled on startup Redis will load the AOF, that is the file**  
**with the batter durability guarantees**  

两者文件损坏可以用Redis-check-aof/dump --fix修复

```
appendonly no 默认是关闭的，因为RDB可以满足大部分应用

appendfsync 分为三种模式:
	- always 每次变化都会被记录到磁盘 Slow Safest
	- everysec 每秒记录，异步操作，默认推荐(If unsure, use "everysec")
	- no 不记录

no-appendfsync-on-rewrite no rewrite是否要使用appendfsync
```

**Rewrite**
```
AOF采用文件追加方式，文件会越来越大为了避免出现这种情况,
新增了重写机制，当AOF文件大小超过索设定的阈值时，Redis就会启动AOF文件的内容压缩，
只保留可以恢复数据的最小命令集，可以使用明镜bgrewriteaof
```

重写原理
```
AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后rename)
遍历新进程的内存数据，每条记录有一条Set语句。重写aof文件的操作，并没有读取旧的aof文件,
而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似
```

重写机制
```
Redis会记录上次重写的AOF大小，默认配置是当AOF文件大小是rewrite后大小的一倍且文件大于64M是触发
```

 
