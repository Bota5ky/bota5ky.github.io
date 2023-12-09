### 博客网站

- 技术博客：适合想学硬核技术的同学，比如美团技术团队、阿里技术团队
- 科技资讯类：量子位、差评、新智元、无敌信息差
- 经验分享、编程趋势、技术干货：程序员鱼皮、小林 coding、java guide、程序喵、神光的编程笔记、小白 debug、古时的风筝、苏三、阿秀等

### 技术文章

- [OpenTelemetry 系列四｜如何使用 Java Agent 来实现无侵入的调用链](https://xie.infoq.cn/article/a3814f9326781409a05edc23d)

- [高性能队列——Disruptor](https://tech.meituan.com/2016/11/18/disruptor.html)



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

### Hook

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

### Testing

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

### Repository

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

### Security

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

### Starter

```bash
helm env HELM_DATA_HOME
```

新建文件夹`/Users/xxx/Library/helm/starters`

替换原先的 chart name 为占位符`<CHARTNAME>`

```bash
helm create --starter springwebappmysql demoapp
```

### Plugins

```bash
helm plugin list
helm plugin install https://github.com/salesforce/helm-starter.git #--verison xxx
#url也可以替换成本地目录或文件
helm starter --help #显示插件使用帮助
helm plugin update starter
helm plugin remove starter
```









---

引用 Chart 内的值需要首字母大写。

```yaml
{{.Chart.AppVersion}}
{{.Release.IsUpgrade}}
{{.Template.Name}}
{{.Values.my.custom.data | default "testdefault" | upper | quote}} #设置默认值，加双引号
```

nindent：nested indent 嵌套缩进
