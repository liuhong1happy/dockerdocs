# 加入到PULL请求（PR）的审查中 #

当创建完一个PULL请求后，你离成功贡献已经不远了。现在，你提交的代码会被持续集成系统（continuous integration 简称 CI）和维护人员测试。

CI是一个自动的系统，你必须了解机器和人工是如何审查你的代码的。

## 如何审查你的代码 ##

首先审查的是Gordon，他会审查你的代码是否有普通的错误比如：缺少标点符号之类的。一旦Gordon 发现错误，他会发邮件到你注册GitHub时用到的邮箱。

![](../Images/gordon.jpeg)

构建系统会构建你的代码，同时Gordon会发送相关的邮件给你。

构建系统会通过使用Docker的master代码编译你的代码，以此来多次检查你的代码。构建过程会包括执行你在本地运行的测试样例。如果你忘记运行测试用例或修改漏洞，自动构建程序会检查出来。

当Gordon和持续集成系统检查完毕后，我们的维护人员会审查你的代码，维护人员会查看的pull请求并在上面发表评论，你也许会看到 LGTM 这样的评论，它的意思是 looks-good-to-me， 如果你收到 LGTM 这样的评论，就表示你通过了审查。

对于一些复杂的代码，维护人员可能会问你一些问题或让你修改你提交的代码。任何维护人员发表的评论都会发送到与你GitHub关联的email上，并且任何参与到你的pull请求的GitHub用户都会收到，参与意味着你创建了一个pull请求或评论了一个pull请求。

我们的维护人员是由经验丰富的docker 用户和开源贡献者组成的，因此，他们会很注重你的时间，并尝试和你一起有效的工作通过他们具体、简明的评论。如果他们让你修改你的代码，改完之后，你必须更新的代码。

## 更新一个现有的PULL请求 ##

1.    在你本机的docker-fork仓库中修改文件

2.    使用 git commit --amend 这个命令提交

	$ git commit --amend

Git会打开你上次提交的信息

3.    调成最后的评论信息来说明你这次修改的内容

		Added a new sentence per Anaud's suggestion
		
		Signed-off-by: Mary Anthony <mary@docker.com>
		
		# Please enter the commit message for your changes. Lines starting
		
		# with '#' will be ignored, and an empty message aborts the commit.
		
		# On branch 11038-fix-rhel-link
		
		# Your branch is up-to-date with 'origin/11038-fix-rhel-link'.
		
		#
		
		# Changes to be committed:
		
		#       modified:   docs/sources/installation/mac.md
		
		#       modified:   docs/sources/installation/rhel.md

4.    push 到你的origin

		$ git push origin

5.    进入 GitHub官网，查看你的pull请求

此时你的代码已经是最新的了

6.    对你的pull请求添加一条评论

当你评论后，GitHub只会提醒参与到pull请求的人。比如：你可以提到你更新了你的pull请求，你的评论会提醒维护人员你已经更新了。

你修改的代码要获得维护那个组件的绝大多数人员的LGTMs 评论，例如：你修改了 docs/ 和 registry/ 的代码，那么 docs/ 和 registry/ 的绝大多数维护人员必须通过你提交的修改，一旦你获得通过，你的代码就会合并到Docker master 分支中去。

## 合并之后 ##

如果你要在Docker 的官方查看被合并的代码，可能要过一段时间，尽管构建master分支是立即的。当master分支每一次合并之后，Docker都会重新构建并更新它的二进制文件。

1. 访问 [https://master.dockerproject.com/](https://master.dockerproject.com/)

2. 查找适合你的系统的二进制文件

3. 下载并运行

你可以在容器中运行这个二进制文件，这会使你的宿主机的环境更干净

4. 你可以访问  [docs.master.dockerproject.com](http://docs.master.dockerproject.com/) 来查看最新的文档

一旦你的Pull 请求验证通过，你就可以从你fork的仓库中删除你的特性分支了。如果比了解，请看这篇文章：[see the GitHub help on deleting branches](https://help.github.com/articles/deleting-unused-branches/)

## 下一节 ##

[学习高级贡献](advanced-contributing.md)