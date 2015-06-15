# README first

 本章节主要介绍了如何对docker的代码和文档做出自己的贡献。最为一个开源社区，我们会分享一些互动的规则，不过在贡献之前你必须要熟悉[docker社区参考](https://github.com/docker/docker/blob/master/CONTRIBUTING.md#docker-community-guidelines)。

## 参与贡献

Docker这个项目是由Github上的多个代码仓库组成的，出了docker/docker这个仓库之外，还有docker/libcontainer、docker/machine等一些其他的仓库。

不是所有的docker代码仓库都是用Go语言写的，每个代码库都有特定的领域。因此，如果你是一位有经验的贡献者，你在贡献的时候可以选择你擅长的领域和用你熟悉的语言。

如果你是刚接触开源社区，或者docker还是正是的编程，你应该开始对docker/docker仓库做出一些贡献。为什么呢？因为这个指南是专门为这个仓库而写的。

最后，代码或是文档并不是贡献的唯一方式。你也可以在docker的社区频道提交一个问题，加入到一个讨论中，写一篇blog，或是做一个可用性的测试。你甚至可以提出自己的类型的贡献，现在我们对于这方面并没有写太多，如果你感兴趣可以发送邮件至 [feedback@docker.com](mailto:feedback@docker.com) 。

##　这里涉及一个乌龟

[](../Images/gordon.jpeg)
    
好了，已经说的够多了。

How to use this guide

这篇指南只是写给那些容易分心，劳累和马虎且有一点对 git 知识了解但又对 GitHub GUI 不太了解的读者的，本文将尽可能准确，可操作的讲解如何使用docker环境。

docker初学者可以尝试着建立自己的docker环境，然后可以对代码做一些简单的改变，之后，你也许会发现在docker上能找到一些工作或提出新的问题。

如果你是编程天才，你也会发现这篇文档非常有用，当你困惑时可以随时查看以前的文章。

## 怎样开始

首先，获取必要的软件，其平台可以是：[Linux or OS X](software-required.md) 或者 [Windows](software-required-win.md)。