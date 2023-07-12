### KubeConfig

使用访问API获取pods信息：

```bash
curl https://my-kube-playground:6443/api/v1/pods \
 --key admin.key
 --cert admin.crt
 --cacert ca.crt
```

用kubectl达成相同的目的：

```bash
kubectl get pods
	--server my-kube-playground:6443
	--client-key admin.key
	--client-certificate admin.crt
	--certificate-authority ca.crt
```

使用kubeconfig代替每次冗长的输入：

```bash
kubectl get pods --kubeconfig config
```

`$HOME/.kube/config`

```yaml
apiVersion: v1
kind: Config

current-context: dev-user@google  ## 指定默认的上下文

clusters:
- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    ## 也可以直接使用证书内容代替 certificate-authority-data: 以cat ca.crt | base64格式写入
    server: https://172.17.0.51:6443

contexts:
- name: admin@production
  context:
    cluster: production
    user: admin
    namespace: finance

users:
- name: admin
  user:
    client-certificate: /etc/kubernetes/pki/users/admin.crt
    client-key: /etc/kubernetes/pki/users/admin.key
```

使用 `kubectl config view` 查看当前kubeconfig

可以 `kubectl config view --kubeconfig=my-custom-config` 指定kubeconfig

使用 `kubectl config use-context prod-user@prodection --kubeconfig /root/my-kube-config` 指定kubeconfig，更换上下文

设置默认config：`mv .kube/config .kube/config.bak` 备份，然后复制 `cp /root/my-kube-config .kube/config`

### RBAC

集群层面请使用 `ClusterRole` 和 `ClusterRoleBindings`

`developer-role.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]  ## 可以用 kubectl api-resources 查看 apiversion
  resources: ["pods"]                         
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```

创建角色：`kubectl create -f developer-role.yaml`

`devuser-developer-binding.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

创建角色绑定：`kubectl create -f devuser-developer-binding.yaml`

查询：

```bash
kubectl get roles
kubectl get rolebindings
kubectl describe role developer
kubectl describe rolebinding devuser-developer-binding
```

验证权限：

```bash
kubectl auth can-i create deployments
kubectl auth can-i delete nodes --as dev-user --namespace test
```

使用命令行创建、绑定、编辑角色：

```bash
kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods
kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user
kubectl edit role developer -n blue
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: blue
rules:
- apiGroups:
  - apps
  resourceNames:
  - dark-blue-app
  resources:
  - pods
  verbs:
  - get
  - watch
  - create
  - delete
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - get
  - watch
  - create
  - delete
```

### Service Accounts

`kubectl create serviceaccount dashboard-sa`

`kubectl get serviceaccount`

`kubectl describe serviceaccount dashboard-sa`

当创建Service Account时，首先创建 Service Account 对象，然后为服务账户生成令牌

查看secret token：`kubectl describe secret dashboard-sa-token-kbbdm`

利用 Service Account 创建 token：`kubectl create token sa-name`

默认采用 `default` 的 Service Account，可以在deployment指明 Service Account，在 pod spec 下添加 `serviceAccountName: sa-name`

使用curl访问：

```bash
curl https://192.168.56.70:6443/api -insecure --header "Authorization: Bearer <dashboard-sa-token-kbbdm>"
```

每个namespace都会创建 default 的 Service Account：`kubectl exec -it my-kubernetes-dashboard ls /var/run/secrets/kubernetes.io/serviceaccount`，默认只有运行基本的Kubernetes API 查询的权限

无法编辑pod中的Service Account，只能删除后新建，在deployment中可以。

如果不想自动挂载Service Account，可以设置：

```yaml
spec:
  automountServiceAccountToken: false
```

### Image Security

`image: docker.io/library/nginx`：不指定用户名或账户名，则会假定为library。Registry，User/Account，Image/Repository

`docker login private-registry.io`

`docker run private-registry.io/apps/internal-app`

```bash
kubectl create secret docker-registry regcred	\ ## 取名为regcred
	--docker-server= private-registry.io	\
	--docker-username= registry-user	\ ## 类型为docker-registry
	--docker-password= registry-password	\
	--docker-email= registry-user@org.com
```

deployment 应用 secret 需要添加以下设置：
```yaml
spec:
  imagePullSecrets:
  - name: regcred
```

查看哪个user在运行pod：`kubectl exec pod-name -- whoami`

修改 `securityContext`
```yaml
spec:
  securityContext:
    runAsUser: 1010  ## 设置在pod或container level
```

```yaml
spec:
  securityContext:
    capabilities:  ## 设置在container level
       add: ["SYS_TIME"]
```