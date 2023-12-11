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
helm lint firstchart #检查语法和缩进错误
```

work with chart repository:

```bash
helm repo list
helm search repo apache --versions
helm repo remove bitnami
helm repo update #升级Charts
helm package firstchart/ --dependencies-update #打包chart -u会把依赖chart放到charts/目录下
helm package firstchart/ --destination #-d打包到指定路径
```

### 2. 自定义值

```bash
helm install mydb bitnami/mysql --set auth.rootPassword=test1234 #权限最高，会覆盖values.yaml
helm install mydb bitnami/mysql --values values.yaml --dry-run #不会在k8s中创建
#--dry-run相比template命令，不仅会做语法检查，还会检查配置schema
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

添加依赖：

```yaml
#Chart.yaml
dependencies:
  - name: mysql
    version: "8.8.6"
    repository: "http://charts.bitnami.com/bitnami"
    condition: mysql.enable #根据values提供的条件引入依赖
```

版本范围控制：=、!=、>、<、>=、<=、and、|、^、~

```yaml
version: "= 8.8.6 and < 9.0.0"
```

`^`表示大于等于当前版本，但不会更新到下一个主版本：如`^1.3.x`表示`>= 1.3.0 < 2.0.0`，`^0.2.4`表示`>= 0.2.4 < 0.3.0`，`^0`表示`>= 0.0.0 < 1.0.0`

`~`表示不会更新到下一个小版本，`~ 1.3.4`表示`>= 1.3.4 < 1.4.0`，`~ 2`表示`>= 2 < 3`，`~ 2.3`表示`>= 2.3 < 2.4`

添加依赖 Chart 到`charts/`文件夹

```bash
helm dependency update firstchart
```

更新依赖，会在目录下生成`Chart.lock`文件，里面包含本次更新的具体版本

```bash
helm update dependency
```

Chart repository 也可以使用 @ 符号标明，但还是建议直接写明 repo url，直接用 repo 名称需要使用 helm 在本地添加：

```yaml
repository: "@bitnami"
```

也可以使用`tags`来控制是否引入依赖

```yaml
#Chart.yaml
dependencies:
  - name: mysql
    version: "8.8.6"
    repository: "http://charts.bitnami.com/bitnami"
    tags:
      - enabled

#values.yaml
tags:
  enabled: false
```

给依赖传递值，覆写默认值

```yaml
#values.yaml
mysql:
  auth:
    rootPassword: test1234
  primary:
    service:
      type: NodePort
      nodePort: 30788
```

从子 Charts 显式读取值：

```yaml
dependencies:
  - name: mysql
    version: "8.8.6"
    repository: "http://charts.bitnami.com/bitnami"
    tags:
      - enabled
    import-values:
      - service

# child chart.yaml
export:
  service:
    port: 8080
    
{{.Values.servce.type}} #使用方式
```

从子 Charts 隐式读取值：

```yaml
dependencies:
  - name: mysql
    version: "8.8.6"
    repository: "http://charts.bitnami.com/bitnami"
    tags:
      - enabled
    import-values:
      - child: primary.service #从子Chart的primary.service下读取值
        parent: mysqlService #赋值到父Chart的values.yaml的mysqlService节点下
```

### 4. Template Actions

使用 `helm template chartname`预览 yaml 文件渲染效果。

```yaml
  {{"Helm Templating is"  -}}, {{- "Cool"}} //-会删除多余的前导后导空格，输出以下
  Helm Templating is,Cool
```

引用 Chart 内的值需要首字母大写。

```yaml
{{.Chart.AppVersion}}
{{.Release.IsUpgrade}}
{{.Template.Name}}
{{.Values.my.custom.data | default "testdefault" | upper | quote}} #设置默认值，加双引号
```

nindent：nested indent 嵌套缩进

条件语句：

```yaml
{{- if not .Values.my.flag }}
{{- end}}

{{- if and .Values.my.flag .Values.my.anotherflag }}
{{- if or }}
{{- else if -}}
{{- else -}}
{{- end}}
```

with 语句：替换`$.`代表的根元素（通常省略 $），常用于列表值

```yaml
{{- with .Values.my.values}}
{{- toYaml . | nindent 2}}
{{- else}} #如果元素为空
{{- end}}
```

define 语句：

