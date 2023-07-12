### Upgrades

默认pod超时时间为5分钟：`kube-controller-manager --pod-eviction-timeout=5m0s`

系统升级时更安全的方式是：`kubectl drain node-1`，节点会被标记为不可调度，会清除pod，但当pod不属于replicaSet时，drain会失败，需要加上 `--force`，这个pod就会丢失，不会再在其他node上被创建

升级完成之后：`kubectl uncordon node-1`

`kubectl cordon node-2`不会清除pod

使用 `kubectl get nodes` 可以在 VERSION 列查看版本

`v1.11.3`

- MAJOR

- MINOR : Features, Functionalities

- PATCH : Bug Fixes

ETCD、CoreDNS的版本独立，kube-apiserver的版本一般比其他的组件更高，kubectl的版本可能会高或者低1个版本，其他Controller-manager、kube-scheduler、kubelet、kube-proxy版本保持一致。

Kubernetes只支持最近的3个版本，当使用的k8s版本不再支持时，建议一次只升级一个版本。

使用kubeadm升级：

- `kubeadm upgrade plan` 查看最近稳定可用的版本

- `kubeadm upgrade apply`

先升级master node，再升级work node


### Releases

- 升级master node：

`apt-get upgrade -y kubeadm=1.12.0-00`

`kubeadm upgrade apply v1.12.0`

`kubectl get nodes` 上显示的是apiserver版本

如果master node上有kubelet，则`apt-get upgrade -y kubelet=1.12.0-00`，然后`systemctl restart kubelet`

- 升级work node：

`kubectl drain node-1` 把节点上的pod转移到其他节点

`apt-get upgrade -y kubeadm=1.12.0-00`

`apt-get upgrade -y kubelet=1.12.0-00`

`kubeadm upgrade node config --kubelet-version v1.12.0`

`systemctl restart kubelet`

`kubectl uncordon node-1`

查看系统版本：`cat /etc/*release*`

- Ubuntu升级master node流程：升级node用 `upgrade apply 版本`

On the controlplane node, run the command run the following commands: 

`apt update` : This will update the package lists from the software repository.

`apt install kubeadm=1.20.0-00` : This will install the kubeadm version 1.20

`kubeadm upgrade apply v1.20.0` : This will upgrade kubernetes controlplane. Note that this can take a few minutes.

`apt install kubelet=1.20.0-00` : This will update the kubelet with the version 1.20.

You may need to restart kubelet after it has been upgraded. Run: `systemctl daemon-reload`, `systemctl restart kubelet`

- Ubuntu升级work node流程：升级node用 `upgrade node`

If you are on the master node, run `ssh node01` to go to node01

`apt update` : This will update the package lists from the software repository.

`apt install kubeadm=1.20.0-00` : This will install the kubeadm version 1.20

`kubeadm upgrade node` : This will upgrade the node01 configuration. 不用写node-name

`apt install kubelet=1.20.0-00` : This will update the kubelet with the version 1.20.

You may need to restart kubelet after it has been upgraded. Run: `systemctl daemon-reload`, `systemctl restart kubelet`

Type `exit` or enter `CTL + d` to go back to the controlplane node.

uncordon work node 需要切换到master node

官方文档：https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/