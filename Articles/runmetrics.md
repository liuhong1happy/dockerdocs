# 运行指标

Linux容器依赖于 [control groups](https://www.kernel.org/doc/Documentation/cgroups/cgroups.txt),不仅可以追踪groups的进程，也能暴露CPU、内存和I/O块的使用情况。你可以管理这些指标，也可以获得网络使用指标。本文讨论纯的LXC容器和Docker容器。

##　Control groups

Control groups被暴露，通过一个虚构的文件系统。在最近的distros，你需要发现这个文件系统在/sys/fs/cgroup下。在那个目录下边，你将看到多个子目录，叫做devices, freezer, blkio等，每个子目录实际上符合一个不同的cgroup分层。

在老的系统，Control groups可能被挂载在/cgroup，没有明显的分层。在那样的情况下，替代可见的子目录，你将看到许多文件在那个目录下，可能某些文件目录相当于存在的容器。

为了弄明白你的Control groups挂载在什么地方，你可以运行：

	$ grep cgroup /proc/mounts

