官方文档：https://helm.sh/zh/docs

### 1. 基础命令

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install <release_name> <repo_name>/apache --namespace=web --create-namespace
helm upgrade <release_name> <repo_name>/apache --namespace=web
helm rollback apache 1 --namespace=web
helm uninstall <release_name> #卸载时没有--delete-namespace
helm list #缩写helm ls
helm status mydb
helm get notes mydb #获取release notes
helm get values mydb
helm get values mydb --all #获取所有包括默认值
helm get values mydb --revision 1 #获取特定版本的值
helm get manifest --revision 1
helm template mydb bitnami/mysql --values=values.yaml #不用连接到api-server
helm history mydb
helm rollback mywebserver 1 #回滚到版本1
helm upgrade --install mywebserver bitnami/apache
helm install bitnami/apache --generate-name
helm install bitnami/apache --generate-name --name-template "mywebserver-{{randAlpha 7 | lower}}" #随机7位小写字母，创建secret不能用大写字母，所以用大写会报错
helm install mywebserver bitnami/apache --wait --timeout 5m10s #等待部署完成，默认5分钟
helm install mywebserver bitnami/apache --atomic #失败自动回滚，默认开启--wait
helm upgrade mywebserver bitnami/apache --force #强制restart，就算资源配置无修改，会导致downtime
helm upgrade mywebserver bitnami/apache --cleanup-on-failure #升级失败则清除相关资源 
```

work with chart repository:

```bash
helm repo list
helm search repo apache --versions
helm repo remove bitnami
helm repo update #升级Charts
```

### 2. 自定义值

```bash
helm install mydb bitnami/mysql --set auth.rootPassword=test1234 #权限最高，会覆盖values.yaml
helm install mydb bitnami/mysql --values values.yaml --dry-run #不会在k8s中创建
helm install mydb bitnami/mysql --reuse-values
#验证密码
echo Password: $(kubectl get secret --namespace default mydb-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)
```

`values.yaml`

```yaml
auth:
  rootPassword: "test1234"
```

每一次升级都会在 k8s 创建一个 secret，uninstall 时也会把这些 secrets 删除。若想要保留：

```bash
helm uninstall mydb --keep-history
```

### 3. Chart

```bash
helm create firstchart
```

chart 文件目录结构：

```
firstchart/
  Chart.yaml          # 包含了chart信息的YAML文件
  values.yaml         # chart 默认的配置值
  charts/             # 包含chart依赖的其他chart
  templates/          # 模板目录， 当和values 结合时，可生成有效的Kubernetes manifest文件
    NOTES.txt # 可选: 包含简要使用说明的纯文本文件
    _helpers.tpl
    deployment.yaml
    hpa.yaml
    ingress.yaml
    service.yaml
    serviceaccount.yaml
    tests/

  LICENSE             # 可选: 包含chart许可证的纯文本文件
  README.md           # 可选: 可读的README文件
  values.schema.json  # 可选: 一个使用JSON结构的values.yaml文件
  crds/               # 自定义资源的定义
```

使用自定义 chart install

```bash
helm install firstapp firstchart
```

默认情况下，service 会暴露 pod 的 clusterIP，只能在 cluster 内部访问。

暴露容器端口 80 到本地 8080：

```bash
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=firstchart,app.kubernetes.io/instance=firstapp" -o jsonpath="{.items[0].metadata.name}")

export CONTAINER_PORT=$(kubectl get pods --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")

kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT

Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

### 4. Templates
