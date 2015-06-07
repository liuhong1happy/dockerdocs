## Command Completion

Compose与bash shell下的command completion 是分不开的。

安装Command Completion

确保bash Completion已安装。如果当前使用的是非最简化安装的linux，那么应该bash Completion已经可以使用。Mac中可以使用brew install bash-completion 安装bash Completion。

把安装脚本放在/etc/bash_completion.d/ (mac路径/usr/local/etc/bash_completion.d/ )，例如 

	curl -L https://raw.githubusercontent.com/docker/compose/1.2.0/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose

下次登录时Completion就能使用了。

可用的completion取决于命令行输入非内容，完整的包括：

- 可用的 docker-compose 命令
- 特殊命令的可用选项
- 特定上下文中有意义的服务(如，运行中或已停止的服务实例或者基于镜像的服务与基于Dockerfile的 服务。 ). 在docker-compose 尺度上, 完全的服务名字将会自动添加 "="。
- 已选项的参数如 docker-compose kill -s 会完像SIGHUP 和IGUSR1的成信号传递

更加高效低错的享受Compose吧!