### 1. Volumes
!!!注意题目给的 `mountPath` 后面有没有 `/`，可能会判定为不同路径。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh","-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt ## Container内的存储路径
      name: data-volume

  volumes:
  - name: data-volume
    hostPath:
      path: /data ## 对应存储在host上的路径
      type: Directory
    ## awsElasticBlockStore: ## 可替换hostPath
      ## volumeID:
      ## fsType: ext4
```

### 2. Persistent Volumes

Persistent Volumes Claim (PVC)

`pv-definition.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

管理员创建一组PV，用户创建PVC以用于存储。单个PVC绑定单个PV。

可以使用label进行匹配，如果所有其他条件都匹配，并且没有更好的选项，则较小的PVC可能会绑定到较大的PV，且剩下的存储空间无法再被其他PVC利用。没有PV可用，则PVC会保持pending状态。

```yaml
selector:
  matchLabels:
    name: my-pv ## PVC
```

```yaml
labels:
  name: my-pv ## PV
```

`pvc-definition.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

PVC被删除后，PV的状态取决于：

```yaml
persistentVolumeReclaimPolicy: Retain | Delete | Recycle
```

### 3. Storage Class

在应用程序需要时自动配置 volumes

`sc-definition.yaml` 不需要再配置 pv，由 Storage Class 自动创建

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gec-pd
parameters: ## 取决于磁盘配置
  type: pd-standard
  replication-type: none
```

`pvc-definition.yaml`

```yaml
spec:
  storageClassName: google-storage
```

### 4. 存储

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

### 5. Container Storage Interface

如同 Container Runtime Interface (CRI)、Container Storage Interface (CSI)可以适配不同的存储驱动，而不用依赖于Kubernetes本身的代码。

RPC (Remote Procedure Call)：远程过程调用