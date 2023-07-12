- Scheduling

未命名隐藏字段`nodeName`的pod将作为候选进行调度

```yaml
spec:
  nodeName: node02
```

但对于已创建的pod，`pod-definition.yaml`中的`nodeName`字段不允许修改，因此将Node分配给pod的另一种方法是创建一个绑定对象，并向pod绑定API发送post请求：

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

把yaml文件转换成Json格式

```bash
curl --header "Content-Type:application/json" --request POST --data '{"apiVersion":"v1", "kind": "Binding", ...}' http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding
```

- Labels and Selectors

`kubectl get all`

`kubectl get pods -l env=dev | wc -l` : 列出env=dev的pod个数（包含header），`-l` short for `--selector`

`kubectl get pods -l env=dev --no-headers | wc -l` : 列出env=dev的pod个数（不包含header）

`kubectl label node node01 color=blue` : 添加Label

- Taints - Node :

`kubectl taint nodes node-name key=value:taint-effect`

taint-effect : NoSchedule | PreferNoSchedule | NoExecute

NoExecute : 新Pod不会部署，已存在的节点也会终止



- Tolerations - Pods :

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

### Node Selectors & Affinity

- Node Selectors

`pod-definition.yml`

```yaml
spec:
  nodeSelector:
    size: Large  ##生效前需要先标记 node
```

`kubectl label nodes <node-name> <label-key>=<label-value>` : 标记node

- Node Affinity

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

DuringScheduling : Pod不存在且是首次创建

|       | DuringScheduling | DuringExecution |
| :---: | :--------------: | :-------------: |
| Type1 |     Required     |     Ignored     |
| Type2 |    Preferred     |     Ignored     |
| Type3 |     Required     |    Required     |