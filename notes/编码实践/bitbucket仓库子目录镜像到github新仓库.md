# bitbucket仓库子目录镜像到github新仓库

背景：项目需要通过 CI 将 bitbucket 工程下的 demo 目录镜像到 github 上

* bitbucket 有插件支持直接镜像，不过不支持子目录的镜像。

* Travis CI 不支持 bitbucket 到 github

## 首先如何拆分出子文件夹

[GitHub官方文档中有相关操作](https://help.github.com/cn/github/using-git/splitting-a-subfolder-out-into-a-new-repository)

* **打开终端**

* **将当前工作目录切换到要创建新仓库的位置**

* **克隆包含子目录的仓库**
```shell
git clone https://github.com/USERNAME/REPOSITORY-NAME
```

* **将当前工作目录更改为您克隆的仓库。**
```shell
cd REPOSITORY-NAME
```

* **要从仓库中的其余文件过滤出该子文件夹，请运行 [git filter-branch](https://git-scm.com/docs/git-filter-branch)，提供以下信息：**

  * **FOLDER-NAME**：项目中您要从其创建单独仓库的文件夹。

  * **BRANCH-NAME**：当前项目的默认分支，例如 master 或 gh-pages。

```shell
git filter-branch --prune-empty --subdirectory-filter FOLDER-NAME  BRANCH-NAME
  # Filter the specified branch in your directory and remove empty commits
  > Rewrite 48dc599c80e20527ed902928085e7861e6b3cbe6 (89/89)
  > Ref 'refs/heads/BRANCH-NAME' was rewritten
```

现在，该仓库应仅包含您的子文件夹中的文件。

* **检查仓库现有的远程名称。 例如，源仓库或上游仓库是两种常见选择。**
```shell
git remote -v
> origin  https://github.com/USERNAME/REPOSITORY-NAME.git (fetch)
> origin  https://github.com/USERNAME/REPOSITORY-NAME.git (push)
```

* **使用现有的远程名称和远程仓库 URL 为新仓库设置新的远程 URL。**
```shell
git remote set-url origin https://github.com/USERNAME/NEW-REPOSITORY-NAME.git
```

* **使用新仓库名称验证远程 URL 是否已更改。**
```shell
git remote -v
# 验证新远程 URL
> origin  https://github.com/USERNAME/NEW-REPOSITORY-NAME.git (fetch)
> origin  https://github.com/USERNAME/NEW-REPOSITORY-NAME.git (push)
```

* **将您的更改推送到 GitHub 上的新仓库。**
```shell
git push -u origin BRANCH-NAME
```

## 子目录推送到新仓库
* **git remote add**
```shell
git remote     //查看远程仓库名字
git remote -v  //查看远程仓库链接
```
查看远程库的的情况，可以看到只有一个叫origin的远程库以及远程库fetch和push链接
```shell
origin	git@github.com:believeszw/server.git (fetch)
origin	git@github.com:believeszw/server.git (push)
```
接着使用如下命令添加一个远程仓库（这里以coding为例）
```shell
git remote add coding git@github.com:believeszw/test.git   //添加一个远程仓库叫coding后面是这个远程仓库的地址
```
然后用如下命令：
```shell
git remote     //查看远程仓库名字
git remote -v  //查看远程仓库链接
```
可以查看是否添加成功，成功所示：
```shell
coding	git@github.com:believeszw/test.git (fetch)
coding	git@github.com:believeszw/test.git (push)
origin	git@github.com:believeszw/server.git (fetch)
origin	git@github.com:believeszw/server.git (push)
```
然后再使用相应的命令 push 到对应的仓库就行了，例如：
```shell
git push origin master   //这里是push到远程主机origin上的对应master分支
$ git push coding master //这里是push到远程主机coding上的对应master分支
```
这种方法的缺点是每次要push两次,因为是两个不一样的远程库名。

#### **注意**
如果当前分支和 push 的远程分支名不同可以尝试使用
```shell
git push coding HEAD:master
```

* **使用 git remote set-url 命令**

使用git remote rm coding命令删除方法一中的coding远程仓库，检查是否删除成功用前面列出的查看远程仓库命令

接着使用如下命令添加一个远程仓库（这里添加相同名称的库为origin）
```shell
git remote set-url --add origin git@github.com:believeszw/test.git
```
查看远程仓库情况，可以看到远程主机仓库列表有两个 push 地址。这种方法的好处是每次只需要 push 一次就行了,如下:
```shell
origin	git@github.com:believeszw/server.git (fetch)
origin	git@github.com:believeszw/server.git (push)
origin	git@github.com:believeszw/test.git (push)
```
* **修改配置文件.git/config**

打开本地项目中 .git/config 文件找到 [remote “origin”]（这里根据个人的实际显示），添加对应的 url 即可。这种方法其实和方法二是一样的，好处也是和方法二一样每次只需要 push 一次就行了,如下图：
```shell
[remote "origin"]
     url = git@github.com:believeszw/server.git
     fetch = +refs/heads/*:refs/remotes/origin/*
     url = git@github.com:believeszw/test.git
```

## 总结
方法二和方法三在 push 的时候比较方便。但是在 pull 的时候只能从方法三图中的第一个 url 地址拉取代码。而方法一则不存在这种问题（可能要解决冲突）。所以，如果只进行 push 操作，推荐方法二和方法三，如果也要进行 pull 操作，推荐方法一。

项目中 CI 用到的脚本
```shell
git checkout -b rtsa_demo
git filter-branch -f --prune-empty --subdirectory-filter app/agora_sdk_demo rtsa_demo
git remote add test git@github.com:believeszw/test.git
git push test HEAD:master
git remote rm test
git reset --hard origin/HEAD
git checkout arsenal
git branch -D rtsa_demo
```
