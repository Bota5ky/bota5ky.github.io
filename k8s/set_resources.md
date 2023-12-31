`pod-definition.yaml` : 也可以手动设定所需资源

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
    resources:
      requests:  ##没有足够的资源创建，则pod会pending
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
```

单位：

- CPU：最小0.1，代表100M，M - million。1 CPU代表1 vCPU aws | 1 core in GCP | 1 hyper thread
- memory：1G - Gigabyte，1M - Megabyte，1K - Kilobyte，1Gi - Gibibyte，1Mi - Mebibyte，1Ki - Kibibyte，带i是精确的用1024换算

默认：

- Docker默认不限制container的CPU使用，Kubernetes默认设置1 CPU
- 对于memory也是类似，Kubernetes默认设置512 Mi

超限：

- Kubernetes默认throttle，CPU不会超出限制
- container可以超限使用内存，pod使用超出限制的内存，则会被terminate

使用默认值前需要在Namespace创建LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/

**References:**

https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource

已存在的pod属性不可编辑，除了以下部分属性：
- `spec.containers[*].image`
- `spec.initContainers[*].image`
- `spec.activeDeadlineSeconds`
- `spec.tolerations`

运行edit编辑pod不可修改的属性，会显示 `Forbidden: pod updates may not change fields other than ...` 的详细信息，并生成临时copy，可以利用copy修改需要的属性，删除原有pod后可根据临时文件生成新的pod。

编辑deployment不受影响，新的配置会自动删除原来的pod生成新的。

`kubectl replace --force -f pod-def.yaml`