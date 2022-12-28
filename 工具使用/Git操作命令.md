# [Git操作命令](../README.md)

[TOC]

## 一 常见错误

### 1 查看提交内容

​		如果你用`git commit -a` 提交了一次变化，而你又不确定提交了那些内容，可以用一下命令显示当前HEAD上的最近一次`commit`：

```shell
$ git show
或者
$ git log -n1 -p
```

### 2 修改提交日志

如果你的提交信息`commit message`写错了且这次提交还没有push，可以通过下面的方法来修改提交信息：

~~~sh
$ git commit --amend --only
~~~

上面操作会打开你的默认编辑器，在这里可以编辑信息，也可以用一条命令完成操作：

~~~sh
$ git commit --amend --only -m '提交信息'
~~~

如果你已经push了这次提交，你可以修改这次提交然后强制推送（不推荐操作）。

### 3 提交的用户名和邮箱不对

如果只是单个commit，执行以下命令修改：

~~~sh
$ git commit --amend --author "张三 <123@qq.com>"
~~~

如果需要修改所有历史，参考`git filter-branch`命令。

### 4 从一个Commit中移除一个文件

通过以下命令移除文件：

~~~sh
$ git checkout HEAD^ test.txt
$ git add -A
$ git commit --amend 
~~~

### 5 删除最后一次提交

如果需要删除推送了的提交，可以用以下方法，但是这会不可逆的改变你的历史，也会搞乱那些已经从该仓库拉取了的人的历史记录。简而言之，非必要不要这么操作：

~~~SH
$ git reset HEAD^ --hard
$ git push -f [remote] [branch]
~~~

如果还没有推送到远程，把git重置到你最后一次提交钱的状态就可以了（同时报错暂存的变化）：

~~~sh
$ git reset --soft HEAD@{1}
~~~

### 6 删除任意提交

不推荐操作：

~~~sh
$ git rebase --onto SHA1_OF_BAD_COMMIT^SHA1_OF_BAD_COMMIT
$ git push -f [remote] [branch]
~~~

或者做一个交互式rebase删除那些你想要删除的提交里对应的行。

### 7 做了一次硬重置（hard reset），想找回内容

如果你意外做了git reset --hard，你通常能找回你的提交，因为Git对每个时间都有操作日志，且都会保存几天：

~~~sh
$ git reflog
e861f52d (HEAD -> dev, origin/dev) HEAD@{0}: pull --no-stat -v --progress origin dev: Merge made by the 'ort' strategy.
c24bf776 HEAD@{1}: commit: refactor: 取消掉无用代码
f9cb0fa6 HEAD@{2}: pull --no-stat -v --progress origin dev: Fast-forward
744f5126 HEAD@{3}: commit: feat: 提交代码
a6daeb7c HEAD@{4}: commit: feat: 同步用户实名信息
cf5df21f HEAD@{5}: commit: feat: 显示市场主体类型
fa5c929d HEAD@{6}: pull --no-stat -v --progress origin dev: Fast-forward
d3830d67 HEAD@{7}: commit: feat: 集成实名信息
d83c4e9f HEAD@{8}: commit: polish: 取消通过时间排序
~~~

你将会看到过去提交的列表，和一个重置提交，选择你想要找回的commit的SHA，再重置一次：

~~~sh
$ git reset --hard c24bf776
~~~

操作完成。

## 二 暂存（Staging）

### 1 把暂存的内容添加到上一次的提交

~~~sh
$ git commit --amend
~~~

### 2 想要暂存一个新文件的一部分，而不是整个文件

一般来说，如果你想要暂存一个文件的一部分：

~~~sh
$ git add --patch test.txt
~~~

-p简写 ，这会打开交互模式，你讲能够用s选项来分隔提交内容，然而如果这个文件是新的则会没有这个选项，添加一个新文件时，需要进行以下操作：

~~~sh
$ git add -N test.txt
~~~

然后需要用e选项来手动选择需要添加的行，执行`git diff --cached`将会显示哪些行暂存了。

### 3 把暂存的内容编程未暂存，把未暂存的内容暂存起来

多数情况先，你应该将所有内容编程未暂存，然后再选择要暂存的文件进行commit。但如果需要这么操作，你可以创建一个临时的commit来报错你已经暂存的内容，然后保存你的未暂存的内容并进行stash，然后reset最后一个commit将原本暂存的内容变为未暂存，然后stash pop回来.

~~~sh
$ git commit -m "cm"
$ git add .
$ git stash
$ git reset HEAD^
$ git stash pop --index 0
~~~

注意：这里使用pop仅仅是因为想尽可能保持幂等，加入不加上--Index你会把暂存的文件标记为存储。

## 三 未暂存（Unstaged）的内容

### 1 把未暂存的内容移动到一个新分支

~~~sh
$ git checkout -b new-branch
~~~

### 2 把未暂存的内容移动到已经存在的分支

~~~sh
$ git stash
$ git checkout my-branch
$ git stash pop 
~~~

### 3 丢弃本地未提交的变化

~~~sh
# one commit 
$ git reset --hard HEAD^
# two commit 
$ git reset --hard HEAD^^
# four commit 
$ git reset --hard HEAD~4
# or
$ git checkout -f 
~~~

重置某个特殊文件，可以使用文件名作为参数：

~~~sh
$ git reset filename
~~~

