Kubernetes没有提供功能全面的内置监控解决方案，但有许多开源解决方案可用，如Metrics-Server、Prometheus、Elastic Stack、DATADOG、dynatrace。

Heapster是Kubernetes启用监控和分析功能的原始项目之一，但现已弃用，并形成了一个精简版本，称为Metrics Server（In-Memory）。

Kubelet包含一个称为cAdvisor或container advisor的子组件，负责从pod中检索性能指标，发送给apiserver。



### 使用Metrics Server

- 安装

minikube : `minikube addons enable metrics-server`

others : `git clone https://github.com/kubernetes-incubator/metrics-server.git`

- 部署：pods、services、roles

`kubectl create -f deploy/1.8+/` : 在下载后repo中执行`-f .`

- 查看：`kubectl top (node | pod)`



### Logs

`docker run kodekloud/event-simulator`，event-simulator用来生成随机事件模拟web服务器，它本身有std ouput，但如果在detach模式下运行`run -d`，那么想查看日志可以使用`docker logs -f container-id`，-f选项帮助查看实时日志跟踪。

在Kubernetes中也是一样：

`kubectl create -f event-simulator.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
```

`kubectl logs -f pod-name container-name`：如果pod中包含多个container，则必须要指明container-name，否则会报错，或用`-c`指定container。

当Pod有多个容器时打开shell：`kubectl -n elastic-stack exec -it app -- cat /log/app.log`或`kubectl exec -i -t my-pod --container main-app -- /bin/bash`，`-i`and`-t`are the same as the long options `--stdin` and `--tty`

### Rolling Updates and Rollbacks

`kubectl rollout status deployment/myapp-deployment`：查看状态

`kubectl rollout history deployment/myapp-deployment`：查看历史

部署策略：

- Recreate：一次销毁所有旧的，新建新版本
- Rolling Update：滚动更新，一次更新一个

更新image的方式：

- `kubectl apply -f deployment-definition.yml`
- `kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1`：会导致部署定义文件具有不同的配置

回滚：`kubectl rollout undo deployment/myapp-deployment`，回滚会在原来的replicasets上重新创建

### 命令和参数 Commands and Arguments

`pod-definition.yml` 实际执行为 `command` 后面跟着 `args`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]  ##覆写dockerfile中的ENTRYPOINT
      args: ["10"]  ##覆写dockerfile中的CMD
```

也可以写成

```yaml
      command:
        - "sleep"
        - "10"
```

### Kubernetes 中的环境变量

`docker run -e APP_COLOR=pink simple-webapp-color`

`pod-definition.yaml`

```yaml
spec:
  containers:
    env:
      - name: APP_COLOR
        value: pink
```

```yaml
        valueFrom:  ## 2. 单个环境变量导入
          configMapKeyRef:  ##ConfigMap
            name:
            key:
          
          secretKeyRef:  ##Secrets
```

### ConfigMaps

```yaml
spec:
  containers:
    envFrom:  ## 1. 多个环境变量导入
    - configMapRef:
        name: app-color
```

```yaml
volumes:  ##3. 作为文件导入volume
- name: app-config-volume
  configMap:
    name: app-config
```

创建：

- `kubectl create configmap <config-name> --from-literal=<key>=<value> --from-literal=...` 或使用文件创建 `--from-file=app_config.properties`

- `kubectl create -f`

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    APP_COLOR: blue
    APP_MODE: prod
  ```

  