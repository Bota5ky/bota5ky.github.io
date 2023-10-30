### 1. 什么是 Raft
相比于 Paxos，Raft 最大的特性就是易于理解，为了达到这个目标 Raft 主要做了两方面的事情：

- 问题分解：把共识算法分为三个子问题，分别是
  - 领导者选举（leader election）
  - 日志复制（log replication）
  - 安全性（safety）
- 状态简化：对算法做出一些限制，减少状态数量和可能产生的变动

**复制状态机的概念：相同的初始状态 + 相同的输入 = 相同的结束状态**

**状态简化：**

只有 leader、follower 或 candidate 三个状态之一。另外可以通过查看一台服务器是否具有某任期内的日志，来判断它是否在这期间出现过宕机。

Raft 算法中服务器节点之间使用 RPC 进行通信，并且 Raft 中只有两种主要的 RPC：

- **RequestVote RPC（请求投票）：**由 candidate 在选举期间发起
- **AppendEntries RPC（追加条目）：**由 leader 发起，用来复制日志和提供一种心跳机制

### 2. 领导者选举

开始一个选举过程后，follower **先增加自己的当前任期号**，并转换到 candidate 状态。**然后投票给自己**，并且并行地向集群中的其他服务器节点发送投票请求（RequestVote RPC）。

最终会有三种结果：

- 它获得超过半数选票赢得了选举 -> 成为主并开始发送心跳
- 其他节点赢得了选举 -> 收到新 leader 的心跳后，如果新 leader 的任期号不小于自己当前的任期号，那么就从 candidate 回到 follower 状态
- 一段时间之后没有任何获胜者 -> 每个 candidate 都在一个自己的随机选举超时时间后增加任期号开始新一轮投票

```c
//请求投票RPC Request
type RequestVoteRequest struct {
  term         int  //自己当前的任期号
  candidateld  int  //自己的ID
  lastLogindex int  //自已最后一个日志号
  lastLogTerm  int  //自己最后一个日志的任期
}
```

```c
// 请求投票RPC Response
type RequestVoteResponse struct {
  term         int  //自己当前任期号
  voteGranted  bool // 自己会不会投票给这个candidate
}
```

第一个认识到集群中没有 leader 的节点会把自己变成 candidate，对于没有成为 candidate 的 follower 节点，对于同一个任期，会按照**先来先得**的原则投出自己的选票。

### 3. 日志复制

客户端怎么知道新 leader 是哪个节点呢？客户端随机向一个节点发送请求：

- 节点正好是 leader
- 节点是 follower，可以通过心跳得知 leader 的 ID
- 节点正好宕机，客户端只能再去找另一个节点，重复上述过程

日志需要具有的三个信息：

- 状态机指令，比如赋值操作
- leader 的任期号
- 日志索引