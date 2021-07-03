# 常用命令

```shell
# 查看历史提交记录
git log
#  -p 或 --patch ，它会显示每次提交所引入的差异 也可以限制显示的日志条目数量，例如使用 -2 选项来只显示最近的两次提交
git log -p -2
# 每次提交的简略统计信息，可以使用 --stat 选项
git log --stat

# 强制删除文件
git rm -f filename
# 比较文件的不同，即暂存区和工作区的差异

git diff
# 查看仓库当前的状态，显示有变更的文件
git status

# 显示所有远程仓库
git remote -v

# 显示当前分支
git branch
```

## 取消暂存的文件

执行`git add .` 之后，取消某些文件的暂存

```shell
git reset HEAD <file>
```

## 撤消对文件的修改

```shell
git checkout -- <file>
```

```shell
git restore <file>
```

## 从远程仓库中抓取与拉取

如果你使用 `clone` 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。 所以，`git fetch origin` 会抓取克隆（或上一次抓取）后新推送的所有工作。 必须注意 `git fetch` 命令**只会将数据下载到你的本地仓库——它并不会自动合并或修改你当前的工作**。 当准备好时你必须手动将其合并入你的工作。

如果你的当前分支设置了跟踪远程分支， 那么可以用 `git pull` 命令来**自动抓取后合并该远程分支到当前分支**。 这或许是个更加简单舒服的工作流程。默认情况下，`git clone` 命令会自动设置本地 master 分支跟踪克隆的远程仓库的 `master` 分支（或其它名字的默认分支）。 运行 `git pull` 通常**会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支**。

origin ——这是 Git 给你克隆的仓库服务器的默认名字

## 推送到远程仓库

```shell
git push <remote> <branch>
```

只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。 当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。 你必须先抓取他们的工作并将其合并进你的工作后才能推送

## 查看命令

查看用户名：

```shell
git config user.name
```

查看用户邮箱：

```shell
git config user.email
```

## 修改命令

修改用户名：

```shell
git config --global user.name "Your_username"
```

修改用户邮箱：

```shell
git config --global user.email "Your_email"
```

注：user.name或者user.email后需有一个空格，再写你的用户名或者用户邮箱

# 创建本地仓库，推送到github

```bash
git init // 初始化一个仓库 
git add -A // 添加所有文件到暂存区，也就是交给由git管理着 
git commit -m "myblogs first commit" // 提交到git仓库，-m后面是注释 
git remote add origin https://github.com/Corefo/myblogs.git 
git push -u origin master // 推送到远程myblogs仓库
```



# 已有代码基础上开发和提交代码

1. 通过Ones系统创建代码分支，选择合理的服务，分支类型和有业务含义的分支名创建分支

   - feature: 日常业务，技术任务迭代
   - Hotfix: 用于线上紧急故障时的修复分支
   - Bugfix: 用于修复bug的分支

2. 创建专门用于开发的文件夹

3. 初始化git仓库

   ```shell
   git init
   ```

   

4. 添加远程仓库

   ```shell
   git remote add origin url
   ```

   

5. 通过`git fetch`命令拉取创建的新分支，通过`checkout`命令切换分支后，进行开发工作

   ```shell
   git fetch origin feature/xxx
   git checkout feature/xxx
   ```

6. track代码，并且完成commit，并向远端提交

   ```shell
   git add -A
   git commit -m ""
   git push origin HEAD:featue/xxx
   ```

   

# 合并分支

1. 进入要合并的分支（如开发分支合并到master，则进入master分支）

   ```shell
   git checkout master
   git pull
   ```

2. 查看所有分支是否都pull下来了

   ```shell
   git branch -a
   ```

3. 使用merge合并开发分支

   ```shell
   git merge 分支名
   ```

4. 查看合并后的状态

   ```shell
   git status
   ```

5. 有冲突的话，通过IDE解决冲突

6. 解决冲突之后，将冲突文件提交暂存区

   ```shell
   git add 冲突文件
   ```

7. 提交merge之后的结果

   ```shell
   git commit
   ```

   如果不使用git commit -m "备注"，那么git会自动将合并的结果作为备注，提交本地仓库

8. 本地仓库代码提交远程仓库

   ```shell
   git push origin HEAD:远程分支名
   ```


# git revert

git revert 撤销 某次操作，此次操作之前和之后的commit和history都会保留，并且把这次撤销
作为一次最新的提交

```shell
# 撤销前一次 commit
git revert HEAD    
# 撤销前前一次 commit
git revert HEAD^        
```

  ## git revert 和 git reset的区别 
-  git revert是用一次新的commit来回滚之前的commit，git reset是直接删除指定的commit。 
- 在回滚这一操作上看，效果差不多。但是在日后继续merge以前的老版本时有区别。因为git revert是用一次逆向的commit“中和”之前的提交，因此日后合并老的branch时，导致这部分改变不会再次出现，但是git reset是直接把某些commit在某个branch上删除，因而和老的branch再次merge时，这些被回滚的commit应该还会被引入。 
- git reset 是把HEAD向后移动了一下，而git revert是HEAD继续前进，只是新的commit的内容和要revert的内容正好相反，能够抵消要被revert的内容。

## 删除上一次提交，git log里不显示此次提交

```shell
git revert ceef575
git reset --hard HEAD^
```






