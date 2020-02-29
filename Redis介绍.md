#### 配置说明
- daemonize: 默认为no，修改为yes启用为守护线程（即后台运行）
- port: 端口号，默认6379
- bind：绑定ip
- databases: 数据库数量，默认16
- save <second> <changes>: 指定多少时间，有多少次更新操作，将数据同步到数据文件 #redis默认配置有三个条件，满足一个即可进行持久化
	* >save 900 1 #900s有一个更改
	* >save 300 10 #300s有10 个更改
- dbfilename: 指定本地数据库的文件名，默认是dump.rdb
- dir: 指定本地数据库存放目录，默认是./当前文件
- requirepass: 设置密码默认是关闭的
	* >redis -cli -h host -p port -a password
  
 
**五种数据类型**  
- String
- Map(类似java中的HashMap)
- List
- Set
- Zset
