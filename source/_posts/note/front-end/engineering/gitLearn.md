---
title: git命令详解
link: gitLearn
catalog: true
date: 2023-11-16 17:39:47
tags:
  - 前端
  - 前端工程化
  - git
categories:
  - [笔记, 前端, 前端工程化]
---

# 创建本地 git 仓库

`git init`会在当前目录下创建一个.git 隐藏文件夹

# 将本地仓库和远程仓库相关联

1. `git remote add origin <registry-url>` 将本地仓库和远程仓库相关联
2. `git remote -v` 查看关联的远程仓库

# 将本地对应的代码提交到暂存区：

1. `git add <file> `将指定的 file 提交到暂存区
2. `git add . `将所有有变动的文件提交到暂存区

# 将暂存区的代码提交到本地 git 仓库：

`git commit -m '<commit-msg>'`

# 查看文件状态

`git status` 可以查看哪些文件被修改了，哪些文件提交到暂存区了但是还没有 commit

# 提交本地 git 仓库代码到远程仓库:

1. `git push` 会将当前分支的最新的 commit 提交到远程仓库对应的分支，然后本地的对应的远程分支(`remotes/origin/对应当前分支名`)也会自动更新
2. `git push origin <source> `会将本地的`<source>分支`的最新 commit 提交到远程仓库对应的分支，然后本地的对应的远程分支（`remotes/origin/<source>`）也会自动更新
3. `git push origin <source>:<destination>` 会将本地的`<source>分支`的最新 commit 提交到远程仓库的`<destination>分支` 然后本地对应的远程分支(`remotes/origin<destination>`)也会自动更新。 如果`<destination>分支`不存在，会在远程仓库自动创建`<destination>分支`，然后在本地创建对应的远程分支(`remotes/origin/<destination>`)并更新
4. `git push origin :<destination> `会在远程仓库直接删除`<destination>` (个人感觉这样设计是因为 push 了一个空给`<destination>`，所以 git 就理解为你要删除`<destination>`)

# 拉取远程仓库代码：

1. `git fetch` 会拉取远程仓库的++所有分支++各自对应的最新代码 将远程仓库所有的分支各自的最新的 commit 添加到对应的本地的各个远程分支（`remotes/origin/\*`） ++但是不会合并分支++。++也就是说 只需一次命令 就可以将远程仓库的所有的最新更新给拉下来++
2. `git fetch origin <source>` 拉取远程仓库的`<source>分支`的最新 commit 然后添加到本地对应的远程分支（`remotes/origin/source`）但是不会合并分支
3. `git fetch origin <source>:<destination> `拉取远程仓库的`<source>分支`的最新 commit 然后添加到本地的`<destination>分支` 但是不会合并分支。如果`<destination>`不存在，会在本地以当前分支为基本自动创建`<destination>`。
   `git fetch origin :<destination>` 会在本地新建一个`<destination>分支` （感觉这样设计是因为，相当于 fetch 了一个空到本地，所以 git 就会理解为你要新建一个分支）

# 拉取远程仓库代码，并和本地的分支做一个合并：

1. `git pull` 其实就是 `git fetch` 和 `git merge` 的缩写，在 `git fetch` 的基础上 会将远程分支（`remotes/origin/对应当前分支名`）和本地当前分支做一个合并
2. `git pull origin <source>` 会拉取远程仓库的`<source>分支`的最新 commit,然后添加到对应的本地的远程分支上面(`remotes/origin/<source>`)，再将这个远程分支和本地当前分支做一个合并
3. `git pull origin <source>:<destination>` 会将远程仓库的`<source>分支`的最新的 commit 添加到本地的`<destination>`分支上面（如果`<destination>`不存在，会自动创建），然后将`<destination>`合并到当前分支。

# 创建分支：

`git branch <branch-name>`

# 查看分支：

1. `git branch` 查看本地分支
2. `git branch -r` 查看远程分支
3. `git branch -a` 查看所有分支

# 删除分支：

1. `git branch -d <branch-name>` 当被删除分支有新内容没有被合并的时候,使用-d，会提示该分支有新内容没有被合并，不执行删除。
2. `git branch -D <branch-name>` 当被删除分支有新内容没有被合并的时候，使用-D，会直接删除

# 切换分支:

`git checkout <branch-name>`

# 创建并切换分支:

`git checkout -b <branch-name>`

# 将分支移动到指定 commit：

`git branch -f <branch-name> <commit-hash>`
以++相对移动++的方式将分支移动到指定 commit
`git branch -f <branch-name> HEAD{^[num], ~[num]}`

# 代码回滚

1. `git reset --mixed <commit-hash>` `git reset <commit-hash>`默认就是这个命令，将++暂存区， 本地 git 仓库++回滚到指定 commit
2. `git reset --hard <commit-hash>` 将++本地代码，暂存区，本地 git 仓库++回滚到指定 commit
3. `git reset --soft <commit-hash>` 将++本地 git 仓库++回滚到指定 commit
4. `git revert <commit-hash> `会在当前分支新添加一个 commit 这个 commit 的作用是抵消之前的对应的 commit，也可以用于回滚分支。
   ![ndPCv.png](https://i.imgs.ovh/2023/11/16/ndPCv.png)
   :::info
   从++数据安全++上角度，`revert` 比 `reset` 安全，因为它的操作可以回溯，反转了还可以倒回来。`reset `比较彻底，是直接丢弃了,不过可以考虑想第一个例子中创建一个备份分支来保证安全。
   从++分支历史++的长期维护角度，`reset` 的历史比较干净，`revert `的反转提交没多大意义，毕竟很少有需求让你滚来滚去的。
   在被撤销提交，不在分支顶端的场景上，`reset` 无法使用，`revert` 可以做到,。
   :::

# 合并分支：

1. `git merge <branch-name>` 将分支合并到当前分支，会在当前分支新增一个 commit（用来合并需要合并的分支）并且当前分支会自动更新 ++不是线性的++
2. `git rebase <target-branch-name>` 将当前分支有的，但是目标分支没有的 commit 直接线性的添加到目标分支 但是目标分支不会自动更新 ++是线性的++
   ![gitlearn01.png](https://i.imgs.ovh/2023/11/16/nYGdI.png)

# 合并指定的 commit：

1. `git check-chery <commit-hash>` 将指定的 commit 添加到当前分支 可以一次添加多个 `commit`

# 切换 HEAD:

1. `git checkout <commit hash>`
2. `git checkout HEAD{^[num], ~[num]} `注：^后面的 num 指的是切换到第几个 parent commit(横向的) ~后面的 num 是指以当前 HEAD 为参考，切换到上面第几个 commit

# 查看 HEAD 指针的移动记录

`git reflog`

# 查看分支历史

1. `git log` 显示 commit 的 SHA1 值，创建作者和时间，提交信息, 会多行显示
2. `git log --oneline` 只显示提交的 SHA1 值和提交信息，SHA1 还是缩短显示前几位，只在一行显示

# 推荐一个特别好的 git 的交互式学习网站

https://learngitbranching.js.org/
