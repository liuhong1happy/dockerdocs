# 在哪可以讨论或寻求帮助？ #

Docker有许多交流的频道，供你和docker用户及开发者一起讨论

## Internet Relay Chat （IRC） ##

IRC频道是直接通向docker用户的，在irc.freenode.net.上面有#docker及#docker-dev群。IRC是在1988年创建的，它是一个丰富的聊天协议，但是可以覆盖掉新用户，你可以搜索我们的聊天存档。

你可以在下面看到我们的IRC快速指南

## Google Groups ##

这里面会有两个群，一个是Docker-user，专为docker使用者创建的，另一个是docker-dev专为docker的贡献者和开发人员创建的

## Twitter ##

你可以关注Docker在Twitter上的账号来了解docker，你也可以向我们提一些关于docker的问题或分享有关docker的博客

## Stack Overflow ##

Stack Overflow 已经有7000K多关于docker的提问，我们会定期的查看这些问题并解决这些问题

## IRC快速指南 ##

如何以最简单的方式使用IRC

1.    在浏览器中打开 [http://webchat.freenode.net](http://webchat.freenode.net/)

![Images](../Images/irc_connect.png)

2.    填写下面的表单

NickName：IRC上你的账户名称

Channels：#docker

reCAPTCHA（验证码）：填写你看到的验证码

3.    点击“Connect”按钮

进去之后你会看到一大推的文字，底部会有一个命令行，在命令行的上面系统会提示你注册


![](../Images/irc_after_login.png)


4.    在命令行中注册

    	/msg NickServ REGISTER password youremail@example.com

![](../Images/register_nic.png)

IRC系统会向你刚才输入的地址发一封邮件，邮件会让你完成更详细的注册

5.    打开你的邮箱查看邮件

    
![](../Images/register_email.png)


6.    回到浏览器，根据邮件的提示完成注册

		/msg NickServ VERIFY REGISTER moxiegirl_ acljtppywjnr

7.    使用如下命令加入到 #docker 群

		/j #docker

你也可以加入 #docker 群

		/j #docker-dev

8.    通过命令行向频道发一些问题

![](../Images/irc_chat.png)

9.    关闭浏览器，退出

## Tips和了解更多关于IRC的知识 ##

下次，你再登陆IRC时，你需要在命令行使用如下的命令再次输入你的密码

		/msg NickServ identify <password>

如果你忘记了密码，请查看：[the FAQ on freenode.net](https://freenode.net/faq.shtml#sendpass)

这篇快速指南仅简单讲了如何使用IRC，如果你觉得IRC对你有用的话，你可以更深入的学习。另一个开源项目 Drupal已经写了一篇关于[IRC更详细的文档](https://www.drupal.org/irc/setting-up)。（非常感谢Drupal）