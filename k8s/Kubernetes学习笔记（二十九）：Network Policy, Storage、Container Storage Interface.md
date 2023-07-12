### Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress ## 设置入口规则，不会影响出口
  - Egress
  ingress:
  - from:
    - podSelector: ## 规则1
        matchLabels:
          name: api-pod
      namespaceSelector: ## 限制namespace，这里必须满足pod筛选条件
        matchLabels:
          name: prod
    - ipBlock: ## 2个规则其一通过即可
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr:
    ports:
    - protocol: TCP
      port: 80
```

查询方式：`kubectl get netpol` 或 `kubectl get networkpolicy`

### Storage

安装Docker后，默认存储路径：

```
/var/lib/docker
|- aufs
|- containers
|- image
|- volumes
```

对于Dockerfile buid时的分层架构：

`Dockerfile`

```dockerfile
FROM Ubuntu
RUN apt-get update && apt-get -y install python
RUN pip install flask flask-mysql
COPY . /opt/source-code
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

`docker build Dockerfile -t my-name/my-custom-app`

```
Layer 1. Base Ubuntu Layer       ## 120MB
Layer 2. Changes in apt packages ## 360MB
Layer 3. Changes in pip packages ## 6.3MB 前3层可以复用
Layer 4. Source code             ## 229B
Layer 5. Update Entrypoint       ## 0B
```

以上都是Image Layers（readonly），Container Layer只有当Container存在时存在

```
Layer 6. Container Layer         ## read & write
```

COPY-ON-WRITE：Image Layers的文件虽然只读，但仍然可以修改，在保存修改之前，Docker会自动在读写层中创建该文件的副本，然后在读写层中修改该文件的不同版本。

volumes：保留持久化数据，即使container被破坏

```
docker volume create data_volume

/var/lib/docker
|- volumes
   |- data_volume
```

设置image的默认存储数据位置：`docker run -v data_volume:/var/lib/mysql mysql`，即使在执行这条命令前没有运行创建卷 `docker volume create`，Docker将会自动创建相对应的卷。

创建的数据卷会挂载到container中的 `/var/lib/mysql`

绑定挂载：如果数据已存在，则需要给出完整的路径，`docker run -v /data/mysql:/var/lib/mysql mysql`

`-v`是旧式的写法，现在一般推荐使用`--mount`，更清晰明了：`docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql`

### Container Storage Interface

如同Container Runtime Interface（CRI），Container Storage Interface（CSI）可以适配不同的存储驱动，而不用依赖于Kubernetes本身的代码。

RPC（Remote Procedure Call）：远程过程调用