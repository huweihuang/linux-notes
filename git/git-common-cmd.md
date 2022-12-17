---
title: "Git常用命令"
weight: 2
catalog: true
date: 2019-09-20 10:50:57
subtitle:
header-img:
tags:
- Git
catagories:
- Git
---

# 1. Git常用命令

| 分类 | 子类                        | git command                                                  | zsh alias |
| ---- | --------------------------- | ------------------------------------------------------------ | --------- |
| 分支 | 查看当前分支                | `git branch`                                                 | gb        |
|      | 创建新分支,仍停留在当前分支 | git branch <new branch>                                      |           |
|      | 创建并切换到新分支          | git checkout -b <new branch>                                 | gcb       |
|      | 切换分支                    | git checkout <branch>                                        |           |
|      | 合并分支                    | git checkout <branch> #切换到要合并的分支git merge –no-ff <to be merged branch> #合并指定分支到当前分支 |           |
| 提交 | 查看状态                    | git status                                                   | gst       |
|      | 查看修改部分                | git diff --color                                             | gd        |
|      | 添加文件到暂存区            | git add --all                                                |           |
|      | 提交本地仓库                | git commit -m "<message>"                                    |           |
|      | 推送到指定分支              | git push -u origin <branch>                                  |           |
|      | 查看提交日志                | git log                                                      | -         |

# 2. git rebase

如果信息修改无法生效，设置永久环境变量：export EDITOR=vim

帮助信息：

```bash
# Rebase 67da308..6ef692b onto 67da308 (1 command)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

## 2.1. 合并多余提交记录

```bash
#以交互的方式进行rebase
git rebase -i master
 
#合并多余提交记录：s, squash = use commit, but meld into previous commit
pick 6ef692b FIX: Fix parsing docker image version error
s 3df667y FIX: the second push
s 3fds95t FIX: the third push
保存退出
 
# 进入修改交互界面
删除需要删除的提交记录，保存退出
 
#查看提交记录是否已被修改
git log
 
 
#最后强制提交到分支
git commit --force -u origin fix/add-unit-test-for-global-role-revoking
```

## 2.2. 修改提交记录

```bash
#以交互的方式进行rebase
git rebase -i master
 
#修改提交记录：e, edit = use commit, but stop for amending
e 6ef692b FIX: Fix parsing docker image version error
e 5ty697u FIX: Fix parsing docker image version error
#保存退出
 
git commit --amend
#修改提交记录内容，保存退出
 
git rebase --continue
git commit --amend
#修改下一条提交记录，保存退出
 
git rebase --continue
git status # 查看状态提示
 
#最后强制提交到分支
git commit --force -u origin fix/add-unit-test-for-global-role-revoking
 
#查看提交记录是否已被修改
git log
```

# 3. git设置忽略特殊文件

## 3.1. 忽略文件的原则

1. 忽略操作系统自动生成的文件，比如缩略图等；
2. 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的`.class`文件；
3. 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

## 3.2. 设置的方法

在项目的workdir 下编辑 .gitignore 文件，文件的路径填写为workdir的相对路径。

```bash
.idea/         #IDE的配置文件
_build/
server/server  #二进制文件
```

## 3.3. gitignore 不生效解决方法

原因是.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未track状态），然后再提交：

```bash
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```

# 4. Git分支重命名

假设分支名称为oldName
想要修改为 newName

**1. 本地分支重命名(还没有推送到远程)**

```bash
git branch -m oldName newName
```

**2. 远程分支重命名 (已经推送远程-假设本地分支和远程对应分支名称相同)**
a. 重命名远程分支对应的本地分支

```bash
git branch -m oldName newName
```

b. 删除远程分支

```bash
git push --delete origin oldName
```

c. 上传新命名的本地分支

```bash
git push origin newName
```

d.把修改后的本地分支与远程分支关联

```bash
git branch --set-upstream-to origin/newName
```

# 5. 代码冲突

```bash
git checkout master
git pull
git checkout <branch>
git rebase -i master
fix conflict
git rebase --continue
git push --force -u origin <branch>
```

# 6. 修改历史提交的用户信息

1、克隆并进入你的仓库

```bash
git clone --bare https://github.com/user/repo.git
cd repo.git
```

2、创建以下脚本，例如命名为rename.sh

```bash
#!/bin/sh
 
git filter-branch --env-filter '
OLD_EMAIL="your-old-email@example.com"          #修改参数为你的旧提交邮箱
CORRECT_NAME="Your Correct Name"                #修改参数为你新的用户名
CORRECT_EMAIL="your-correct-email@example.com"  #修改参数为你新的邮箱名
if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

3、执行脚本

```bash
chmod +x rename.sh

sh rename.sh
```

4、查看新 Git 历史有没有错误。

```bash
#可以看到提交记录的用户信息已经修改为新的用户信息
git log 
```

5、确认提交内容，重新提交（可以先把rename.sh移除掉）

```bash
git push --force --tags origin 'refs/heads/*'
```

# 7. 撤销已经push的提交

```bash
# 本地仓库回退到某一版本
git reset -hard <commit-id>
# 强制 PUSH，此时远程分支已经恢复成指定的 commit 了
git push origin master --force
```
