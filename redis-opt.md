Redis Cluster 安装部署
参考 https://blog.csdn.net/zimu312500/article/details/123466423

OS 8.0
Redis  7.0
Redis node1 Master 127.0.0.1 6401 /data/redis/data/6401
Redis node1 Slave 127.0.0.1 6402 /data/redis/data/6402
Redis node2 Master 127.0.0.1 6403 /data/redis/data/6403
Redis node2 Slave 127.0.0.1 6404 /data/redis/data/6404
Redis node3 Master 127.0.0.1 6405 /data/redis/data/6405
Redis node3 Slave 127.0.0.1 6406 /data/redis/data/6406


wget https://download.redis.io/redis-stable.tar.gz
tar -xzvf redis-stable.tar.gz
cd redis-stable
make

mkdir -p /data/redis/{bin,data,conf,log}
mkdir -p /data/redis/data/{6401..6406}
make PREFIX=/data/redis/bin install

./redis-server --version
Redis server v=7.0.2 sha=00000000:0 malloc=jemalloc-5.2.1 bits=64 build=317a0dc33f11edb5

提前准备好各实例的配置文件，以6401为例
bind 0.0.0.0
port 6401
pidfile /var/run/redis_6401.pid
logfile ""/data/redis/log/redis_6401.log""
dir /data/redis/data/6401
cluster-enabled yes
cluster-node-timeout 15000
cluster-config-file ""nodes-6401.conf""
daemonize yes
protected-mode yes
tcp-backlog 511
timeout 0
tcp-keepalive 300
supervised no
loglevel notice
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
replica-priority 100
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
appendonly no
appendfilename ""appendonly.aof""
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events """"
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes

启动各Redis实例
redis-server /data/redis/conf/redis_6401.conf
redis-server /data/redis/conf/redis_6402.conf
redis-server /data/redis/conf/redis_6403.conf
redis-server /data/redis/conf/redis_6404.conf
redis-server /data/redis/conf/redis_6405.conf
redis-server /data/redis/conf/redis_6406.conf

搭建Redis Cluster
# cluster-replicas 设置为1表示分配1个Slave节点
[root@VM-32-26-centos bin]# ./redis-cli --cluster-replicas 1 --cluster create 127.0.0.1:6401 127.0.0.1:6402 127.0.0.1:6403 127.0.0.1:6404 127.0.0.1:6405 127.0.0.1:6406
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:6405 to 127.0.0.1:6401
Adding replica 127.0.0.1:6406 to 127.0.0.1:6402
Adding replica 127.0.0.1:6404 to 127.0.0.1:6403
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 059779536a3fcbdd3e326f84dacc9bc5db52fa5b 127.0.0.1:6401
   slots:[0-5460] (5461 slots) master
M: b9aef8906f61806de8f84cfa6f90cde007c0bc9f 127.0.0.1:6402
   slots:[5461-10922] (5462 slots) master
M: 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 127.0.0.1:6403
   slots:[10923-16383] (5461 slots) master
S: c63b93805c90125453773de1458fd00ee3f26ae4 127.0.0.1:6404
   replicates 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c
S: a6fb5f4722d526a1a96d434ea2d66b700c3a7ed2 127.0.0.1:6405
   replicates 059779536a3fcbdd3e326f84dacc9bc5db52fa5b
S: 23b3d4319ab60bafb545bbfbaa963b2b56fad120 127.0.0.1:6406
   replicates b9aef8906f61806de8f84cfa6f90cde007c0bc9f
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 127.0.0.1:6401)
M: 059779536a3fcbdd3e326f84dacc9bc5db52fa5b 127.0.0.1:6401
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: c63b93805c90125453773de1458fd00ee3f26ae4 127.0.0.1:6404
   slots: (0 slots) slave
   replicates 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c
S: 23b3d4319ab60bafb545bbfbaa963b2b56fad120 127.0.0.1:6406
   slots: (0 slots) slave
   replicates b9aef8906f61806de8f84cfa6f90cde007c0bc9f
M: 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 127.0.0.1:6403
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: b9aef8906f61806de8f84cfa6f90cde007c0bc9f 127.0.0.1:6402
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: a6fb5f4722d526a1a96d434ea2d66b700c3a7ed2 127.0.0.1:6405
   slots: (0 slots) slave
   replicates 059779536a3fcbdd3e326f84dacc9bc5db52fa5b
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
[root@VM-32-26-centos bin]# 

检查集群是否搭建完成、槽完全被分配
[root@VM-32-26-centos bin]# ./redis-cli --cluster check 127.0.0.1:6401
127.0.0.1:6401 (05977953...) -> 0 keys | 5461 slots | 1 slaves.
127.0.0.1:6403 (75fea4cb...) -> 0 keys | 5461 slots | 1 slaves.
127.0.0.1:6402 (b9aef890...) -> 0 keys | 5462 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 127.0.0.1:6401)
M: 059779536a3fcbdd3e326f84dacc9bc5db52fa5b 127.0.0.1:6401
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: c63b93805c90125453773de1458fd00ee3f26ae4 127.0.0.1:6404
   slots: (0 slots) slave
   replicates 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c
S: 23b3d4319ab60bafb545bbfbaa963b2b56fad120 127.0.0.1:6406
   slots: (0 slots) slave
   replicates b9aef8906f61806de8f84cfa6f90cde007c0bc9f
M: 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 127.0.0.1:6403
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: b9aef8906f61806de8f84cfa6f90cde007c0bc9f 127.0.0.1:6402
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: a6fb5f4722d526a1a96d434ea2d66b700c3a7ed2 127.0.0.1:6405
   slots: (0 slots) slave
   replicates 059779536a3fcbdd3e326f84dacc9bc5db52fa5b
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
[root@VM-32-26-centos bin]# 


添加新节点到集群
## 添加新节点到集群，第一个地址为新节点IP和端口，第二个地址为集群中任意存在的节点即可
./redis-cli -p 6401 --cluster add-node 127.0.0.1:7401 127.0.0.1:6401
>>> Adding node 127.0.0.1:7401 to cluster 127.0.0.1:6401
>>> Performing Cluster Check (using node 127.0.0.1:6401)
M: 059779536a3fcbdd3e326f84dacc9bc5db52fa5b 127.0.0.1:6401
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: c63b93805c90125453773de1458fd00ee3f26ae4 127.0.0.1:6404
   slots: (0 slots) slave
   replicates 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c
S: 23b3d4319ab60bafb545bbfbaa963b2b56fad120 127.0.0.1:6406
   slots: (0 slots) slave
   replicates b9aef8906f61806de8f84cfa6f90cde007c0bc9f
M: 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 127.0.0.1:6403
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: b9aef8906f61806de8f84cfa6f90cde007c0bc9f 127.0.0.1:6402
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: a6fb5f4722d526a1a96d434ea2d66b700c3a7ed2 127.0.0.1:6405
   slots: (0 slots) slave
   replicates 059779536a3fcbdd3e326f84dacc9bc5db52fa5b
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Getting functions from cluster
>>> Send FUNCTION LIST to 127.0.0.1:7401 to verify there is no functions in it
>>> Send FUNCTION RESTORE to 127.0.0.1:7401
>>> Send CLUSTER MEET to node 127.0.0.1:7401 to make it join the cluster.
[OK] New node added correctly.
[root@VM-32-26-centos bin]# 

## 新节点添加成功后，可以使用 cluster nodes命令查看集群节点列表信息
./redis-cli -p 6401 cluster nodes
c63b93805c90125453773de1458fd00ee3f26ae4 127.0.0.1:6404@16404 slave 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 0 1655552360248 3 connected
23b3d4319ab60bafb545bbfbaa963b2b56fad120 127.0.0.1:6406@16406 slave b9aef8906f61806de8f84cfa6f90cde007c0bc9f 0 1655552360000 2 connected
18d36784879469a0c26d867193b6098360b473e6 127.0.0.1:7401@17401 master - 0 1655552361000 0 connected
75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 127.0.0.1:6403@16403 master - 0 1655552361251 3 connected 10923-16383
059779536a3fcbdd3e326f84dacc9bc5db52fa5b 127.0.0.1:6401@16401 myself,master - 0 1655552358000 1 connected 0-5460
b9aef8906f61806de8f84cfa6f90cde007c0bc9f 127.0.0.1:6402@16402 master - 0 1655552360000 2 connected 5461-10922
a6fb5f4722d526a1a96d434ea2d66b700c3a7ed2 127.0.0.1:6405@16405 slave 059779536a3fcbdd3e326f84dacc9bc5db52fa5b 0 1655552359000 1 connected
[root@VM-32-26-centos bin]# 

给新节点分配槽
Redis Cluster集群如果16384个槽全部被分配，那么分配槽给新加节点则需要使用reshard命令
 ./redis-cli -p 6401 --cluster  reshard 127.0.0.1:6401
>>> Performing Cluster Check (using node 127.0.0.1:6401)
M: 059779536a3fcbdd3e326f84dacc9bc5db52fa5b 127.0.0.1:6401
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: c63b93805c90125453773de1458fd00ee3f26ae4 127.0.0.1:6404
   slots: (0 slots) slave
   replicates 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c
S: 23b3d4319ab60bafb545bbfbaa963b2b56fad120 127.0.0.1:6406
   slots: (0 slots) slave
   replicates b9aef8906f61806de8f84cfa6f90cde007c0bc9f
M: 18d36784879469a0c26d867193b6098360b473e6 127.0.0.1:7401
   slots: (0 slots) master
M: 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 127.0.0.1:6403
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: b9aef8906f61806de8f84cfa6f90cde007c0bc9f 127.0.0.1:6402
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: a6fb5f4722d526a1a96d434ea2d66b700c3a7ed2 127.0.0.1:6405
   slots: (0 slots) slave
   replicates 059779536a3fcbdd3e326f84dacc9bc5db52fa5b
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 2500
What is the receiving node ID? 18d36784879469a0c26d867193b6098360b473e6
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: all

Ready to move 2500 slots.
  Source nodes:
    M: 059779536a3fcbdd3e326f84dacc9bc5db52fa5b 127.0.0.1:6401
       slots:[0-5460] (5461 slots) master
       1 additional replica(s)
    M: 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 127.0.0.1:6403
       slots:[10923-16383] (5461 slots) master
       1 additional replica(s)
    M: b9aef8906f61806de8f84cfa6f90cde007c0bc9f 127.0.0.1:6402
       slots:[5461-10922] (5462 slots) master
       1 additional replica(s)
  Destination node:
    M: 18d36784879469a0c26d867193b6098360b473e6 127.0.0.1:7401
       slots: (0 slots) master
  Resharding plan:
    Moving slot 5461 from b9aef8906f61806de8f84cfa6f90cde007c0bc9f
    Moving slot 5462 from b9aef8906f61806de8f84cfa6f90cde007c0bc9f
 ……
    Moving slot 11755 from 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c
Do you want to proceed with the proposed reshard plan (yes/no)? yes
Moving slot 5461 from 127.0.0.1:6402 to 127.0.0.1:7401: 
Moving slot 5462 from 127.0.0.1:6402 to 127.0.0.1:7401: 
……
Moving slot 11755 from 127.0.0.1:6403 to 127.0.0.1:7401: 
[root@VM-32-26-centos bin]# 

## 分配完毕后，通过cluster nodes查看槽位分配情况
./redis-cli -p 6401 cluster nodes
c63b93805c90125453773de1458fd00ee3f26ae4 127.0.0.1:6404@16404 slave 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 0 1655552726282 3 connected
23b3d4319ab60bafb545bbfbaa963b2b56fad120 127.0.0.1:6406@16406 slave b9aef8906f61806de8f84cfa6f90cde007c0bc9f 0 1655552726000 2 connected
18d36784879469a0c26d867193b6098360b473e6 127.0.0.1:7401@17401 master - 0 1655552727000 7 connected 0-832 5461-6294 10923-11755
75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 127.0.0.1:6403@16403 master - 0 1655552727284 3 connected 11756-16383
059779536a3fcbdd3e326f84dacc9bc5db52fa5b 127.0.0.1:6401@16401 myself,master - 0 1655552724000 1 connected 833-5460
b9aef8906f61806de8f84cfa6f90cde007c0bc9f 127.0.0.1:6402@16402 master - 0 1655552728287 2 connected 6295-10922
a6fb5f4722d526a1a96d434ea2d66b700c3a7ed2 127.0.0.1:6405@16405 slave 059779536a3fcbdd3e326f84dacc9bc5db52fa5b 0 1655552726000 1 connected
[root@VM-32-26-centos bin]# 

新节点置为从库
第一种是随机被分配到从库较少的主节点（--cluster-slave）
./redis-cli -p 6401 --cluster add-node 127.0.0.1:7402 127.0.0.1:6401 --cluster-slave
>>> Adding node 127.0.0.1:7402 to cluster 127.0.0.1:6401
>>> Performing Cluster Check (using node 127.0.0.1:6401)
M: 059779536a3fcbdd3e326f84dacc9bc5db52fa5b 127.0.0.1:6401
   slots:[833-5460] (4628 slots) master
   1 additional replica(s)
S: c63b93805c90125453773de1458fd00ee3f26ae4 127.0.0.1:6404
   slots: (0 slots) slave
   replicates 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c
S: 23b3d4319ab60bafb545bbfbaa963b2b56fad120 127.0.0.1:6406
   slots: (0 slots) slave
   replicates b9aef8906f61806de8f84cfa6f90cde007c0bc9f
M: 18d36784879469a0c26d867193b6098360b473e6 127.0.0.1:7401
   slots:[0-832],[5461-6294],[10923-11755] (2500 slots) master
M: 75fea4cb3b2622fcd2b983ecd65ef7d3091b8c2c 127.0.0.1:6403
   slots:[11756-16383] (4628 slots) master
   1 additional replica(s)
M: b9aef8906f61806de8f84cfa6f90cde007c0bc9f 127.0.0.1:6402
   slots:[6295-10922] (4628 slots) master
   1 additional replica(s)
S: a6fb5f4722d526a1a96d434ea2d66b700c3a7ed2 127.0.0.1:6405
   slots: (0 slots) slave
   replicates 059779536a3fcbdd3e326f84dacc9bc5db52fa5b
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
Automatically selected master 127.0.0.1:7401
>>> Send CLUSTER MEET to node 127.0.0.1:7402 to make it join the cluster.
Waiting for the cluster to join

>>> Configure node as replica of 127.0.0.1:7401.
[OK] New node added correctly.
[root@VM-32-26-centos bin]# 

第二种指定主节点建立复制关系，灾备场景、跨机房场景非常适用（--cluster-slave --cluster-master-id node_id）

## 除--cluster-slave外，还需使用--cluster-master-id指定需要复制的主节点node_id，该示例选择7401该主节点作为新节点的主库
# redis-cli -p 6401 --cluster add-node 127.0.0.1:7402 127.0.0.1:6401 --cluster-slave --cluster-master-id 18d36784879469a0c26d867193b6098360b473e6


集群全局命令
cluster nodes 列出Redis Cluster各节点信息与槽位分布
redis-cli -p 6401 cluster nodes

cluster info 输出集群整体信息
redis-cli -p 6401 cluster info

集群节点命令
cluster replicate 将当前节点与指定node_id的主节点建立复制
cluster forget 将指定node_id的节点从集群中移除
[root@VM-32-26-centos bin]# ./redis-cli -c -h 127.0.0.1 -p 6401 cluster forget c0331f8d562a992930b14504ce68390407354ce4
OK
[root@VM-32-26-centos bin]# 

ps -ef|grep 7402
kill 1649516
rm -rf //data/redis/data/7402/*
./redis-server /data/redis/conf/redis_7402.conf
./redis-cli -p 6401 --cluster add-node 127.0.0.1:7402 127.0.0.1:6401 --cluster-slave --cluster-master-id 18d36784879469a0c26d867193b6098360b473e6
==============================================================================================================================================================================
redis主从哨兵和集群的区别
一、架构不同
　　redis主从：一主多从；
　　redis集群：多主多从；
二、存储不同
　　redis主从：主节点和从节点都是存储所有数据；
　　redis集群：数据的存储是通过hash计算16384的槽位，算出要将数据存储的节点，然后进行存储；
三、选举不同
　　redis主从：通过启动redis自带的哨兵（sentinel）集群进行选举，也可以是一个哨兵
　　　　选举流程：1、先发现主节点fail的哨兵，将成为哨兵中的leader，之后的主节点选举将通过这个leader进行故障转移操作，从存活的slave中选举新的master，新　　的master选举同集群的master节点选举类似；
　　redis集群：集群可以自己进行选举
　　　　选举流程：1、当主节点挂掉，从节点就会广播该主节点fail；
　　　　　　　　　2、延迟时间后进行选举（延迟的时间算法为：延迟时间+随机数+rank*1000，从节点数据越多，rank越小，因为主从数据复制是异步进行的，所以　　所有的从节点的数据可能会不同），
     延迟的原因是等待主节点fail广播到所有存活的主节点，否则主节点会拒绝参加选举；
　　　　　　　　　3、参加选举的从节点向所有的存活的节点发送ack请求，但只有主节点会回复它，并且主节点只会回复第一个到达参加选举的从节点，一半以上的主节点回复，
     该节点就会成为主节点，广播告诉其他节点该节点成为主节点。
四、节点扩容不同
　　redis主从：只能扩容从节点，无法对主节点进行扩容；
　　redis集群：可以扩容整个主从节点，但是扩容后需要进行槽位的分片，否则无法进行数据写入
==============================================================================================================================================================================
常用操作
关闭、启动
关闭 /apps/svr/redis-2.8.19/bin/redis-cli -p 6370 shutdown
启动 /apps/svr/redis-2.8.19/bin/redis-server /apps/conf/redis/redis7900.conf

基本命令
#查看所有key
keys *  或  keys ""*""
#查看匹配前缀的keys
keys ""miao*""
#清空redis
flushdb
#随机取出一个key
randomkey
#查看key的类型
type key
#查看数据库中key的数量
dbsize
#查看服务器信息
info
#查看redis正在做什么
monitor
#查看日志
slowlog get
slowlog get 10


redis-cli -p 6381  config set maxmemory 8589934592
redis-cli -p 6381  config get maxmemory

127.0.0.1:6401> client list
id=6 addr=127.0.0.1:59288 laddr=127.0.0.1:6401 fd=20 name= age=3888 idle=1 flags=S db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=20474 argv-mem=0 multi-mem=0 rbs=1024 rbp=0 obl=0 oll=1 omem=20504 tot-me2
id=39 addr=127.0.0.1:39192 laddr=127.0.0.1:6401 fd=22 name= age=627 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 qbuf-free=20448 argv-mem=10 multi-mem=0 rbs=1024 rbp=0 obl=0 oll=0 omem=0 tot-mem=2

==============================================================================================================================================================================
查找大KEY
[root@VM-32-26-centos bin]# ./redis-cli -p 6401 --bigkeys

# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).


-------- summary -------

Sampled 0 keys in the keyspace!
Total key length in bytes is 0 (avg len 0.00)


0 hashs with 0 fields (00.00% of keys, avg size 0.00)
0 lists with 0 items (00.00% of keys, avg size 0.00)
0 strings with 0 bytes (00.00% of keys, avg size 0.00)
0 streams with 0 entries (00.00% of keys, avg size 0.00)
0 sets with 0 members (00.00% of keys, avg size 0.00)
0 zsets with 0 members (00.00% of keys, avg size 0.00)
[root@VM-32-26-centos bin]# 
==============================================================================================================================================================================
Redis数据类型

字符串
Redis中的字符串是一个字节序列。Redis中的字符串是二进制安全的，这意味着它们的长度不由任何特殊的终止字符决定。因此，可以在一个字符串中存储高达512M字节的任何内容。
127.0.0.1:6401> set name ""yiibai.com"" 
(error) MOVED 5798 127.0.0.1:7401
127.0.0.1:6401> get name
(error) MOVED 5798 127.0.0.1:7401

散列/哈希  每个散列/哈希可以存储多达2^32 - 1个健-值对(超过40亿个)
127.0.0.1:6401> HMSET ukey username ""yiibai"" password ""passswd123"" points 200
OK
127.0.0.1:6401> hmget ukey password
1) ""passswd123""
127.0.0.1:6401> HGETALL ukey
1) ""username""
2) ""yiibai""
3) ""password""
4) ""passswd123""
5) ""points""
6) ""200""

列表
列表的最大长度为2^32 - 1个元素(4294967295，每个列表可容纳超过40亿个元素)
Redis列表只是字符串列表，按插入顺序排序。您可以向Redis列表的头部或尾部添加元素
127.0.0.1:6401> lpush alist redis 
(integer) 1
127.0.0.1:6401> lpush alist mongodb 
(integer) 2
127.0.0.1:6401> lrange alist 0 1
1) ""mongodb""
2) ""redis""
127.0.0.1:6401> lrange alist 0 0
1) ""mongodb""
127.0.0.1:6401> 

集合
一个集合中的最大成员数量为2^32 - 1(即4294967295，每个集合中元素数量可达40亿个)个
redis 127.0.0.1:6379> sadd yiibailist redis 
(integer) 1 
redis 127.0.0.1:6379> sadd yiibailist mongodb 
(integer) 1 
redis 127.0.0.1:6379> sadd yiibailist sqlite 
(integer) 1 
redis 127.0.0.1:6379> sadd yiibailist sqlite 
(integer) 0 
redis 127.0.0.1:6379> smembers yiibailist  
1) ""sqlite"" 
2) ""mongodb"" 
3) ""redis""

可排序集合
redis 127.0.0.1:6379> zadd yiibaiset 0 redis
(integer) 1 
redis 127.0.0.1:6379> zadd yiibaiset 0 mongodb
(integer) 1 
redis 127.0.0.1:6379> zadd yiibaiset 1 sqlite
(integer) 1 
redis 127.0.0.1:6379> zadd yiibaiset 1 sqlite
(integer) 0 
redis 127.0.0.1:6379> ZRANGEBYSCORE yiibaiset 0 1000  
1) ""mongodb"" 
2) ""redis"" 
3) ""sqlite""

==============================================================================================================================================================================
REDIS跟MYSQL数据同步
1、先清除缓存，再更新数据库的方式显然是不行的，可能存在数据永远不正确的情况。
2、先更新数据库再清缓存的方式，虽然可能会存在少数的错误数据的情况，但是相对来说，后续的查询可以得到更新的值。


击穿：redis缓存击穿是指某一个非常热点的key(即在客户端搜索的比较多的关键字)突然失效了,这时从客户端发送的大量的请求在redis里找不到这个key，就会去数据里找，最终导致数据库压力过大崩掉。
 解决：
  1.将value的时效设置成永不过期 这种方式非常简单粗暴但是安全可靠。但是非常占用空间对内存消耗也是极大。个人并不建议使用该方法，应该根据具体业务逻辑来操作。
   穿透：因为不良用户恶意频繁查询才会对系统造成很大的问题: key缓存并且数据库不存在，所以每次查询都会查询数据库从而导致数据库崩溃。
  2.使用Timetask做一个定时任务 使用Timetask做定时，每隔一段时间对一些热点key进行数据库查询，将查询出的结果更新至redis中。前条件是不会给数据库过大的压力。
  3.通过synchronized+双重检查机制 当发生reids穿透的时候，这时海量请求发送到数据库。这时我们的解决办法是只让只让一个线程去查询这个热点key，其它线程保持阻塞状态(可以让它们sleep几秒)。
   当这个进入数据库的线程查询出key对应的value时，我们再将其同步至redis的缓存当中，其它线程睡醒以后再重新去redis里边请求数据
 解决：
  1.当类似的请求发过来，无论查出什么结果都放入redis缓存
  2.拉黑其ip
  3.对请求的参数进行合法性校验，在判断其不合法的前提下直接return掉
  4.使用布隆过滤器。布隆过滤器可能会造成误判，从而穿透redis进入DB，但是这个误判概率是非常小的。
雪崩：和击穿类似，不同的是击穿是一个热点key某时刻失效，而雪崩是大量的热点key在一瞬间失效
 解决：
  1.设置缓存时,随机初始化其失效时间。如果是redis的key同时失效,可采取该办法,具体失效时间根据业务情况决定…
  2.将不同的热点key放置到不同的节点上去。因redis一般都是集群部署,将不同的热点key平均的放置到不同节点,也可以有效避免雪崩。
  3.将value的时效设置成永不过期
  4.使用Timetask做一个定时任务，在失效之前重新刷redis缓存
  
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
==============================================================================================================================================================================
