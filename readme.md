# get起源，linux创造的分布式版本管理

## 开源协议：

Apache License
使用这个协议可以进行商用
你可以对其修改、分发，但是你要声明作者来源和你的修改以及协议，很多大型项目都使用这个协议，比如 tensorflow、puppeteer。

MIT License
这是个人用得比较多的协议，因为比较宽松精简
只要声明版权和协议就可以了，可以商用、修改、复制、重新发布等操作，使用这个协议的就有vue、react等。

BSD License
这个和 MIT 协议类似除了声明协议和来源，其它基本操作都可以使用，flask 用的就是这个协议。


GNU License
你可以私用也可以商用，但是你必须声明来源，并且需要声明原有的协议以及你的代码也必须开源出来，我们很熟悉的 Linux 就是
采用这种协议，现在知道为什么有那么多免费的 Linux 发行版了就是得益于这个协议。

NO License
也就是什么都不声明，但是并不意味着就可以乱来
这比声明了协议还严格，你可以使用、商用
但是你需要声明协议和来源，而且你不能对代码进行修改、复制、再次发布，不过，你在 GitHub 使用了这个协议还是可以被别人观看代码，fork 操作。

Eclipse License
这个协议允许你商用、复制、修改、再次发布等。
需要声明来源和协议，像 java 中的 junit4 就是使用这个协议。

## ssh密钥生成：

### win10生成sshkeys：

powershell > ssh-keygen -t rsa -C "邮箱名"

![image-20220320131404279](readme.assets/image-20220320131404279.png)

id_rsa私钥一定要保留好！！！

![image-20220320131543897](readme.assets/image-20220320131543897.png)

### Mac生成sshkeys：

```bash
> ssh-keygen -t rsa -b 4096 -C "xxxxxx@email.com"
```

```
# 密钥文件位置
/Users/you/.ssh/id_rsa.pub
```



### Linux生成sshkeys：

```
> ssh-keygen -t ras -C “youremail@domain.com” 
```

```
# 密钥文件位置
vim ~/.ssh/id_rsa.pub
```

## 主流代码托管：

RCS：本地版本控制。一般是个人备份。

![image-20220320130255654](readme.assets/image-20220320130255654.png)

**GIT：分布式版本控制。每个人拥有全部代码，分布式保存代码。**

![image-20220320130612321](readme.assets/image-20220320130612321.png)

SVN ：集中式版本控制。不可靠。

![image-20220320130343670](readme.assets/image-20220320130343670.png)

CVS： 集中式版本控制。

VSS ：集中式版本控制。

TFS ：没用过

## Github用法：

github传入密钥：

![image-20220320125601931](readme.assets/image-20220320125601931.png)

![image-20220320125830338](readme.assets/image-20220320125830338.png)

git的真正用法：

![git命令](readme.assets/git%E5%91%BD%E4%BB%A4.png)



### 创建.gitignore文件

![image-20220320133956828](readme.assets/image-20220320133956828.png)

![image-20220320132605341](readme.assets/image-20220320132605341.png)

当创建.gitignore规则不生效

![image-20220320132732444](readme.assets/image-20220320132732444.png)



#### 示例写法：

```
# .gitignore文件放在根目录下
# 忽略后缀名为class/war/ear的文件，可以用来排除一些特定格式的文件。
*.class
*.war
*.ear
*.zip
*.tar.gz
*.rar

# 忽略target/目录下的文件
target/
build/

# 忽略当前目录下的这几个文件
.settings/
.project
.classpatch

# 忽略当前目录额下的idea/文件夹
.idea/
# 忽略根目录下的deea/文件夹
/idea/
*.ipr
*.iml
*.iws

# 忽略这些后缀文件
*.log
*.cache
*.diff
*.patch
*.tmp

# 忽略这些文件
.DS_Store
Thumbs.db
```

![image-20220320134916429](readme.assets/image-20220320134916429.png)

#### 主要配置：

```
# 查看所有配置
>git config -l

# 查看本地配置
>git config --global --list

# 配置本地用户
>git config --global user.name "name"
>git config --global user.email "you@qq.com"

# 如果需要特殊配置，则修改git文件
./git/etc/gitconfig
```

![image-20220320141657981](readme.assets/image-20220320141657981.png)

```
git r
```







```
# 设置允许git克隆子目录
git config core.sparsecheckout true

# 创建本地空repo
git init myRepo && cd myRepo

# 设置要克隆的仓库的子目录路径, “*” 是通配符，“!” 是反选
echo deployment >> .git/info/sparse-checkout

# 设置远程仓库地址
git remote add origin ssh://github.com/abc.git

# 用 pull 来拉取代码
git pull origin master

#############################
# 如果需要添加目录，就增加sparse-checkout的配置，再checkout master
echo another_folder >> .git/info/sparse-checkout
git checkout master
```

### 其他

```
>git --help
```

```
start a working area (see also: git help tutorial)
   clone             直接复制master or main下全部内容
   init              创建.git文件夹

work on the current change (see also: git help everyday)
   add               加入缓存区
   mv                移动或者重命名
   restore           回退版本
   rm                删除版本
   sparse-checkout   类似.gitignore不过是命令的形式

examine the history and state (see also: git help revisions)
   bisect            使用二分法找错误的提交
   diff              比较缓存区和工作区的差异
   grep              查询工作区内容
   log               查询工作区提交历史
   show              展示对象，文件名， 内容，标记等等
   status            查询缓存区状态

grow, mark and tweak your common history
   branch            分支操作
   commit            提交暂存区到工作区
   merge             分支合并，合并差异
   rebase            合并某个分支到当前分支上
   reset             回退版本
   switch            切换分支，新命令
   tag               标记操作
   checkout          切换分支

collaborate (see also: git help workflows)
   fetch             从远程获取代码库，这个需要仓库权限（一般是加了自己sshkey）
   pull              拉取自己的github仓库
   push              推送自己到自己github仓库


```













