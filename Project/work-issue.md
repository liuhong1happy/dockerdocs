#解决你关注的问题#

本节将会带你一步一步的了解工作流程，同时还会给你一些命令实例。

不过这只是一个大概的流程，也许和你关注的问题会略有不同，当然你花费在上面的时间也会不同。

## 如何完成你的本地分支 ##

步骤如下：

1.    查看合适的样式指南

如果你修改了代码，请查看[编码规范](coding-style.md)，如果修改的是文档，请查看[文档格式规范](doc-style.md)

2.    对你的特性分支进行修改

这个特性分支是在上一节中创建的，在这里你是用的是开发容器。如果你对代码进行了修改，你可以将代码挂载到开发容器中使用。如果只是修改了文档，你可以在本机访问。

你不能修改 vendor 目录及其子目录里的内容，它们包含了第三方的依赖代码，如果你忘记的话，请查看这篇文章：[使用一个开发容器](set-up-dev-env.md)

3.    测试你修改的部分


如果你是按照这篇指南做的话，你就会知道 make test 目标运行了整个测试套件并且make docs 构建了这个文档。如果你忘记了其它的测试目标，请查看这篇文章：[运行测试程序并测试文档](test-and-docs.md)

4.    如果修改了代码，必要的发可以添加单元测试

如果你添加了一个新的功能或是修改了现有的功能，你就要添加一个单元测试。你可以从现有的测试文件中找到一些灵感，如果你不确定是否要测试，你可跳过这个步骤，如果以后必要的话你可以加进去。

5.    格式化你的源文件

![](Images/2406048100940639758.png)

6.    列出你修改的地方

	$ git status
	
	On branch 11038-fix-rhel-link
	
	Changesnot staged for commit:
	
	    (use"git add <file>..." to update what will be committed)
	
	    (use"git checkout -- <file>..." to discard changes in working directory)
	
	
	
	modified:   docs/sources/installation/mac.md
	
	modified:   docs/sources/installation/rhel.md

    status 命令会显示你修改过的所有文件

7.    将你修改过的文件添加到 Git

	$ git add docs/sources/installation/mac.md
	
	$ git add docs/sources/installation/rhel.md

8.    提交你的修改并用 -s 指出你修改的内容

	$ git push origin
	
	Usernamefor'https://github.com': moxiegirl
	
	Passwordfor'https://moxiegirl@github.com':
	
	Counting objects:60,done.
	
	Compressing objects:100%(7/7),done.
	
	Writing objects:100%(7/7),582 bytes |0 bytes/s,done.
	
	Total7(delta 6), reused 0(delta 0)
	
	To https://github.com/moxiegirl/docker.git
	
	*[new branch]11038-fix-rhel-link ->11038-fix-rhel-link
	
	Branch11038-fix-rhel-link set up to track remote branch 11038-fix-rhel-link from origin.

你第一次提交的时候，必须指明是哪个分支，下回你只需使用下面的命令就可以了：

	$ git push origin

## 在GitHub上查看你的分支 ##

当你提交了一个新的分支后，你可以在[GitHub](https://github.com/)上查看

1.    登陆 GitHub

2.    转到 Docker fork

3.    选择你的分支

![Images/locate_branch.png](Images/locate_branch.png)

4.    点击 “Compare”按钮，将你的分支和 master 分支进行对比

如果你一直使用的是你的分支，那么你的分支的版本可能会落后于上游的docker 版本库

5.    审查提交

确保你的分支只显示你修改的地方

## 频繁的pull和rebase ##

当你在使用的时候你应该频繁的pull 和 rebase

1.    进入终端

2.    检查是否处于本地的分钟中

		$ git branch 11038-fix-rhel-link

3.    从 upstream/master 分支获取更新

		$ git fetch upstream/master

4.    将 Docker upstream/master 分支重写（rebase）到你的主机

		$ git rebase -i upstream/master

这条命令会将 Docker 的 upstream/master 分支上的代码合并到你的分支，如果你不熟悉 rebase 操作的话，可以查看这篇文章： [learn more about rebasing](http://nathanleclaire.com/blog/2014/09/14/dont-be-scared-of-git-rebase)

5.    Rebase 操作会打开编辑器并显示提交的列表

	    pick 1a79f55Tweak some of the other text for grammar 
	
	    pick 53e4983Fix a link 
	
	    pick 3ce07bbAdd a new line about RHEL

如果遇到错误，git --rebase abort 命令会撤销所有的修改

6.    将编辑器中除了第一个以外的所有 pick 关键字替换成 squash

	    pick 1a79f55Tweak some of the other text for grammar
	
	    squash 53e4983Fix a link
	
	    squash 3ce07bbAdd a new line about RHEL

7.    编辑完成之后保存提交的信息

        确保已经包括了你的签名

8.    push 到你的代码库

		$ git push origin 11038-fix-rhel-link

## 下一节 ##

[学习如何创建一个pull 请求](create-pr.md)