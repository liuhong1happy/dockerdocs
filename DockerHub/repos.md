## Hub仓库

Docker Hub仓库让你和同事、顾客以及docker大社区分享镜像成为可能。如果你在建立自己内部的镜像，在你的守护进程又或者用你的连续集成服务时，你可以把他们放到一个docker的hub仓库里面，然后你可以添加用户和管理账户。

另外，如果你的docker镜像的源代码在GitHub或者bitbucket，您可以使用“自动生成”库，这是由docker服务构建。见自动生成文档来阅读这些服务提供的额外功能。

![repos.png](../Images/repos.png)

你的docker库有大量有用的功能。

### 星级

你可以在返回的库中，把你的知识库用星标记。星级是一种表明你喜欢某仓库的方法。他们也是书签收藏夹的简便方法。

### 评论

你可以通过docker窗社区的留言和维护其他成员互动。如果你发现任何不恰当的评论，你可以将它们进行审查。

### 合作者及其角色

合作者是一个你想访问的私人仓库。一旦被指定，他们可以推到你的仓库。他们不被允许执行任何管理任务，如删除库或改变其状态从私人到公众。

注：合作者不能添加其他合作者。只有拥有的知识库的所有者具有管理访问权限。

你还可以指定更多的合作者的权利（“读”，“写”，或“admin”）的docker组织和团体。对于更多的信息，请参阅帐户文档。

### 私人库

私人知识库可以让你拥有一个包含镜像的存储库，你可以在存储库里面保持自己的图像，或者是你自己的帐户，或者是在一个组织或组中的图像。

在docker私人仓库工作，你将需要添加一个通过添加存储库的链接。你可以免费得到一个专用库docker帐户。如果你需要更多的账户，你可以升级你的docker规划。

一旦私人库创建，你可以推拉的图像和使用docker窗。

注意：您需要在和一个私有存储库中签名并访问工作。

私人知识库就像是公共的。然而，它是不可能浏览或搜索他们的内容在公共登记。它们没有得到相同的方式作为一个公共存储库。

这是可能的，以获得一私个人资料库，您所指定的人（即，合作者）从其设置页面。从那里，您还可以切换到存储库状态（公众到私人，反之亦然）。您需要有一个可用的私有存储库，在您可以做这样一个开关之前打开。如果你没有任何可用的，你可以升级你的docker规划。

### Webhooks

一个是一个HTTP回电webhook由特定的事件触发。你可以使用一个中心仓库webhook通知人，服务，和之后的新形象推到你的库的其他应用（这也会自动生成）。例如，你可以触发一个自动测试或部署，只要图像是可用的。

开始添加webhooks，去中心所需的存储库，然后单击“webhooks”下的“设置”对话框。一个webhook之后才成功推动了所谓的。调用HTTP POST请求的webhook用类似下面的示例JSON有效负载。

例如webhook JSON有效负载：

    {
      "callback_url": "https://registry.hub.docker.com/u/svendowideit/busybox/hook/2141bc0cdec4hebec411i4c1g40242eg110020/",
      "push_data": {
        "images": [
            "27d47432a69bca5f2700e4dff7de0388ed65f9d3fb1ec645e2bc24c223dc1cc3",
            "51a9c7c1f8bb2fa19bcd09789a34e63f35abb80044bc10196e304f6634cc582c",
            ...
        ],
        "pushed_at": 1.417566822e+09,
        "pusher": "svendowideit"
      },
      "repository": {
        "comment_count": 0,
        "date_created": 1.417566665e+09,
        "description": "",
        "full_description": "webhook triggered from a 'docker push'",
        "is_official": false,
        "is_private": false,
        "is_trusted": false,
        "name": "busybox",
        "namespace": "svendowideit",
        "owner": "svendowideit",
        "repo_name": "svendowideit/busybox",
        "repo_url": "https://registry.hub.docker.com/u/svendowideit/busybox/",
        "star_count": 0,
        "status": "Active"
    }

测试，你可以试着向一个HTTP请求的工具requestb.in。

注：docker服务器当前的IP范围162.242.195.64 - 162.242.195.127，所以你可以限制webhooks接受那套IP地址webhook的请求。

### webhook链

允许你调用链webhook链多种服务。例如，您可以使用此引发的部署你的容器只有在它已成功通过测试，然后更新一个单独的更新一旦部署完成。在点击“添加webhook”按钮，只需添加尽可能多的URL是必要的在你的链。

在一个链的第一个webhook将成功推动后调用。随后的网址将被联系后，回调已验证。

### 验证回调

为了验证回调在webhook链，你需要

检索请求中的JSON有效负载的callback_url价值。

发送一个POST请求该URL包含一个有效的JSON的身体。

注：一个链的请求，只会被认为是完整的，一旦最后的回调已经验证。

帮助你调试或者查看webhook结果，查看“历史”在其设置页面可用webhook。

回调的JSON数据

回调数据中的下列参数：

状态（必需）：接受的价值是成功，失败和错误。如果国家不成功，将被打断的webhook链。

描述：一个包含各种信息，将在泊坞枢纽可。最大255个字符。

上下文：一个包含操作上下文的字符串。可以从泊坞枢纽检索。最大100个字符。

target_url：URL的运行结果可以发现。可以在泊坞枢纽检索。

例如回调有效载荷：

    {
      "state": "success",
      "description": "387 tests PASSED",
      "context": "Continuous integration by Acme CI",
      "target_url": "http://ci.acme.com/results/afd339c1c3d27"
    }

### 标记为未上市

通过标记库作为未上市，您可以创建一个公开pullable库不会在集线器或命令行搜索。这允许你有一个有限的释放，但不限制访问任何人，被告知，或猜测的知识库名称。
