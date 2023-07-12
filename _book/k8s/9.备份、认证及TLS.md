`kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml`

与其备份单个资源，不如备份ETCD：

`etcd.service`

```yaml
  --data-dir=/var/lib/etcd
```

etcd也自带快照功能

```shell
ETCDCTL_API=3 etcdctl \  ##根据etcdctl版本 不想重复写就export ETCDCTL_API=3 设置全局参数
	snapshot save snapshot.db
	snapshot status snapshot.db  ##查看备份状态
```

Restore：会初始化新的集群配置，将etcd配置为新成员，以防止新成员加入现有集群

```shell
ETCDCTL_API=3 etcdctl \
service kube-apiserver stop
snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup  ##使用新的数据目录，将备份还原到此数据目录
systemctl daemon-reload
service etcd restart
service kube-apiserver start
```

可以对备份指定访问端口和密钥信息：

```shell
ETCDCTL_API=3 etcdctl \
snapshot save snapshot.db  ##如果启用了TLS，以下选项是强制性的
  --endpoints=https://127.0.0.1:2379
  --cacert=/etc/etcd/ca.crt
  --cert=/etc/etcd/etcd-server.crt
  --key=/etc/etcd/etcd-server.key
```

还原后需要设置新的 `etcd-data` 的 `hostPath`：

```yaml
volumes:
- hostPath:
    path: /var/lib/etcd  ##修改为etcd-from-backup新目录
    type: DirectoryOrCreate
  name: etcd-data
```

查看 node 相关的 clusters：

```bash
kubectl config view
kubectl config get-clusters
```

更换 node 上的 cluster context：

```bash
kubectl config use-context cluster1
```

查看ETCD服务器所属的ETCD集群中有多少节点：

```bash
ETCDCTL_API=3 etcdctl \
 --endpoints=https://127.0.0.1:2379 \
 --cacert=/etc/etcd/pki/ca.pem \
 --cert=/etc/etcd/pki/etcd.pem \
 --key=/etc/etcd/pki/etcd-key.pem \
  member list
```

跨node复制文件：`scp cluster1-controlplane:/opt/cluster1.db /opt/cluster1.db`

如果是外部的etcd，需要修改 `/etc/systemd/system/etcd.service`中的 `data-dir`，添加新路径的etcd权限 `chown -R etcd:etcd /var/lib/etcd-data-new`，最后重启服务 `systemctl daemon-reload`, `systemctl restart etcd`

### Authentication

- 禁用基于密码的身份验证
- 仅提供基于SSH密钥的身份验证

第一道防线：控制对API服务器本身的访问

- 用户名+密码
- 用户名+Token
- 证书
- 与LDAP等外部身份验证提供程序集成
- 服务账户

可以做什么？

- RBAC Authorization 基于角色的访问控制
- ABAC Authorization 基于属性的访问控制
- Node Authorization 基于节点的访问控制
- Webhook Mode

`kubectl create serviceaccount sa1`

`kubectl get serviceaccount`

验证方式`kube-apiserver`：

- Static Password File
- Static Token File
- Certificates
- Identity Services，第三方身份验证协议，如LDAP、Kerberos等

### Static Password File ：

`user-details.csv` :

```
password,username,userid(,groupname optional)
...
```

`user-token-details.csv` :

```
token,username,userid(,groupname optional)
...
```

定义方式：

- `--basic-auth-file=user-details.csv` `--token-auth-file=user-details.csv` apiserver重启才能生效

- kubeadm `/etc/kubernetes/manifests/kube-apiserver.yaml  `

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: null
    name: kube-apiserver
    namespace: kube-system
  spec:
    containers:
    - command:
      - kube-apiserver
      - --authorization-mode=Node,RBAC
      - --advertise-address=172.17.0.107
      - --allow-privileged=true
      - --enable-admission-plugins=NodeRestriction
      - --enable-bootstrap-token-auth=true
      image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
      name: kube-apiserver
  ```

访问：

- `curl -v -k https://master-node-ip:6443/api/v1/pods -u "username:password" {data...}`

- `curl -v -k <link> --header "Authorization: Bearer xxxxxxxx"`

注意点：

- 不推荐使用静态储存（在v1.19已弃用）
- Consider volume mount while providing the auth file in a kubeadm setup
- Setup Role Based Authorization for the new users

### TLS

查看Common Name (CN)：`openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text`

Symmetric Encryption：对称加密，使用相同的密钥来加密和解密数据，必须在发送方和接收方之间交换，因此存在风险

Asymmetric Encryption：非对称加密，Private Key和Public Lock

`ssh-keygen` 生成私钥 `id_rsa` 和公钥 `id_rsa.pub`

