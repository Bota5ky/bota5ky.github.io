### 1. Linux 单节点部署

**下载安装**

```bash
tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz -C /opt/module
```

**创建用户**

因为安全问题，Elasticsearch 不允许 root 用户直接运行，所以要创建用户，在 root 用户中创建新用户

```bash
useradd es                      # 新增es用户
passwd es                       # 为es用户设置密码

userdel -r es                   # 如果添加错误可以删除再添加
chown -R es:es /opt/module/es   # 文件夹所有者
```

**修改配置文件**

修改`/opt/module/es/config/elasticsearch.yml` 文件

```ini
# 加入如下配置
cluster.name: elasticsearch
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"]
```

修改`/etc/security/limits.conf`

```ini
# 在文件末尾中增加下面内容
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
```

修改`/etc/security/limits.d/20-nproc.conf`

```ini
# 在文件末尾中增加下面内容
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
```

```ini
# 操作系统级别对每个用户创建的进程数的限制
hard nproc 4096
# 注: 代表 Linux 所有用户名称
```

修改`/etc/sysctl.conf`

```ini
# 在文件中增加下面内容
# 一个进程可以拥有的VMA(虚拟内存区域)的数量，默认值为 65536
vm.max_map_count=655360
```

重新加载

```bash
sysctl -p
```

