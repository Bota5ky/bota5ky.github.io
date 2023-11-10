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

机器IP:

- 10.211.55.3
- 10.211.55.7 装有 nginx

本地转发，将远程机器的指定端口，通过本地的一个端口转发，适用于本地仅能连接远程机器的 ssh 端口：

`-L`：表示使用本地端口转发创建 ssh 隧道

`-R`：表示使用远程端口转发创建 ssh 隧道

`-N`：表示创建隧道以后不连接到 sshServer 端，通常与`-f`选项连用

`-f`：表示在后台运行 ssh 隧道，通常与`-N`选项连用

```bash
ssh -fNL 12345:10.211.55.7:80 root@10.211.55.7
```

访问地址：http://localhost:12345/

同上，区别：适用于远程机器仅能通过中间跳板机连接，本地无法直接访问：

```bash
ssh -fNL 12345:10.211.55.7:80 root@10.211.55.7 -J root@10.211.55.3
```

访问地址：http://localhost:12345/

远程转发，适用于自己有公网IP的虚机，且开放了对应端口的防火墙，并且远程虚机修改过对应配置：

```ini
/etc/ssh/sshd_config
GatewayPorts yes
```

```bash
ssh -fNTR 71.132.36.61:10035:0.0.0.0:80 ec2-user@71.132.36.61 -i ./.ssh/temp.pem
```

访问地址：http://71.132.36.61:12345/

### 4. Git 命令备忘

