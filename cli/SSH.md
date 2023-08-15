### 1. 常用命令

```bash
# 1.使用ssh连接远程主机
# 最简单的用法只需要指定用户名和主机名参数即可，主机名可以是IP地址或者域名。
ssh user@hostname

# 2.ssh连接到其他端口
# SSH 默认连接到目标主机的22端口上，可以使用-p选项指定端口号
ssh -p 10022 user@hostname

# 3.使用ssh在远程主机执行一条命令并显示到本地, 然后继续本地工作
# 直接连接并在后面加上要执行的命令就可以了
ssh pi@10.42.0.47 ls -l

# 4.在远程主机运行一个图形界面的程序
# 使用ssh的-X选项，然后主机就会开启X11转发功能
ssh -X feiyu@222.24.51.147

# 5.构建ssh密钥对
# 使用ssh-keygen -t <rsa|dsa|...>，现在大多数都使用rsa或者dsa算法。
ssh-keygen -t rsa

# 6.使用-F选项查看是否已经添加了对应主机的密钥
ssh-keygen -F 222.24.51.147

# 7，使用-R选项删除主机密钥，也可以在~/.ssh/known_hosts文件中手动删除
ssh-keygen -R 222.24.51.147

# 8.绑定源地址
# 如果你的客户端有多于两个以上的IP地址，你就不可能分得清楚在使用哪一个IP连接到ssh服务器。为了解决这种情况，我们可以使用-b选项来指定一个IP地址。这个IP将会被使用做建立连接的源地址。
ssh -b 192.168.0.200  root@192.168.0.103

# 9.对所有数据请求压缩
# 使用 -C 选项，所有通过ssh发送或接收的数据将会被压缩，并且仍然是加密的。
ssh -C root@192.168.0.103

# 10.打开调试模式
# 因为某些原因，我们想要追踪调试我们建立的ssh连接情况。ssh提供的-v选项参数正是为此而设的。其可以看到在哪个环节出了问题。
ssh -v root@192.168.0.103

# 11.上传公钥到服务器，之后就可以免密登录
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.235.22
```

### 2. /etc/ssh/sshd_config 配置文件详细说明

```ini
# 设置sshd监听的端口号
Port 2

# 设置sshd服务器绑定的IP地址
ListenAddress 192.168.1.1

# 设置包含计算机私人密匙的文件
HostKey /etc/ssh/ssh_host_key

# 定义服务器密匙的位数
ServerKeyBits 1024

# 设置如果用户不能成功登录，在切断连接之前服务器需要等待的时间（以秒为单位）
LoginGraceTime 600

# 这个参数的是意思是每5分钟，服务器向客户端发一个消息，用于保持连接
ClientAliveInterval 300（默认为0）

# 设置在多少秒之后自动重新生成服务器的密匙（如果使用密匙）。重新生成密匙是为了防止用盗用的密匙解密被截获的信息
KeyRegenerationInterval 3600

# 设置root能不能用ssh登录。这个选项一定不要设成"yes"
PermitRootLogin no

# 设置验证的时候是否使用"rhosts"和"shosts"文件
IgnoreRhosts yes

# 设置ssh daemon是否在进行RhostsRSAAuthentication安全验证的时候忽略用户的"$HOME/.ssh/known_hosts
IgnoreUserKnownHosts yes

# 设置ssh在接收登录请求之前是否检查用户家目录和rhosts文件的权限和所有权。这通常是必要的，因为新手经常会把自己的目录和文件设成任何人都有写权限
StrictModes yes

# 设置是否允许X11转发
X11Forwarding no

# 设置sshd是否在用户登录的时候显示"/etc/motd"中的信息
PrintMotd yes

# 设置在记录来自sshd的消息的时候，是否给出"facility pre"
SyslogFacility AUTH

# 设置记录sshd日志消息的层次
LogLevel INFO

# 设置只用rhosts或"/etc/hosts.equiv"进行安全验证是否已经足够了
RhostsAuthentication no

# 设置是否允许用rhosts或"/etc/hosts.equiv"加上RSA进行安全验证
RhostsRSAAuthentication no

# 设置是否允许只有RSA安全验证
RSAAuthentication yes

# 设置是否允许口令验证
PasswordAuthentication yes

# 设置是否允许用口令为空的帐号登录
PermitEmptyPasswords no
```

### 3. ssh隧道

#### 3.1 机器IP:

- 10.211.55.3
- 10.211.55.7 装有 nginx

#### 3.2 本地转发，将远程机器的指定端口，通过本地的一个端口转发，适用于本地仅能连接远程机器的 ssh 端口

“-L选项”：表示使用本地端口转发创建 ssh 隧道

“-R选项”：表示使用远程端口转发创建 ssh 隧道

“-N选项”：表示创建隧道以后不连接到 sshServer 端，通常与`-f`选项连用

"-f选项"：表示在后台运行 ssh 隧道，通常与`-N`选项连用

```bash
ssh -fNL 12345:10.211.55.7:80 root@10.211.55.7
```

访问地址：http://localhost:12345/

#### 3.3 同上，区别：适用于远程机器仅能通过中间跳板机连接，本地无法直接访问

```bash
ssh -fNL 12345:10.211.55.7:80 root@10.211.55.7 -J root@10.211.55.3
```

访问地址：http://localhost:12345/

#### 3.4 远程转发，适用于自己有公网IP的虚机，且开放了对应端口的防火墙，并且远程虚机修改过对应配置

```ini
/etc/ssh/sshd_config
GatewayPorts yes
```

```bash
ssh -fNTR 71.132.36.61:10035:0.0.0.0:80 ec2-user@71.132.36.61 -i ./.ssh/temp.pem
```

访问地址：http://71.132.36.61:12345/