### Commands

- SUBSCRIBE channel [channel ...]
- PUBLISH channel message
- UNSUBSCRIBE [channel [channel ...]]
- PSUBSCRIBE pattern [pattern ...] 正则订阅
- PUBSUB subcommand [argument [argument ...]] 查看订阅与发布系统状态
- PUNSUBSCRIBE [pattern [pattern ...]] 退订所有给定模式的频道

### 原理

Redis是使用C实现的，通过分析 Redis 源码里的`pubsub.c`文件，了解发布和订阅机制的底层实现，籍此加深对 Redis 的理解。

Redis 通过 PUBLISH、SUBSCRIBE 和 PSUBSCRIBE 等命令实现发布和订阅功能。

通过 SUBSCRIBE 命令订阅某频道后，redis-server 里维护了一个字典，字典的键就是一个个channel，而字典的值则是一个链表，链表中保存了所有订阅这个 channel 的客户端。SUBSCRIBE 命令的关键，就是将客户端添加到给定 channel 的订阅链表中。

通过 PUBLISH 命令向订阅者发送消息，redis-server 会使用给定的频道作为键，在它所维护的 channel 字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者。
Pub/Sub 从字面上理解就是发布（Publish）与订阅（Subscribe），在Redis中，你可以设定对某一个kev值进行消息发布及消息阅，当一个key值上进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这一功能最明显的用法就是用作实时消息系统，比如普通的即时聊天，群聊等功能。

### 主从复制的作用：

- 数据冗余
- 故障恢复
- 负载均衡
- 高可用

原因：

- 单点故障
- 单台服务器内存有限，一般来说，单台最大不应该超过20G

### 配置

master服务器不用特殊配置

```bash
info replication # 查看集群主从信息
```

需要配置的项：

```ini
port 6379
daemonize yes # 打开后台运行
pidfile /var/run/reds_6379.pid
logfile "6379.log"
dbfilename dump6379.rdb
```

从机配置

从机ReadOnly无法写入，链式主从复制，中间的节点依旧属于Slave。

```bash
SLAVEOF host port # 使用命令进行配置
SLAVEOF no one # 从slave更新为master节点
```

```ini
replicaof <masterip> <masterport> # 使用配置项进行永久配置
```

### 复制原理

slave启动成功连接到 master 后会发送一个sync同步命令

master 接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，并完成一次完全同步。

全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。

增量复制:：master 继续将新的所有收集到的修改命令依次传给slave，完成同步但是只要是重新连接master，一次完全同步（全量复制）将被自动执行。

### 哨兵模式

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令等待Redis服务器响应从而监控运行的多个Redis实例。

多哨兵模式：

假设主服务器宕机，哨兵1先检测到这个结巢，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象成为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover[故障转移]操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。

conf配置：

```ini
sentinel monitor <master-name> <ip> <redis-port> <quorum>
# 告诉Sentinel监听指定主节点，并且只有在至少<quorum>哨兵达成一致的情况下才会判断它 O_DOWN 状态
```

```
redis-sentinel kconfig/sentinel.conf # 启动sentinel
```

故障下线又恢复的master节点会变成slave。

全部配置项：

