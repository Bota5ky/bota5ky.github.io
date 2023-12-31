### 1. Upgrades

默认 pod 超时时间为5分钟：`kube-controller-manager --pod-eviction-timeout=5m0s`

系统升级时更安全的方式是：`kubectl drain node-1`，节点会被标记为不可调度，会清除 pod，但当 pod 不属于 replicaSet 时，drain 会失败，需要加上 `--force`，这个 pod 就会丢失，不会再在其他node 上被创建

升级完成之后：`kubectl uncordon node-1`

`kubectl cordon node-2`不会清除pod

使用 `kubectl get nodes` 可以在 VERSION 列查看版本

`v1.11.3`

- MAJOR

- MINOR : Features, Functionalities

- PATCH : Bug Fixes

ETCD、CoreDNS 的版本独立，kube-apiserver 的版本一般比其他的组件更高，kubectl 的版本可能会高或者低1个版本，其他 Controller-manager、kube-scheduler、kubelet、kube-proxy 版本保持一致。

Kubernetes 只支持最近的3个版本，当使用的 k8s 版本不再支持时，建议一次只升级一个版本。

使用 kubeadm 升级：

- `kubeadm upgrade plan` 查看最近稳定可用的版本

- `kubeadm upgrade apply`

先升级 master node，再升级 work node


### 2. Releases

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

- Ubuntu 升级 master node 流程：升级 node 用 `upgrade apply 版本`

On the controlplane node, run the command run the following commands: 

`apt update` : This will update the package lists from the software repository.

`apt install kubeadm=1.20.0-00` : This will install the kubeadm version 1.20

`kubeadm upgrade apply v1.20.0` : This will upgrade kubernetes controlplane. Note that this can take a few minutes.

`apt install kubelet=1.20.0-00` : This will update the kubelet with the version 1.20.

You may need to restart kubelet after it has been upgraded. Run: `systemctl daemon-reload`, `systemctl restart kubelet`

- Ubuntu 升级 work node 流程：升级 node 用 `upgrade node`

If you are on the master node, run `ssh node01` to go to node01

`apt update` : This will update the package lists from the software repository.

`apt install kubeadm=1.20.0-00` : This will install the kubeadm version 1.20

`kubeadm upgrade node` : This will upgrade the node01 configuration. 不用写node-name

`apt install kubelet=1.20.0-00` : This will update the kubelet with the version 1.20.

You may need to restart kubelet after it has been upgraded. Run: `systemctl daemon-reload`, `systemctl restart kubelet`

Type `exit` or enter `CTL + d` to go back to the controlplane node.

uncordon work node 需要切换到 master node

官方文档：https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

### 3. Rolling Updates and Rollbacks

`kubectl rollout status deployment/myapp-deployment`：查看状态

`kubectl rollout history deployment/myapp-deployment`：查看历史

部署策略：

- Recreate：一次销毁所有旧的，新建新版本
- Rolling Update：滚动更新，一次更新一个

更新 image 的方式：

- `kubectl apply -f deployment-definition.yml`
- `kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1`：会导致部署定义文件具有不同的配置

回滚：`kubectl rollout undo deployment/myapp-deployment`，回滚会在原来的replicasets上重新创建