### 1. 绑定 Pod

未命名隐藏字段`nodeName`的 pod 将作为候选进行调度

```yaml
spec:
  nodeName: node02
```

但对于已创建的 pod，`pod-definition.yaml`中的`nodeName`字段不允许修改，因此将Node分配给 pod 的另一种方法是创建一个绑定对象，并向 pod 绑定 API 发送 post 请求：

`Pod-bind-definition.yaml`

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

把 yaml 文件转换成 Json 格式

```bash
curl --header "Content-Type:application/json" --request POST --data '{"apiVersion":"v1", "kind": "Binding", ...}' http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding
```

### 2. Labels

`kubectl get all`

`kubectl get pods -l env=dev | wc -l` : 列出env=dev的pod个数（包含header），`-l` short for `--selector`

`kubectl get pods -l env=dev --no-headers | wc -l` : 列出env=dev的pod个数（不包含header）

`kubectl label node node01 color=blue` : 添加Label

### 3. Taints - Node

`kubectl taint nodes node-name key=value:taint-effect`

taint-effect : NoSchedule | PreferNoSchedule | NoExecute

NoExecute : 新Pod不会部署，已存在的节点也会终止

### 4. Tolerations - Pods

`pod-definition.yml`

```yaml
spec:
  tolerations:
  - key: app
    operator: Equal
    value: blue
    effect: NoSchedule
```

`kubectl describe node kubemaster | grep Taint` : 查看master node的Taint

`kubectl taint node node-name key=value:NoSchedule-` : 清除taint

### 5. Node Selectors

`pod-definition.yml`

```yaml
spec:
  nodeSelector:
    size: Large  ##生效前需要先标记 node
```

`kubectl label nodes <node-name> <label-key>=<label-value>` : 标记node

### 6. Node Affinity

`pod-definition.yml`

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: NotIn | In | Exists ## Exists运算符甚至不需要下面的values
            values:
            - Large
            - Medium
```

Available ：

- requiredDuringSchedulingIgnoredDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution

Planned :

- requiredDuringSchedulingrequiredDuringExecution

DuringScheduling：Pod 不存在且是首次创建

|       | DuringScheduling | DuringExecution |
| :---: | :--------------: | :-------------: |
| Type1 |     Required     |     Ignored     |
| Type2 |    Preferred     |     Ignored     |
| Type3 |     Required     |    Required     |

### 7. Multiple Schedulers

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