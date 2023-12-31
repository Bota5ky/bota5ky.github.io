### 1. 创建

- `kubectl create secret generic <secret-name> --from-literal=<key>=<value>` 或 `--from-file=<path-to-file>`

- `kubectl create -f secret-data.yaml`

  `secret-data.yaml`

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
  name: app-secret
  data:
  DB_Host: mysql
  DB_User: root
  DB_Password: paswrd
  ```

### 2. 转换编码

- 编码：`echo -n 'mysql' | base64`
- 解码：`echo -n 'bXlzcWw=' | base64 --decode`

### 3. 查看

`kubectl get secret app-secret -o yaml`

### 4. 添加到pod

`pod-definition.yaml` : secrets 可以作为数据卷挂载或公开为`环境变量`由 Pod 中的容器使用

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    envFrom:
    - secretRef:
        name: app-config
```

```yaml
    env:
      - name: DB_Password
        valueFrom:
          secretKeyRef:
            name: app-secret
            key: DB_Password
```

```yaml
volumes:
- name: app-secret-volume
  secret:
    secretName: app-secret
```

如果以文件形式创建secret，则对每个secret会生成对应的文件：`ls /opt/app-secret-volumes`，`cat /opt/app-secret-volumes/DB_Password`

最佳实践：

- 未将secret对象定义文件签入源代码存储库
- 为Secret启用[静态加密](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)，以便将它们加密存储在ETCD中

Kubernetes处理secret的方式：

- 仅当节点上的pod需要时，才会将secret发送到该节点
- Kubelet将secret存储到tmpfs中，这样secret就不会写入磁盘存储
- 一旦依赖于secret的Pod被删除，kubelet也会删除其本地的secret数据副本