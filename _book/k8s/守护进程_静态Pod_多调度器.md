### 1. Daemon Sets

Daemon Sets确保 pod 的一个副本始终存在于集群的所有节点中，常用于 Monitoring Solution、Logs Viewer、Kube-porxy、Weave-net（networking）。

`daemon-set-definition.yaml`

```yaml
apiVersion: apps/v1
kind: DaemonSet ##唯一区别
metadata:
  name: elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: k8s.gcr.io/fluentd-elasticsearch:1.20
```

在 v1.12 之前，pod 可以设置 nodeName 以放置到想要的 node 上，之后使用 scheduler 和 affinity。

因为没有`kubectl create daemonset`相关的命令，所以创建DaemonSets时可以先用create deployment 命令生成 yaml 模板，`kubectl create deployment ds-name -n=namespace-name --image=image-name --dry-run=client -o yaml > app.yaml`，修改后 apply。

### 2. Static Pods

kubelet 依赖于 kube-apiserver 来获得关于在其 node 上加载哪些 pod 的指令，这是基于存储在 etcd 数据库中的 kube-scheduler 所做的决定。

kubelet 也可以独立运行，可以创建 pod，可以指定用于存储 pod 信息的目录中读取 pod 定义文件。kubelet 会每隔一段时间确认 pod 定义文件的信息，并保持一致。

replicasets、deployment、service无法独立运行。它们都是整个 Kubernetes 架构的概念组成部分，需要复制和部署控制器等其他控制平面组件。

kubelet 在 pod 级别工作，只能理解 pod，这也是为什么它能够创建 static pod。

指定目录可以是任意地址，指定方式为`kubelet.service`文件中

- `--pod-manifest-path=/etc/Kubernetes/manifests`
- `--config=kubeconfig.yaml` ps : kubeadmin 也是用这种方式实现的

其中`kubeconfig.yaml`

```yaml
staticPodPath: /etc/Kubernetes/manifests
```

用`docker ps`查看 Static Pod 生成结果，如果没有 Kubernetes cluster。

如果有 Kubernetes cluster，kube-apiserver 会知道 Static Pod 的情况。（kube-apiserver 上会有个 Static Pod 的只读镜像，pod 的 name 会附加 node 的名称）

可以使用 Static Pod 将控制平面组件本身作为 pod 部署在 node 上，这样就可以在本地进行部署，不必下载二进制文件配置服务或担心服务崩溃，这也是 kubeadmin 工具设置 Kubernetes 集群的方式。

**Static Pods vs DaemonSets**

|                  Static Pods                   |                    DaemonSets                     |
| :--------------------------------------------: | :-----------------------------------------------: |
|             Created by the Kubelet             | Created by Kube-API server (DaemonSet Controller) |
| Deploy Control Plane components as Static Pods | Deploy Monitoring Agents, Logging Agents on nodes |
|         Ignored by the Kube-Scheduler          |           Ignored by the Kube-Scheduler           |



判断是 Static Pod 的几种方式：

- pod name 结尾带有 node name
- `kubectl get pod pod-name -n=kube-system -o yaml`中查看配置文件，ownerReferences 属性下 kind 为 Node，普通的为 ReplicaSet 等

查看 Static Pod 的配置文件位置：

- 查找config文件的方式 `ps -aux | grep kubelet` 查看 `--config` 项

- 查看`/var/lib/kubelet/config.yaml`中的staticPodPath



添加 command 的方式：在 kubectl 命令后加上`--command -- sleep 1000`，请保证`--command`放在整条命令之后，所有在`--`后的都会被视为添加的 command。

创建 Static Pod 的方式就是把 pod 定义文件放到 staticPath

切换 node 的方式`ssh node-ip-address`

### 3. Multiple Schedulers

`kube-scheduler.service`

```yaml
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --scheduler-name= default-scheduler  
```

`/etc/kubernetes/manifests/kube-scheduler.yaml  `

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - command:  ##包含用于启动调度程序的命令和相关选项
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true  ##没有多个主节点运行scheduler就设置为false
    - --scheduler-name=my-custom-scheduler  ##设置调度程序的自定义名称
    - --lock-object-name=my-custom-scheduler  ##用于区分自定义调度程序和默认调度程序
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler
```

`pod-definition.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  schedulerName: my-custom-scheduler
```

`kubectl get events`：查看哪个 scheduler 调用

`kubectl logs my-custom-scheduler --name-space=kube-system`：查看日志

`kubectl create configmap my-scheduler-config --from-file=/root/my-scheduler-config.yaml -n kube-system`：创建 configmap