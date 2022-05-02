Raft是分布式一致性协议之一。维护多个server之间的一致性。  
每个server都以Follower角色启动，根据不同条件切换为不同角色。  

# Follower  
Leader与Follower组成主从关系。  
一般情况下，Follower复制Leader的日志Log到本地即执行Leader的AppendEntries请求（当没有具体LogEntry时为心跳请求，用于维系主从关系）。  
当因为网络原因或其他原因收不到Leader的AppendEntries请求时，Follower切换为Candidate。

# Candidate
server在Candidate角色时发起Leader Election。  
在等待随机时长后更新任期 currentTerm += 1，将voteFor指向自己，向其他server广播RequestVote请求。当收到超过半数server的支持后，server转变为Leader角色。
等待随机时长的原因：
* 在最坏情况下，每个server同时发现丢失Leader角色，发起新一轮Leader Election，出现多Leader的情况。引入随机等待时长，将发起选举时间分散开，第一个满足条件的server成为Leader。

# Leader
server在Leader角色时负责处理Client的请求，发起并控制Follower角色server的日志Log，接收client的更新请求进行本地处理后再同步到其他副本。  

# Leader Election  
1. Follower角色的currentTerm += 1, voteFor指向自己，重置选举计时器
2. 广播投票请求，根据结果处理：  
    * 如果获得半数以上赞成票则切换为Leader角色。如果接收RPC请求的server发现currentTerm落后于请求中的term，则切换为Follower角色；如果接收RPC请求的server发现currentTerm比请求中的term更新，说明RPC请求过时，无视~
    * 如果Candidate角色server在等待投票时，收到了>=currentTerm且不同LeaderId的AppendEntriesRPC时，将自己切换为Follower角色，并更新currentTerm
    * 没有选出主。Split vote时,选举计时器超时，更新currentTerm，重新发起一轮投票进行Leader Election
    
# Log Replication
Leader角色server将client发来的请求作为Log Entry追加到本地logs，向其他server广播AppendEntries请求。  
当Leader角色server确定一个Log Entry被safely replicated时（majority将该命令写入本地log），就apply这条Log Entry到状态机中，然后返回结果给客户端。  
如果某个Follower宕机或运行很慢或网络故障，则会持续发送AppendEntries直到该Follower的日志与Leader一致。  
当一条日志committed了，Leader才将它应用到状态机中。Raft保证了该日志已经持久化且会被所有的节点执行。
