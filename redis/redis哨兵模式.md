# redis哨兵模式

## 启动的顺序

主机-->从机-->哨兵

主机：能够正常的使用
从机：只能够读取，无法写入
哨兵：只能查看监管的redis数据

哨兵模式皆在于切换主从机的位置，如果设置好同步频率那数据就是一样的，但主从机切换了，端口也变动了，这需要重新发现主机的端口

## 配置顺序
配置一拖二的主从结构
配置三个哨兵（配置相同，端口不同）
monitor mymaster 127.0.0.1 6379 2		monitor表示监控，2表示如果有两个哨兵认为这个主机挂了，那他就挂了,之后将交由哨兵来选出新主机,mymaster是主机的名字，随便起的，后面紧跟着地址
down-after-milliseconds mymaster 30000	        此处表示主机超过30000没反应就判断为宕机
parallel-syncs mymaster 1			判断新任命的主机一次需要同步多少台副主机
failover-timeout mymaster 180000		判断多长时间同步不了算超时



redis-cli -h 127.0.0.1 -p 26379					redis连接数据库
redis-server redis-5.0.7/conf/sentinel-26379.conf --sentinel	运行哨兵模式     
redis-server redis-5.0.7/conf/redis-6379.conf			运行redis数据库,注意一定要加后面的--sentinel参数，否则会错误

哨兵命令行语句：
	info可以查看当前服务器信息

## 其他
port 6380			端口
daemonize no			不开启守护进程模式
dir ./redis-5.0.7/data		在redis下创建data目录，用来存放日志
logfile "6380.log"		使用日志，存放到当前文件的6380.log中
slaveof 127.0.0.1 6379		保存6379端口的数据
dbfilename dump-6379.rdb	保存数据库文件
save 10 2			表示10秒内有两次更新数据的操作

