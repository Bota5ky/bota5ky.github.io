### 1. 基础命令

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install <release_name> <repo_name>/apache --namespace=web
helm upgrade <release_name> <repo_name>/apache --namespace=web
helm rollback apache 1 --namespace=web
helm uninstall <release_name>
helm list #缩写helm ls
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
helm install mydb bitnami/mysql --set auth.rootPassword=test1234
helm install mydb bitnami/mysql --values values.yaml
helm install mydb bitnami/mysql --reuse-values
#验证密码
echo Password: $(kubectl get secret --namespace default mydb-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)
```

`values.yaml`

```yaml
auth:
  rootPassword: "test1234"
```

每一次升级都会在 k8s 创建一个 secret，uninstall 时也会把这些 secrets 删除。若想要保留

```bash
helm uninstall mydb --keep-history
```