```ini
# Example sentinel.conf

# *** IMPORTANT ***
# 绑定IP地址
# bind 127.0.0.1 192.168.1.1
# 保护模式（是否禁止外部链接，除绑定的ip地址外）
# protected-mode no

# port <sentinel-port>
# 此Sentinel实例运行的端口
port 26379

# 默认情况下，Redis Sentinel不作为守护程序运行。 如果需要，可以设置为 yes。
daemonize no

# 启用守护进程运行后，Redis将在/var/run/redis-sentinel.pid中写入一个pid文件
pidfile /var/run/redis-sentinel.pid

# 指定日志文件名。 如果值为空，将强制Sentinel日志标准输出。守护进程下，如果使用标准输出进行日志记录，则日志将发送到/dev/null
logfile ""

# sentinel announce-ip <ip>
# sentinel announce-port <port>
#
# 上述两个配置指令在环境中非常有用，因为NAT可以通过非本地地址从外部访问Sentinel。
#
# 当提供announce-ip时，Sentinel将在通信中声明指定的IP地址，而不是像通常那样自动检测本地地址。
#
# 类似地，当提供announce-port 有效且非零时，Sentinel将宣布指定的TCP端口。
#
# 这两个选项不需要一起使用，如果只提供announce-ip，Sentinel将宣告指定的IP和“port”选项指定的服务器端口。
# 如果仅提供announce-port，Sentinel将通告自动检测到的本地IP和指定端口。
#
# Example:
#
# sentinel announce-ip 1.2.3.4

# dir <working-directory>
# 每个长时间运行的进程都应该有一个明确定义的工作目录。对于Redis Sentinel来说，/tmp就是自己的工作目录。
dir /tmp

# sentinel monitor <master-name> <ip> <redis-port> <quorum>
#
# 告诉Sentinel监听指定主节点，并且只有在至少<quorum>哨兵达成一致的情况下才会判断它 O_DOWN 状态。
#
#
# 副本是自动发现的，因此您无需指定副本。
# Sentinel本身将重写此配置文件，使用其他配置选项添加副本。另请注意，当副本升级为主副本时，将重写配置文件。
#
# 注意：主节点（master）名称不能包含特殊字符或空格。
# 有效字符可以是 A-z 0-9 和这三个字符 ".-_".
sentinel monitor mymaster 127.0.0.1 6379 2

# 如果redis配置了密码，那这里必须配置认证，否则不能自动切换
# Example:
#
# sentinel auth-pass mymaster MySUPER--secret-0123passw0rd

# sentinel down-after-milliseconds <master-name> <milliseconds>
#
# 主节点或副本在指定时间内没有回复PING，便认为该节点为主观下线 S_DOWN 状态。
#
# 默认是30秒
sentinel down-after-milliseconds mymaster 30000

# sentinel parallel-syncs <master-name> <numreplicas>
#
# 在故障转移期间，多少个副本节点进行数据同步
sentinel parallel-syncs mymaster 1

# sentinel failover-timeout <master-name> <milliseconds>
#
# 指定故障转移超时（以毫秒为单位）。 它以多种方式使用：
#
# - 在先前的故障转移之后重新启动故障转移所需的时间已由给定的Sentinel针对同一主服务器尝试，是故障转移超时的两倍。
#
# - 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#
# - 取消已在进行但未生成任何配置更改的故障转移所需的时间
#
# - 当进行failover时，配置所有slaves指向新的master所需的最大时间。
#   即使过了这个超时，slaves依然会被正确配置为指向master。
#
# 默认3分钟
sentinel failover-timeout mymaster 180000

# 脚本执行
#
# sentinel notification-script和sentinel reconfig-script用于配置调用的脚本，以通知系统管理员或在故障转移后重新配置客户端。
# 脚本使用以下规则执行以进行错误处理：
#
# 如果脚本以“1”退出，则稍后重试执行（最多重试次数为当前设置的10次）。
#
# 如果脚本以“2”（或更高的值）退出，则不会重试执行。
#
# 如果脚本因为收到信号而终止，则行为与退出代码1相同。
#
# 脚本的最长运行时间为60秒。 达到此限制后，脚本将以SIGKILL终止，并重试执行。

# 通知脚本
#
# sentinel notification-script <master-name> <script-path>
#
# 为警告级别生成的任何Sentinel事件调用指定的通知脚本（例如-sdown，-odown等）。
# 此脚本应通过电子邮件，SMS或任何其他消息传递系统通知系统管理员 监控的Redis系统出了问题。
#
# 使用两个参数调用脚本：第一个是事件类型，第二个是事件描述。
#
# 该脚本必须存在且可执行，以便在提供此选项时启动sentinel。
#
# 举例:
#
# sentinel notification-script mymaster /var/redis/notify.sh

# 客户重新配置脚本
#
# sentinel client-reconfig-script <master-name> <script-path>
#
# 当主服务器因故障转移而变更时，可以调用脚本执行特定于应用程序的任务，以通知客户端，配置已更改且主服务器地址已经变更。
#
# 以下参数将传递给脚本：
#
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
#
# <state> 目前始终是故障转移 "failover"
# <role> 是 "leader" 或 "observer"
#
# 参数 from-ip, from-port, to-ip, to-port 用于传递主服务器的旧地址和所选副本的新地址。
#
# 举例:
#
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh

# 安全
# 避免脚本重置，默认值yes
# 默认情况下，SENTINEL SET将无法在运行时更改notification-script和client-reconfig-script。
# 这避免了一个简单的安全问题，客户端可以将脚本设置为任何内容并触发故障转移以便执行程序。
sentinel deny-scripts-reconfig yes

# REDIS命令重命名
#
#
# 在这种情况下，可以告诉Sentinel使用不同的命令名称而不是正常的命令名称。
# 例如，如果主“mymaster”和相关副本的“CONFIG”全部重命名为“GUESSME”，我可以使用：
#
# SENTINEL rename-command mymaster CONFIG GUESSME
#
# 设置此类配置后，每次Sentinel使用CONFIG时，它将使用GUESSME。 请注意，实际上不需要尊重命令案例，因此在上面的示例中写“config guessme”是相同的。
#
# SENTINEL SET也可用于在运行时执行此配置。
#
# 为了将命令设置回其原始名称（撤消重命名），可以将命令重命名为它自身：
#
# SENTINEL rename-command mymaster CONFIG CONFIGss
```