```yaml
{{ $myFLAG := true }} #后续再赋值其他类型的值无效
```

循环语句：同样会替换`$.`代表的根元素，可以遍历字典类型的数据

```yaml
{{- range .Values.my.values}}

{{- range $key,$value := .Values.image}}
  - {{$key}}: {{$value}}
{{- end}}
```

_helpers.tpl

```yaml
{{- define "firstchart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-"}}
{{- end}}
```

可以防止同名函数冲突

引用模版内容

```yaml
{{ template "firstchart.name" . }} #无法使用管道
{{ include "firstchart.name" . }}
```

### 5. Hook

hook 类型：pre-install、post-install、pre-delete、post-delete、pre-upgrade、post-upgrade、pre-rollback、post-rollback、test

hook 权重：默认为 0

hook 删除策略：before-hook-creation（默认）、hook-succeeded、hook-failed

```yaml
metadata:
  annotations: 
    "helm.sh/hook": pre-install,post-install
    "helm.sh/hook-weight": "1"
    "helm.sh/hook-delete-policy": hook-failed
```

### 6. Testing

```yaml
metadata:
  annotations: 
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      imgae: busybox
      command: ['wget']
      args: ['{{ include "firstchart.fullname" . }}: {{ .Values.service.port }}']
  restartPolicy: Never
```

只会在执行`helm test`命令时运行，前提需要 release installed，命令返回非 0 则表示 test failed，配合 pipeline 做后续处理。

test 的删除策略默认不设置，因为如果测试失败可以查看失败原因。

### 7. Repository

创建 index.yaml

```bash
helm repo index chartsrepo
```

把创建的 chart 打包到 repository 目录下

```
helm package firstchart -d chartsrepo
helm repo index chartsrepo #更新index.yaml
helm repo index .
```

用 python 在本地启动一个 chart repo server

```bash
# 在chartsrepo目录下执行
python3 -m http.server --bind 127.0.0.1 8080
```

添加 repo 地址

```bash
helm repo add localrepo http://localhost:8080
```

拉取 chart

```bash
helm pull localrepo/firstchart
```

查找、更新

```bash
helm search repo firstchart
helm repo update
```

使用 github pages，用生成的页面作为 chart 发布的 url

helm 3.0 之后 OCI（open container initiative registries）功能，启用：

```bash
export HELM_EXPERIMENTAL_OCI=1
```

使用 OCI repo

```bash
docker run -d --name oci-registry -p 5000:5000 registry #-d detach mode
helm package firstchart
helm push firstchart-0.1.0.tgz oci://localhost:5000/helm-charts #helm-charts为repo name
helm show all oci://localhost:5000/helm-charts/firstchart --version 0.1.0
helm pull oci://localhost:5000/helm-charts/firstchart --version 0.1.0
helm template myrelease oci://localhost:5000/helm-charts/firstchart --version 0.1.0
helm install myrelease oci://localhost:5000/helm-charts/firstchart --version 0.1.0
helm upgrade myrelease oci://localhost:5000/helm-charts/firstchart --version 0.2.0 #新版本
helm registry login -u myuser <oci registry>
helm registry logout <oci registry url>
```

### 8. Security

用户使用公钥对`myChart.tgz.prov`文件进行签名校验

```bash
helm install --verify
```

使用 GUNPG 制作公钥、私钥

```bash
gpg --version
gpg --full-generate-key
gpg --export-secret-keys > ~/.gnupg/secring.gpg
helm package --sign --key bharath@helm.com --keyring ~/.gnupg/secring.gpg firstchart -d chartsrepo
#输入密码，生成
helm verify chartsrepo/firstchart-0.1.0.tgz --keyring ~/.gnupg/secring.gpg #使用公钥校验
helm install --verify --keyring ~/.gnupg/secring.gpg releasename localrepo/firstchart
```

### 9. Starter

```bash
helm env HELM_DATA_HOME
```

新建文件夹`/Users/xxx/Library/helm/starters`

替换原先的 chart name 为占位符`<CHARTNAME>`

```bash
helm create --starter springwebappmysql demoapp
```

### 10. Plugins

```bash
helm plugin list
helm plugin install https://github.com/salesforce/helm-starter.git #--verison xxx
#url也可以替换成本地目录或文件
helm starter --help #显示插件使用帮助
helm plugin update starter
helm plugin remove starter
```
