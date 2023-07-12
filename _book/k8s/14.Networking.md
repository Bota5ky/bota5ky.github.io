### Docker Networking

<br/><img src="https://img2022.cnblogs.com/blog/2122768/202210/2122768-20221009091908981-214431760.png" style="zoom:45%"/>

当您运行容器时，您有不同的网络选项可供选择：

- container无法到达外部，外部也无法访问container `docker run --network none nginx`

- 容器链接到host，host和容器之间没有网络隔离无需接口转发，但两个进程无法同时侦听同一端口 `docker run --network host nginx`

- 第三种网络选项是bridge，在这种情况下，会创建一个内部专用网络，Docker主机和容器将连接到该网络。

Docker在内部使用了一种类似于我们在network ns中讲的技术，即运行类型设置为bridge的IP link add 命令，`ip link add docker0 type bridge`。

当Docker安装在主机上时，默认情况下，会创建一个称为 `bridge` 的内部专用网络，可由 `docker network ls` 查看。但在host上由 `ip link` 查看，显示为 `docker0`。

根据`ip link` 查看 `docker0`接口状态为 `DOWN`，接口或网络当前已关闭。

还记得我们说过，bridge网络就像是host的接口，但它也是host内部ns或者container的交换机。根据`ip addr`可以查看`docker0`被分配的IP地址。

每当创建container时，Docker都会为其创建一个ns，运行 `ip netns`命令可以列出ns（需要设置）。

`docker inspect ns-name`可以看到与每个容器关联的ns。

docker或者说 ns 连接到 bridge 上的方式和之前说的一样。如果在docker host上运行 `ip link`，我们会看到接口的一端连接到本地 bridge `master docker0`

用 `ip -n b3165c10a92b link` 连接到container上，可以看到对应的接口，用 `ip -n b3165c10a92b addr` 查看对应IP地址

接口通常成对匹配，container偶数，bridge接口为奇数

将docker上的8080端口映射到container的80上：`docker run -p 8080:80 nginx`，实现原理也一样 `iptables -t nat -A DOCKER --dport 80 --to-destination 172.17.0.3:80 -j DNAT`，查看的命令 `iptables -nvL -t nat`

### CNI - Container Networking Interface

VETH : virtual Ethernet devices

| Network Namespaces                         | docker                                     |
| :----------------------------------------- | ------------------------------------------ |
| 1. Create Network Namespace                | 1. Create Network Namespace                |
| 2. Create Bridge Network/Interface         | 2. Create Bridge Network/Interface         |
| 3. Create VETH Pairs (Pipe, Virtual Cable) | 3. Create VETH Pairs (Pipe, Virtual Cable) |
| 4. Attach VETH to Namespace                | 4. Attach VETH to Namespace                |
| 5. Attach Other VETH to Bridge             | 5. Attach Other VETH to Bridge             |
| 6. Assign IP Address                       | 6. Assign IP Address                       |
| 7. Bring the interfaces up                 | 7. Bring the interfaces up                 |
| 8. Enable NAT - IP Masquerade              | 8. Enable NAT - IP Masquerade              |

统一的步骤2~8步，可以用bridge命令运行，指定将容器添加到ns：

```bash
bridge add 2e34dcf34 /var/run/netns/2e34dcf34
bridge add <container id> <namespace>
```

CNI (Container Networking Interface) :
- container runtime
  - 容器运行时必须创建网络命名空间
  - 标识容器必须连接到的网络
  - 添加容器时调用网络插件（网桥）的容器运行时
  - 删除容器时调用网络插件（网桥）的容器运行时
  - 网络配置的JSON格式
- 插件方面
  - 必须支持命令行参数ADD/DEL/CHECK
  - 必须支持参数container id、network ns等
  - 必须管理POD的IP地址分配
  - 必须以特定格式返回结果

CNI已经附带了一组支持的插件，如bridge、vlan、ipvlan、macvlan、windows，以及IPAM插件，如DHCP、host-local，还有一些其他第三方组织提供的插件。

但Docker有一套自己的标准，称为CNM（Container Network Model），不适配CNI，所以无法运行 `docker run --network=cni-bridge nginx`，但这并不意味着Docker无法应用CNI，你可以创建一个没有任何网络配置的Docker容器 `docker run --network=none nginx`，然后手动调用bridge插件，k8s就是这么做的 `bridge add 2e34dcf34 /var/run/netns/2e34dcf34`

https://kubernetes.io/docs/setup/independent/install-kubeadm/#check-required-ports

查看所有支持的CNI插件：`/opt/cni/bin`

查看当前使用的CNI插件：`ls /etc/cni/net.d/`

查看kubelet的container runtime：`ps -aux | grep kubelet | grep --color container-runtime`

### 常用命令

`ifconfig -a`：显示所有接口，包括环回接口、集群使用的实际物理接口等

`cat /etc/network/interfaces`：显示所有物理接口，环回接口、物理接口

