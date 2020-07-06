---
title: git的使用
date: 2018-10-08 00:21:52
tags: 
- Git
comments: true
toc: true
mathjax: true
urlname:
categories:
thumbnail:
---
创建版本库

```
mkdir learngit
cd learngit
```

# 初始化一个Git仓库

```
git init
```

# 添加文件到Git仓库，分两步：

```
git init
git add <filename>
git commit -m <message>
```
使用命令git add <file>，注意，可反复多次使用，添加多个文件；
使用命令git commit -m <message>，完成。
参考
[创建仓库/添加文件/提交](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013743256916071d599b3aed534aaab22a0db6c4e07fd0000)

# 查看哪些文件修改

```
git status
```
# 查看修改的内容

```
git diff httpserver.js
```

# 查看提交日志

```
git log
```
# 返回上一个版本

```
git reset  --hard HEAD~1
```
# 工作区、版本库、暂存区

![](http://localhost:9001/api/file/getImage?fileId=5b71273016454611ab000013)

# 放弃工作区的修改
讲工作区的修改撤销为“git commit”或者“git add”时候的状态
例如加入“I like men”

 - 未“git add”，则撤销后为原状态
 - 已“git add”，则撤销后为 I like men 状态

```
git checkout -- httpserver.js
```
--代表master支线

# 放弃暂存区修改
已经“git add”加入暂存区了，只能用“git reset HEAD”去撤销修改

```
git reset HEAD httpserver.js
```
然后撤销工作区修改

```
git checkout -- httpserver.js
```
如果已经“git commit”，那么“返回上一个版本”

```
git reset --hard HEAD~1
```
# 删除工作区文件

```
rm httpserver.js
```
# 删除暂存区文件

```
git rm httpserver.js
```
# 删除master分支文件

```
git rm httpserver.js
git commit -m "delete it"
```
# 跟远程仓库连接
第1步：创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

$ ssh-keygen -t rsa -C "youremail@example.com"
你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码。

如果一切顺利的话，可以在用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。

第2步：登陆GitHub，打开“Account settings”，“SSH Keys”页面：

然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容
为什么GitHub需要SSH Key呢？因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。

当然，GitHub允许你添加多个Key。假定你有若干电脑，你一会儿在公司提交，一会儿在家里提交，只要把每台电脑的Key都添加到GitHub，就可以在每台电脑上往GitHub推送了。

最后友情提示，在GitHub上免费托管的Git仓库，任何人都可以看到喔（但只有你自己才能改）。所以，不要把敏感信息放进去。

如果你不想让别人看到Git库，有两个办法，一个是交点保护费，让GitHub把公开的仓库变成私有的，这样别人就看不见了（不可读更不可写）。另一个办法是自己动手，搭一个Git服务器，因为是你自己的Git服务器，所以别人也是看不见的。这个方法我们后面会讲到的，相当简单，公司内部开发必备。

确保你拥有一个GitHub账号后，我们就即将开始远程仓库的学习。
```
git remote add origin git@github.com：tkhfree/learn_git.git
```

# 推送本地仓库到远程仓库

```
git remote add origin git@github.com：tkhfree/learn_git.git
git push -u origin master
```
# 从远程仓库克隆到本地仓库

```
git clone git@github.com:tkhfree/learn_git.git
```

# 创建并切换分支

```
git checkout -b dev
```
等于

```
git branch dev
git checkout dev
```
# 查看当前分支

```
git branch
```
 #分支合并

```
git merge dev
```
# 删除分支

```
git branch -d dev
```
# bug分支

 Git还提供了一个stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作

工作区和暂存区是一个公开的工作台，任何分支都会用到，并能看到工作台上最新的内容，只要在工作区、暂存区的改动未能够提交到某一个版本库（分支）中，那么在任何一个分支下都可以看得到这个工作区、暂存区的最新实时改动。
使用git stash就可以将暂存区的修改藏匿起来，使整个工作台看起来都是干净的。所以要清理整个工作台，那么前提是必须先将工作区的内容都add到暂存区中去。之后在干净的工作台上可以做另外一件紧急事件与藏匿起来的内容是完全独立的

```
git stash
```

```
git stash list
```

```
git stash pop
```
## **为什么要用git stash。**
1、在dev分支，创建一个新文件test6.txt。并add它，让它stage。
2、这时切回master分支，你会看到这个test6.txt居然也在master分支里。但它实际上是属于dev分支的。怎么办？
3、git stash就有作用了。切回dev分支，执行git stach。这时git bash会告诉你
“Saved working directory and index state WIP on newF2: b63fbcb add test6.txt
HEAD is now at b63fbcb add test6.txt”。它已经把test6.txt的现场保存好了。
4、这时你在切回master分支，test6.txt就消失了。

# 查看远程仓库的信息

```
git remote -v
```
# 从本地仓库任一分支推送到远程仓库

```
git push origin master
git push origin dev
```
# 从远程仓库分支推送到本地创建新分支

```
git checkout -b dev origin/dev
```
# 协同工作提交分支至远程仓库
当你提交的和另一个人提交的有冲突，则需要先pull下来，在本地合并，再push上远程仓库

```
git branch --set-upstream-to=origin/dev dev
git pull
git push origin dev
```

----------
因此，多人协作的工作模式通常是这样：

首先，可以试图用git push origin <branch-name>推送自己的修改；

如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

如果合并有冲突，则解决冲突，并在本地提交；

没有冲突或者解决掉冲突后，再用git push origin <branch-name>推送就能成功！

如果git pull提示no tracking information，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream-to <branch-name> origin/<branch-name>。

----------


