### mysql
1.　阿里电商平台峰值，下单３０ｗ+/s ,支付20w+/s,数据库峰值2400w+/s,数据库节点3000左右；
2. 春晚摇一摇8.1亿/s
3.　mｙsql单表数据超２０００ｗ，性能急剧下降
mysqlslap mysql自带压测工具

#### mysql集群解决方案
1. mycat数据切片(有自己的切分规则)
2. mycat下面链接多个片，这个片指的是有负载均衡的数据库集群，一般是用ｈａ代理到不同的mysql实例(他们之前做同步)(可能是PXC也可能是RP)
3. 保存珍贵数据，采用ＰＸＣ，保存海量数据才有RP，他们都用mycat管理

#### 微服务之间通讯
1. 异步，采用消息队列,也算是业务解耦了
2. 同步,接口调用

#### PXC Percona XtraDB Cluster (保证了强一致性，牺牲速度)
1. PXC 解释：通过mysql自带的galera技术把不同的mysql链接起来，每个节点都可以读写,自动同步
2. RP (Replication)读写分离，一个节点写，其他节点读,同步是单向的
3. PXC同步机理(数据强一致)，请求到某个机器实例的时候，先逻辑写，然后提交事物，这台机器会生成一个GTID，把这组操作再其他机器依次执行，当返回结果全部正常，然后自己再存到磁盘彻底持久化。如果有不成功的，则回滚。
4. PXC和RP互补，可以结合，但是没有数据分片功能
5. pxc用的时候集群小点，会快
6. 低价值用RP，敏感用PXC

##### 插入sql语句
1. 传统的，insert into T values(***)
2. 方言，insert into T SET a=b，c=d

#### RP 操作
1. 显示同步状态，在丛机`SHOW SLAVE STATUS`
2. 停止从同步 `STOP SLAVE`
3. 停止了之后，主可以继续更新，但是丛机数据没有变化

#### PXC 操作
1. 插入数据前，把部分机器的的端口内部禁止，或者其他办法让数据不能插入，此时进写操作时候提示：`WSREP has not yet prepared node for application use`
2. 并发向多台主机发送insert最后数据肯定同步了，也是唯一的，但是主键不一定是连续的，因为写操作的时候会占用某个主键，同步失败，这样就用下个主键了。想要避免则使用mycat这种中间件

#### PXC集群搭建建议(haproxy篇)
1. 采用奇数个机器(偶数容易脑裂)
![avatar](./ha-admin-tcp.jpg)
