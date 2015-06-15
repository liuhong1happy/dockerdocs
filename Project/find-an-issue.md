# 找出一个问题，并修改提交

## 查找并发布这个问题 ##

本节中，你要学会找到一个问题点，并解决它。作为一个贡献人员，你可以做你想做的事。如果是一个新手，你可以从我们已知的一个问题着手。

## 了解问题的类型 ##

已经存在的问题大多是 docker 用户提交的，我们的维护人员会将它们分类。现在，将用户提交的问题按照困难度分类是你首要的任务。

主要有如下几种分类：

![Images/6630719220513025143.png](Images/6630719220513025143.png)

作为表的状态，这些标签仅用作指导。你也许在你的个人项目中已经为 docker 写好了一个插件但你从没有将它贡献到 docker 社区，对于这种情况，你可以将它归为 exp/expert 或 exp/master 一类。

## 发布一个新手或初学者问题 ##

1. 进入 docker/docker [代码库](https://github.com/docker/docker)

2. 点击 “Issues”链接

会显示如下的问题列表

![Images/issue_list.png](Images/issue_list.png)

3. 在列表中寻找带有 `exp/beginner` 这样的标签的问题

4. 在 “labels”的下拉菜单中选择 `exp/beginner` 这个标签，系统就会只显示 `exp/beginner` 这个标签

5. 打开一个你比较感兴趣的问题，下面会有一些关于这个问题的评论，它会告诉你具体的问题和潜在的解决方案

6. 确保没有其他用户关注这个问题，我们不允许外部的贡献者将问题分配给他们自己，因此，你需要阅读评论，并通过评论中的 #dibs 来查找是否已经有用户提出了这个问题

7. 当你觉得这个问题你很感兴趣并且之前没有提出过，你可以在评论之前加一个 #dibs 

![Images/easy_issue.png](Images/easy_issue.png)


这个例子中问题的编号是 11038，你提交的问题的编号会和这个不同。

8. 记住你的问题的编号，一会你将需要它

## 同步fork的代码，再创建一个分支 ##

如果你是按照上面的步骤来的话，你现在已经 fork 了一个代码库了，在你进行以下步骤时，你需要和 docker/docker 分支进行同步，来确保你的分支是最新版本的。

接下来，我们先同步代码：

1. 打开 终端

2. 切换至 `docker-fork` 的根目录

	$ cd ~/repos/docker-fork

3. 校验（checkout）master 分支

	$ git checkout master
	
	Switched to branch 'master'
	
	Your branch is up-to-date with'origin/master'.

4. 确保有 upstream 来来连接到远程的 docker/docker 代码库

	$ git remote -v 

	origin  https://github.com/moxiegirl/docker.git (fetch)
	
	origin  https://github.com/moxiegirl/docker.git (push)
	
	upstream    https://github.com/docker/docker.git (fetch)
	
	upstream    https://github.com/docker/docker.git (

如果缺少upstream，你需要手动添加上去

	$ git remote add upstream https://github.com/docker/docker.git

5. 从 upstream/master 这个分支上获取所有的修改

	$ git fetch upstream
	
	remote:Counting objects:141,done.
	
	remote:Compressing objects:100%(29/29),done.
	
	remote:Total141(delta 52), reused 46(delta 46), pack-reused 66
	
	Receiving objects:100%(141/141),112.43KiB|0 bytes/s,done.
	
	Resolving deltas:100%(79/79),done.
	
	From github.com:docker/docker
	
	        9ffdf1e..01d09e4  docs       -> upstream/docs
	
	        05ba127..ac2521b  master     -> upstream/master

这条命令是说从 master 分支获取更新所有修改过的地方

6. 重写（rebase）本地的 upstream/master

	$ git rebase upstream/master
	
	First, rewinding head to replay your work on top of it...
	
	Fast-forwarded master to upstream/master.

7. 再来检查一下本地的分支

	$ git status
	
	On branch master
	
	Your branch is ahead of 'origin/master'by38 commits.
	
	    (use"git push" to publish your local commits)
	
	nothing to commit, working directory clean

你的本地库现在已经有了 upstream 远程修改过的文件，你需要将修改过后的代码推送（push）到 origin/master 这个分支

8. 将本地代码推送到 `origin/master` 分支

	$ git push origin
	
	Usernamefor'https://github.com': moxiegirl
	
	Passwordfor'https://moxiegirl@github.com':
	
	Counting objects:223,done.
	
	Compressing objects:100%(38/38),done.
	
	Writing objects:100%(69/69),8.76KiB|0 bytes/s,done.
	
	Total69(delta 53), reused 47(delta 31)
	
	To https://github.com/moxiegirl/docker.git
	
	    8e107a9..5035fa1  master -> master

9. 创建一个有关你的问题的新的分支

这个分支的命名应该有如下规则：XXXX-descriptive  其中 XXXX 代表上面提到的问题编号，比如：

	$ git checkout -b 11038-fix-rhel-link
	
	Switched to a new branch '11038-fix-rhel-link'

    你的分支应该和 upstream/master 保持最新，因为你的分支是从 master分支上分出来的。

10. 再从 upstream/master 上重写（rebase）刚才新建的分支
	
	$ git rebase upstream/master
	
	Current branch 11038-fix-rhel-link is up to date.

ok，现在你的本地分支，远程代码库，和docker 代码库都已经同步完成，且都是最新的。

## 下一节 ##

[如何完成你的修改](work-issue.md)

