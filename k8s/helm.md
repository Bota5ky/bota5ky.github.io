### 1. 基础命令

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install <release_name> <repo_name>/apache --namespace=web
helm upgrade <release_name> <repo_name>/apache --namespace=web
helm rollback apache 1 --namespace=web
helm uninstall <release_name>
```

work with chart repository:

```bash
helm repo list
helm search repo apache --versions
helm repo remove bitnami
```



