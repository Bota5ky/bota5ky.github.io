### 1. 监控

Kubernetes 没有提供功能全面的内置监控解决方案，但有许多开源解决方案可用，如 Metrics-Server、Prometheus、Elastic Stack、DATADOG、dynatrace。

Heapster 是 Kubernetes 启用监控和分析功能的原始项目之一，但现已弃用，并形成了一个精简版本，称为 Metrics Server（In-Memory）。

Kubelet 包含一个称为 cAdvisor 或 container advisor 的子组件，负责从 pod 中检索性能指标，发送给 apiserver。

### 2. 使用 Metrics Server

- 安装

minikube : `minikube addons enable metrics-server`

others : `git clone https://github.com/kubernetes-incubator/metrics-server.git`

- 部署：pods、services、roles

`kubectl create -f deploy/1.8+/` : 在下载后 repo 中执行`-f .`

- 查看：`kubectl top (node | pod)`

### 3. Logs

`docker run kodekloud/event-simulator`，event-simulator 用来生成随机事件模拟 web 服务器，它本身有 std ouput，但如果在 detach 模式下运行`run -d`，那么想查看日志可以使用`docker logs -f container-id`，-f选项帮助查看实时日志跟踪。

在 Kubernetes 中也是一样：

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

`kubectl logs -f pod-name container-name`：如果 pod 中包含多个 container，则必须要指明 container-name，否则会报错，或用`-c`指定 container。

当 Pod 有多个容器时打开 shell：`kubectl -n elastic-stack exec -it app -- cat /log/app.log`或`kubectl exec -i -t my-pod --container main-app -- /bin/bash`，`-i`and`-t`are the same as the long options `--stdin` and `--tty`

### 5. 命令和参数（Commands and Arguments）

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

### 6. Kubernetes 中的环境变量

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

### 7. ConfigMaps

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

  