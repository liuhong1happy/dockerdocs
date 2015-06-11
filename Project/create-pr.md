# 创建一个 Pull 请求(PR) #

Pull 请求（PR）会提交你的修改，来让docker 的维护者审查。当你在 GitHub上创建一个Pull起来后，这个请求会将你fork的代码库中修改的地方提交到 docker/docker 代码库。

你可以在 GitHub上[查看所有pull请求的列表](https://github.com/docker/docker/pulls)

## Pull之前的检查 ##

1.    在终端中，进入 docker-fork 的根目录

		$ cd ~/repos/docker-fork

2.    检查（checkout）你的特性分支

		$ git checkout 11038-fix-rhel-link
	
		Already on '11038-fix-rhel-link'

3.    在你的分支中运行整个测试套件

		$ make test

这个测试中，所有的测试都必须通过

4.    如果你修改了文档，你也可以重新构建文档，但这不是必须的

		$ make docs

5.当你检查完之后，你就可以提交了

## 重写（Rebase）分支 ##

在你提交之前你还必须要重写你的代码

1.    先于docker/docker同步

		$ git fetch upstream master
		
		From github.com:docker/docker
		
		    * branch            master     -> FETCH_HEAD

2.    启动一个交互式的rebase操作

		$ git rebase -i upstream/master

3.    rebase操作会显示如下的提交列表

		pick 1a79f55Tweak some of the other text for grammar
		
		pick 53e4983Fix a link
		
		pick 3ce07bbAdd a new line about RHEL
一旦你哪个地方改错了，你可以使用 git --rebase abort 命令撤销所有的修改

4.    将除第一个 pick 关键字之外的其他 pick 关键字替换成 squash

		pick 1a79f55Tweak some of the other text for grammar
		
		squash 53e4983Fix a link
		
		squash 3ce07bbAdd a new line about RHEL

当关闭文件后，git 会重新打开编辑器并编辑提交信息

5.    编辑并保存你要提交的信息
    
    	`git commit -s`
    
    	Make sure your message includes <a href="./set-up-git" target="_blank>your signature</a>.

6.    push 代码

		$ git push origin 11038-fix-rhel-link

## 在GitHub上创建一个Pull请求（PR） ##

1. 登陆GitHub并选择fork的代码库

你会看到在你的分支上最新的活动

![Images/latest_commits.png](Images/latest_commits.png)

2.  点击 “Compare & pull request”按钮

系统会展示Pull请求的全部对话框

![Images/to_from_pr.png](Images/to_from_pr.png)

这会将你的分支和 docker/docker 代码库中的master分支进行比较

3.  编辑对话框中的描述信息，并对你的问题添加一些参考

你可以在GitHub上搜索一些问题来给你的问题提供一些参考

![Images/fixes_num.png](Images/fixes_num.png)

4.  鼠标线下滚动，查看 Pull请求（PR）中有没有包含你提交的内容

比如，文件数量是不是正确？是不是都有你修改的地方？

![commits_expected.png](Images/commits_expected.png)

5.  点击 “Create pull request”

系统就会创建一个请求，然后再 docker/docker 代码库中打开你的请求

![pull_request_made.png](Images/pull_request_made.png)

## 下一节 ##

[加入到 Pull请求的审查中](review-pr.md)