```bash
git add *                                                 # 会忽略.gitignore过滤
git commit --amend -m 'xxx'                               # 合并上一次提交
git commit -am 'xxx'                                      # 添加同时提交，不包含untrack的文件
git rm xxx                                                # 删除index中的文件
git rm -r *                                               # 递归删除
git log -n                                                # 显示n行日志
git log --stat                                            # 显示提交日志及相关变动文件
git log -p -m
git show                                                  # 默认显示HEAD日志的相关信息
git show dfb02e6e4f2f7b573337763e5c0013802e392818         # 显示某个提交的详细内容
git show dfb02                                            # 可只用commitid的前几位
git show HEAD                                             # 显示HEAD提交日志
git show HEAD^                                            # 显示HEAD上一个版本的提交日志^^为上两个版本^5为上5个版本
git tag                                                   # 显示已存在的tag
git tag -a v2.0 -m 'xxx'                                  # 增加v2.0的tag
git show v2.0                                             # 显示v2.0的日志及详细内容
git log v2.0                                              # 显示v2.0的日志
git log --oneline                                         # 一行显示日志
git diff HEAD^                                            # 比较与上一个版本的差异
git diff HEAD -- ./lib                                    # 比较与HEAD版本lib目录的差异
git diff origin/master..master                            # 比较远程分支master上有本地分支master上没有的
git diff origin/master..master --stat                     # 只显示差异的文件，不显示具体内容
git remote                                                # 列出它存储的远端仓库别名
git remote -v                                             # 还可以看到每个别名的实际链接地址
git remote add origin git+ssh://git@192.168.53.168/VT.git # 增加远程定义（用于push/pull/fetch）
git branch --contains 50089                               # 显示包含提交50089的分支
git branch -a                                             # 显示所有分支
git branch -r                                             # 显示所有原创分支
git branch --merged                                       # 显示所有已合并到当前分支的分支
git branch --no-merged                                    # 显示所有未合并到当前分支的分支
git branch -m master master_copy                          # 本地分支改名
git checkout -b master_copy                               # 从当前分支创建新分支master_copy并检出
git checkout -b master master_copy                        # 上面的完整版
git checkout features/performance                         # 检出已存在的features/performance分支
git checkout --track hotfixes/BJVEP933                    # 检出远程分支hotfixes/BJVEP933并创建本地跟踪分支
git checkout v2.0                                         # 检出版本v2.0
git checkout -b devel origin/develop                      # 从远程分支develop创建新本地分支devel并检出
git checkout -- README                                    # 检出head版本的README文件（可用于修改错误回退）
git merge origin/master                                   # 合并远程master分支至当前分支
git cherry-pick ff44785404a8e                             # 合并提交ff44785404a8e的修改
git push origin master                                    # 将当前分支push到远程master分支
git push origin :hotfixes/BJVEP933                        # 删除远程仓库的hotfixes/BJVEP933分支
git push --tags                                           # 把所有tag推送到远程仓库
git fetch --prune                                         # 获取所有原创分支并清除服务器上已删掉的分支
git pull --rebase
git pull origin master                                    # 获取远程分支master并merge到当前分支
git mv README README2                                     # 重命名文件README为README2
git reset --soft HEAD                                     # 重置位置的同时，保留working Tree工作目录和index暂存区的内容，只让repository中的内容和 reset 目标节点保持一致，因此原节点和reset节点之间的「差异变更集」会放入index暂存区中(Staged files)。所以效果看起来就是工作目录的内容不变，暂存区原有的内容也不变，只是原节点和Reset节点之间的所有差异都会放到暂存区中
git reset (--mixed) HEAD                                  # 重置位置的同时，只保留Working Tree工作目录的內容，但会将 Index暂存区 和 Repository 中的內容更改和reset目标节点一致，因此原节点和Reset节点之间的「差异变更集」会放入Working Tree工作目录中。所以效果看起来就是原节点和Reset节点之间的所有差异都会放到工作目录中
git reset --hard HEAD                                     # 重置位置的同时，直接将 working Tree工作目录、 index 暂存区及 repository 都重置成目标Reset节点的內容,所以效果看起来等同于清空暂存区和工作区
git rebase
git rebase -i head~4                                      # 查看当前后面的4个提交
git branch -d hotfixes/BJVEP933                           # 删除分支hotfixes/BJVEP933（本分支修改已合并到其他分支）
git branch -D hotfixes/BJVEP933                           # 强制删除分支hotfixes/BJVEP933
git ls-files                                              # 列出git index包含的文件
git show-branch                                           # 图示当前分支历史
git show-branch --all                                     # 图示所有分支历史
git whatchanged                                           # 显示提交历史对应的文件修改
git revert dfb02e6e4f2f7b573337763e5c0013802e392818       # 撤销提交dfb02e6e4f2f7b573337763e5c0013802e392818
git ls-tree HEAD                                          # 内部命令：显示某个git对象
git rev-parse v2.0                                        # 内部命令：显示某个ref对于的SHA1 HASH
git reflog                                                # 显示所有提交，包括孤立节点
git show HEAD@{5}
git show master@{yesterday}                               # 显示master分支昨天的状态
git log --pretty=format:'%h %s' --graph                   # 图示提交日志
git show HEAD~3
git show -s --pretty=raw 2be7fcb476
git stash list                                            # 查看所有暂存
git stash save "test-cmd-stash"                           # 暂存时添加message
git stash show -p stash@{0}                               # 参考第一次暂存
git stash apply stash@{0}                                 # 应用第一次暂存但不删除
git stash drop stash@{0}                                  # 删除暂存
git stash clear                                           # 删除所有缓存的stash
git grep "delete from"                                    # 文件中搜索文本“delete from”
git grep -e '#define' --and -e SORT_DIRENT
git gc
git fsck
```

### 5. PowerShell

[How do you set PowerShell's default directory?](https://stackoverflow.com/questions/32069265/how-do-you-set-powershells-default-directory)

- 删除文件

```powershell
Remove-Item -Path "*" -Include "*.txt" -Recurse -Force
```

- 获取文件个数

```powershell
Get-ChildItem <path> -Include *.dump -Recurse | select Name | Measure-Object
```

- 删除Sample开头的sqlserver数据库

```powershell
$Databases = Invoke-SQLcmd -ServerInstance $ServerInstance -Query ("SELECT * FROM sys.databases WHERE NAME LIKE 'Sample%'")
ForEach ($Database in $Databases){Invoke-SQLcmd -ServerInstance $ServerInstance -Query ("DROP DATABASE [" + $Database.Name + "]")
"$($Database.Name) is deleted."}
```

- dump文件转postgres

```powershell
psql -U postgres -d <databaseName> -f 'C:\xxxxxx.dump'
```

- 删除Sample开头的postgres数据库

```powershell
psql -U postgres -t -A -c "select datname from pg_database where datname ~ 'Sample\w*'" | ForEach-Object { dropdb --force --echo --username postgres "$_" }
```

