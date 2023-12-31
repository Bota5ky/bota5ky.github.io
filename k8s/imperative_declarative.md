### 1. 命令式和声明式

命令式 Imperative：

```bash
kubectl run nginx --image=nginx --port=80 --labels="tier=db,env=prod" --expose=true  ##container的port
kubectl create deployment --image=nginx nginx
kubectl expose deployment nginx --port 80 ## cluster的port
kubectl edit deployment nginx ##仅修改活动对象配置文件
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.18
kubectl create -f nginx.yaml
kubectl replace -f nginx.yaml ##更新本地配置文件后运行
kubectl replace --force -f nginx.yaml ##删除后再创建
kubectl delete -f nginx.yaml
```

声明式 Declarative：

```bash
kubectl apply -f nginx.yaml
kubectl apply -f /path ##根据路径创建多个对象
```

**创建 NGINX Pod**

```bash
kubectl run nginx --image=nginx
```

**生成 POD 清单 YAML 文件`-o yaml`，如果不想实际创建则用`--dry-run`**

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

**创建部署**

```bash
kubectl create deployment --image=nginx nginx
```

**生成部署 YAML 文件`-o yaml`，如果不想实际创建则用`--dry-run`**

```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

**使用 4 个副本生成部署**

```bash
kubectl create deployment nginx --image=nginx --replicas=4
```

您还可以使用该`kubectl scale`命令扩展部署。

```bash
kubectl scale deployment nginx --replicas=4
```

**另一种方法是将 YAML 定义保存到文件并修改**

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

然后，您可以在创建部署之前使用副本或任何其他字段更新 YAML 文件。

**创建一个名为 redis-service 的 ClusterIP 类型的 Service 以在端口 6379 上公开 pod redis**

```bash
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

（这将自动使用 pod 的标签作为选择器）

或者

```bash
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

（这不会将 pods 标签用作选择器，而是将选择器假定为**app=redis。**[您不能将选择器作为选项传递。](https://github.com/kubernetes/kubernetes/issues/46191)因此，如果您的 pod 具有不同的标签集，它就不能很好地工作。所以生成文件并在创建服务之前修改选择器）

**创建一个名为 nginx 的 NodePort 类型的 Service 以在节点上的 30080 端口上公开 pod nginx 的 80 端口：**

```bash
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```

（这将自动使用 pod 的标签作为选择器，[但您不能指定节点端口](https://github.com/kubernetes/kubernetes/issues/25478)。您必须生成定义文件，然后手动添加节点端口，然后再使用 pod 创建服务。）

或者

```bash
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

（这不会使用 pods 标签作为选择器）

上述两个命令都有自己的挑战。虽然其中一个不能接受选择器，但另一个不能接受节点端口。我建议使用`kubectl expose`命令。如果需要指定节点端口，请使用相同的命令生成定义文件，并在创建服务之前手动输入节点端口。

#### **参考：**

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

https://kubernetes.io/docs/reference/kubectl/conventions/

### 2. kubectl apply 原理

- 本地的yaml配置文件会转换成 json 格式的文件

- `kubectl apply` 会对本地配置文件、最后一次 apply 的配置文件（Json）和实时对象配置文件进行对比，当本地配置文件更新后也会同时更新其他2个配置文件

- 合并更改：https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/#merging-changes-to-primitive-fields

- Json 内容实际保存在实时对象配置文件中的

  ```yaml
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: {Json Content}
  ```

  实际请不要这样配置