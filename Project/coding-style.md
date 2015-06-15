# 关于编码的风格 #

## 编码风格的清单 ##

这个清单会根据你对[代码的贡献](https://docs.docker.com/project/make-a-contribution)和[其他高级的贡献](https://docs.docker.com/project/advanced-contributing)来做一个总结，该清单适用于程序的代码，或文档的代码。

## 修改，提交代码 ##

- fork `docker/docker` 仓库
- 在你fork的仓库中创建一个特性分支，并修改，最后如下命名：`XXXX-something` 其中 XXXX 代表你的问题编号
- 在你提交更改之前执行 `gofmt -s -w file.go` 命令，大多数编辑器都会带有这种自动化的插件
- 更新文档
- 如果是修改一个提交或关闭一个问题，你应该加入以下的信息：`Closes #XXXX` 或 `Fixes #XXXX`，当问题因合并关闭时应有一些提示
- 每次提交后，都要执行测试套件，确保它能通过
- 保持和 docker master 的同步
- 设置你的git签名，确保每次提交时都有签名
- 不要将你自己加到 AUTHORS 文件中，此文件是 Git 的历史自动生成的

## 测试 ##

- 修改完代码后提交你的单元测试
- 利用内置的go测试框架构建
- 从现有的docker测试文件（name_test.go）寻找灵感
- 在提交pull请求时运行[整个的测试套件](https://docs.docker.com/project/test-and-docs)
- 执行 make docs 命令来构建docker文档，然后再本地验证一下
- 使用[在线的语法检查工具](http://www.hemingwayapp.com/)测试你的文档是否清晰，正确和简洁

## PULL请求 ##

 

- pull请求时先将本地代码和docker master 的代码同步并重构，但不要混入多个分支
- pull请求前，运行 `git rebase -i` 和 `git push -f` 命令来执行‘squash ’操作
- 在提交中包含所有的修改过的文档修改，以便以后的恢复和修改操作
- 在你的pull请求中列出每个关联到的问题（#xxx）

## 回复PULL请求的审查 ##

- 如果Docker 的维护者在你的PULL请求中评论：LGTM（looks-good-to-me）表明你的请求已经被接受了
- 代码审查时的一些评论会加入到你的pull请求中。这期间你要和docker的维护者讨论你的pull请求的价值，然后你再根据建议进行修改，再提交
- 将你修改的代码合并到你自己的特性分支，然后推送到你fork的仓库中，这会自动更新到你的正处于开放状态的pull请求
- push之后在最底下添加一条评论，来提醒维护者你的pull请求已经修改了，因为push操作不会自动发送通知
- 你的修改必须要得到维护这个组件的绝大多数人员的同意才能通过。例如：如果你修改了 docs/ 和 registry 的代码，就需要维护 docs/ 和registry/ 这两个组件的绝大多数人员的同意才能通过

## PULL请求通过后，会合并到docker master中 ##

- 合并后，你可以立即从 a master build 中获取
- 如果你只是对文档做了修改， [docs.master.dockerproject.com](http://docs.master.dockerproject.com/) 这个上面可以看到