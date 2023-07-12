[词汇表](https://kubernetes.io/zh-cn/docs/reference/glossary/?fundamental=true)
[考试报名链接CN](https://training.linuxfoundation.cn/certificates/1)
[考试报名链接EN](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/)

Master：manage, plan, schedule, monitor nodes（主节点上也可以安装容器引擎）

worker nodes：host application containers

etcd cluster：存储所有集群数据的数据库

kube-scheduler：调度程序，确定要放置容器的正确节点

controller-manager：

- node-controller：加载新节点，替换不可用节点

- replication-controller：保证一定数量的容器运行

kube-api：协调集群中的所有操作（api是公开的），定期从kubelet获取状态报告

kubelet：集群中在每个节点上运行的代理

kube-proxy：负责节点间通信