添加公钥：通常是在服务器SSH授权的下划线密钥文件中添加一个包含公钥的条目来完成的 `cat ~/.ssh/authorized_keys`

```
ssh-rsa AAAAB3Nza... user1
ssh-rsa AAAXCV2b8... user2
```

指定私钥 `ssh -i id_rsa user1@server1`

使用openssl生成私钥、公钥：

```bash
openssl genrsa -out my-bank.key 1024  ## private key
## my-bank.key
openssl rsa -in my-bank.key -pubout > mybank.pem  ## public key
## my-bank.key mybank.pem
```

证书包含：

- 有关谁向该服务器的公钥颁发证书的信息
- 该服务器地址
- ...

生成证书的时候必须带有签名，自签名的证书会被认为非法

Certificate Authority (CA) 负责签署和验证证书，比较著名的有 Symantec、Desert、Comodo、Global Sign 等等

签名的方式：

- 用之前生成的密钥和网站域名生成Certificate Signing Request (CSR)

  ```bash
  openssl req -new -key my-bank.key -out my-bank.csr -subj "/C=US/ST=CA/O=MyOrg, Inc./CN=mydomain.com"  ## 对于hacker Validate Information会失败
  ## my-bank.key my-bank.csr
  ```

- 验证通过，Sign and Send Certificate

CA合法性的验证：签名用私钥，所有CA的公钥都存在浏览器中

私有CA：一样的运作方式，为组织内所有浏览器安装私有CA的公钥

![](https://img2022.cnblogs.com/blog/2122768/202208/2122768-20220828202435126-1276433229.png)

Kubernetes要求集群至少有一个证书颁发机构CA，也可以设置多个

#### 之前尝试使用命令行登陆证书过期的https失败

```shell
openssl x509 -inform der -in \*.thoughtworks.cn.cer -out certificate.pem
```
但是缺少私钥，一般会生成公钥和私钥，或者合并为同一份pem文件。
```shell
curl --cert certificate.pem --header 'Content-Type: application/json' -d '{"captcha": "111", "captchaId": "captchaId", "password": "password", "username": "user"}' --request POST https://sample.com
```

更改hosts方法
```bash
sudo vim /etc/hosts
```

如果api-server不可用，用命令查看频繁退出的container的id `crictl ps -a | grep kube-apiserver`，并查看日志 `crictl logs --tail=2 1fb242055cff8`找出原因 - Container Runtime Interface (CRI)

### Certificate

#### 证书创建

证书生成工具：easyrsa、openssl、cfssl

生成CA certificate步骤：

- Generate Keys

  `openssl genrsa -out ca.key 2048`

- Certificate Signing Request

  `openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr`

- Sign Certificates : CA创建root certificate是自我签名

  `openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt`

生成client certificate步骤：

- Generate Keys

  `openssl genrsa -out admin.key 2048`

- Certificate Signing Request

  `openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr`

- Sign Certificates : CA创建root certificate是自我签名

  `openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt`

可以通过在证书中添加用户组详细信息以区分不同的注册用户：`openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr`

系统组件其名称必须以关键字system作为前缀

kube-api server 有多个别名：kubernetes、kubernetes.default、kubernetes.default.svc、kubernetes.default.svc.cluster.local

kube-api 添加别名方式：

```bash
openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr -config openssl.cnf
```

openssl.cnf

```yaml
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 1720.17.0.87
```

#### 查看证书细节

x509解码证书以查看详细信息：

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

如果核心组件（如kubernetes api-server或etcd-server关闭），kubectl命令无法使用，可以用docker命令查看日志

Kubernetes Certificate Health Check Spreadsheet : https://github.com/mmumshad/kubernetes-the-hard-way/tree/master/tools

### Certificates API

用户创建key：

```bash
openssl genrsa -out jane.key 2048
```

用户把key发给admin，admin用这个key创建certificate signing request对象：

```bash
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr
```

`Jane-csr.yaml` `kubectl create -f Jane-csr.yaml`

```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  request:
    ## 请求内容用base64 encode
    ## cat jane.csr | base64 | tr -d "\n" 或 base64 -w 0 禁用换行
```

创建对象后，admin可以通过 `kubectl get csr` 查看所有证书请求，通过 `kubectl certificate approve jane`，拒绝则用 `deny`，删除则用 `kubectl delete csr jane`

通过以yaml格式查看证书 `kubectl get csr jane -o yaml`，用base64解码 `echo "...=" | base64 --decode`

所有与证书相关的操作都由Controller Manage执行：其中包含CSR-APPROVING、CSR-SIGNING等控制器

其中的key和根证书在 `/etc/kubernetes/manifests/kube-controller-manager.yaml` 中的

```yaml
spec:
  containers:
  - command:
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
```