`ip link`：显示此系统上的所有物理链路，`ip link show eth0` = `ifconfig eth0`

`ip route show default`：查看默认网关，或者 `ip r`

`netstat -natulp | grep scheduler`：-p programs，-l listening，-t tcp

### Cluster Networking

<br/><img src="https://img2022.cnblogs.com/blog/2122768/202210/2122768-20221009104211752-1308136240.png" style="zoom:38%"/>

kubernetes集群由master node和worker node组成。每个节点必须至少有一个连接到网络的接口，每个接口必须配置一个IP地址，host必须设置唯一的主机名以及唯一的mac地址。如果通过现在VM克隆来创建VM，则应特别注意这一点。还有一些端口也需要打开，这些由控制平面中的各种组件使用。

<br/><img src="https://img2022.cnblogs.com/blog/2122768/202210/2122768-20221009105538878-936889953.png" style="zoom:32%"/>

参考文档：https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports

因此，当您在防火墙中为节点设置网络时，或者在GCP、Azure或AWS等云环境中设置ip表规则或网络安全组时，请考虑这些问题。

### Pod Networking

到目前为止，k8s还没有为此提供内置解决方案，它希望您实施一个网络解决方案来解决这些难题。

但是k8s已经明确列出了对pod networking的要求：

- 每个POD都应该有一个IP地址
- 每个POD都应该能够与同一节点中的其他POD通信
- 每个POD应该能够在没有NAT的情况下与其他节点上的每个其他POD通信

步骤：

- 在每个节点上创建一个bridge网络 `ip link add v-net-0 type bridge`
- 然后 `ip link set dev v-net-0 up`
- 设置bridge的IP地址 `ip addr add 10.244.1.0/24 dev v-net-0`
- 向默认网关添加IP地址 `ip route add 10.244.1.0/24 via 192.168.15.5`

但与其在每台服务器上配置路由，不如在路由器上配置路由。如果您的网络中有一个网关，并指定所有主机使用该网关作为默认网关，这样，您就可以轻松地管理路由器上路由表中所有网络的路由。

然后，我们编写了一个脚本，可以为每个容器运行该脚本，那么当我们在 k8s 上创建端口时，我们如何自动运行脚本呢？这就是 CNI 充当中间人的原因。CNI 告诉 k8s，这是您在创建容器后应该立即调用脚本的方式。

根据 CNI 标准，脚本应该有 `ADD` 和 `DEL` 部分。

执行步骤：

- `kubelet` 在运行时查看之前的配置 `--cni-conf-dir=/etc/cni/net.d` 并识别脚本名称
- 然后在目录 `--cni-bin-dir=/etc/cni/bin` 中寻找脚本
- 使用命令 `./net-script.sh add <container> <namespace>` 执行脚本

### CNI in kubernetes

`ps -aux | grep kubelet`：查看设置为CNI的网络插件和一些与CNI相关的其他选项，如CNI bin目录和CNI config目录

`ls /opt/cni/bin`：bin目录包含所有支持的CNI插件作为可执行文件，如bridge、DHCP、flannel等

`ls /etc/cni/net.d`：conf文件是kubelet查找需要使用哪个插件的地方

### CNI weave

weave CNI插件部署在集群上，会在每个节点上部署一个代理或服务

一个pod可以连接到多个bridge

安装：`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.50.0.0/16"`，可以指定IP范围以防止和host系统IP重叠

### Service Networking

Service承载在整个cluster上，整个cluster的pod都可以访问到这个节点，Service没有绑定到特定节点，但只能从集群内部访问该服务。

NodePort可以不仅让cluster内部的节点访问，它还会再cluster中所有节点的端口上公开application。

查看IPtable转发规则：`iptables -L -t nat | grep db-service`

或者查看日志：`cat /var/log/kube-proxy.log`，文件位置可能因安装而异

查看services的IP范围：`cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range`

查看pod的IP范围和使用的proxy类型：`kubectl logs <weave-pod-name> weave -n kube-system`

### DNS in kubernetes

| Hostname    | Namespace | Type | Root          | IP Address    |
| ----------- | --------- | ---- | ------------- | ------------- |
| web-service | apps      | svc  | cluster.local | 10.107.37.188 |
| 10-244-2-5  | apps      | pod  | cluster.local | 10.244.2.5    |

`curl http://10-244-2-5.apps.pod.cluster.local`

### CoreDNS in Kubernetes

建立DNS的方式：

- 设置每个pod上的 `/etc/hosts`
- 设置CoreDNS `/etc/resolv.conf`，pod名字为用短横线连接的IP

在v1.12版本之前k8s实施的DNS为kube-dns，之后为CoreDNS。

CoreDNS服务器作为POD部署在kubernetes集群的kube-system namespace中，它们被部署为两个pod以实现冗余，作为replicaSet的一部分。

查看CoreDNS的配置文件：`kubectl -n kube-system describe deployments.apps coredns | grep -A2 Args | grep Corefile`