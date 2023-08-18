### 1. 原理
git 工作区（workspace）、暂存区（index/staging）、版本库（local repository）

<center><img src="git.png" style="zoom:90%"/></center>

### 2. 常用命令

```bash
git init                                                  # 初始化本地git仓库（创建新仓库）
git config --global user.name "xxx"                       # 配置用户名
git config --global user.email "xxx@xxx.com"              # 配置邮件
git config --global color.ui true                         # git status等命令自动着色
git config --global color.status auto
git config --global color.diff auto
git config --global color.branch auto
git config --global color.interactive auto
git clone git+ssh://git@192.168.53.168/VT.git             # clone远程仓库
git status                                                # 查看当前版本状态（是否修改）
git add xyz                                               # 添加xyz文件至index
git add .                                                 # 增加当前子目录下所有更改过的文件至index，包括untrack的文件，会根据.gitignore过滤
git add *                                                 # 会忽略.gitignore过滤
git add -p                                                # 交互式选择代码片段，辅助整理出所需的patch
git commit -m 'xxx'                                       # 提交
git commit --amend -m 'xxx'                               # 合并上一次提交（用于反复修改）
git commit -am 'xxx'                                      # 将add和commit合为一步，一般不用，不包含untrack的文件
git rm xxx                                                # 删除index中的文件
git rm -r *                                               # 递归删除
git log                                                   # 显示提交日志
git log -1                                                # 显示1行日志 -n为n行
git log -5
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
git diff                                                  # 显示所有未添加至index的变更
git diff --cached                                         # 显示所有已添加index但还未commit的变更，和--staged相同
git diff HEAD^                                            # 比较与上一个版本的差异
git diff HEAD -- ./lib                                    # 比较与HEAD版本lib目录的差异
git diff origin/master..master                            # 比较远程分支master上有本地分支master上没有的
git diff origin/master..master --stat                     # 只显示差异的文件，不显示具体内容
git remote                                                # 列出它存储的远端仓库别名
git remote -v                                             # 还可以看到每个别名的实际链接地址
git remote add origin git+ssh://git@192.168.53.168/VT.git # 增加远程定义（用于push/pull/fetch）
git branch                                                # 显示本地分支
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
git push -f                                               # 暴力覆盖
git fetch                                                 # 获取所有远程分支（不更新本地分支，另需merge）
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
git stash                                                 # 暂存所有未提交修改（包括暂存的和非暂存的）
git stash list                                            # 查看所有暂存
git stash save "test-cmd-stash"                           # 暂存时添加message
git stash show -p stash@{0}                               # 参考第一次暂存
git stash apply stash@{0}                                 # 应用第一次暂存但不删除
git stash pop                                             # 应用堆栈中第一个暂存并删除
git stash drop stash@{0}                                  # 删除暂存
git stash clear                                           # 删除所有缓存的stash
git grep "delete from"                                    # 文件中搜索文本“delete from”
git grep -e '#define' --and -e SORT_DIRENT
git gc
git fsck
```

### 3. 常用设置

Q: git pull 出错 fatal: Could not read from remote repository.Please make sure you ha...

A: 出现这个问题是因为，没有在github账号添加[SSH key](https://bbs.csdn.net/topics/391950974)

```bash
git config  --global user.name "yourName"
git config  --global user.email "yourEmail"
```
SSO 需要在 GitHub 授权

```bash
ssh-keygen
ls -al ~/.ssh
less ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub
```

git 如何设置使用代理

```bash
git config --global https.proxy http://127.0.0.1:80809
git config --global https.proxy https://127.0.0.1:80809
git config -l --global  # 查看设置，SSH访问不通过SOCK5或HTTP，所以以上设置其实无效，需要进行以下设置
```

git 设置 SSH 代理

ssh 配置文件地址为：`~/.ssh/config`
windows 中就是：`C:\Users\你的用户名\.ssh\config` (若不存在自行创建)
配置文件中增加以下内容：

```bash
Host github.com *.github.com
    User git
    # SSH默认端口22， HTTPS默认端口443
    Port 22
    Hostname %h
    # 这里放你的SSH私钥
    IdentityFile ~\.ssh\id_rsa
    # 设置代理, 127.0.0.1:10808 换成你自己代理软件监听的本地地址
    # HTTPS使用-H，SOCKS使用-S
    ProxyCommand connect -S 127.0.0.1:10808 %h %p
```

CTRL+R 自动查找命令历史，CTRL+E 使用历史命令

### 4. 参考资料

- [Git新命令switch和restore](https://zhuanlan.zhihu.com/p/259385054?utm_source=wechat_timeline)
- [git docs](https://git-scm.com/docs)
- [Beginner’s Guide to using Git](https://medium.com/cs-code/beginners-guide-to-using-git-8e5001791fa6)
- [git - 简明指南](https://www.bootcss.com/p/git-guide/)
- [Git 取消commit](https://www.cnblogs.com/lyy-2016/p/6509707.html)
- [Git rebase 用法小结](https://www.jianshu.com/p/4a8f4af4e803)
- [Git Reset 三种模式](https://www.jianshu.com/p/c2ec5f06cf1a)
- [Git Reset 三种模式 详细](https://blog.csdn.net/xuw_xy/article/details/121653777)