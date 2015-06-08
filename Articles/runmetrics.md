# 运行指标

Linux容器依赖于 [control groups](https://www.kernel.org/doc/Documentation/cgroups/cgroups.txt),不仅可以追踪groups的进程，也能暴露CPU、内存和I/O块的使用情况。你可以管理这些指标，也可以获得网络使用指标。本文讨论纯的LXC容器和Docker容器。

##　Control groups

Control groups被暴露，通过一个虚构的文件系统。在最近的distros，你需要发现这个文件系统在/sys/fs/cgroup下。在那个目录下边，你将看到多个子目录，叫做devices, freezer, blkio等，每个子目录实际上符合一个不同的cgroup分层。

在老的系统，Control groups可能被挂载在/cgroup，没有明显的分层。在那样的情况下，替代可见的子目录，你将看到许多文件在那个目录下，可能某些文件目录相当于存在的容器。

为了弄明白你的Control groups挂载在什么地方，你可以运行：

	$ grep cgroup /proc/mounts

## 列举 Cgroups

你可以在`/proc/cgroups`看到不同的Control group子系统，了解到系统所属于的层次，并知道他们有多少的组。

你可以在`/proc/<pid>/cgroup`看到一个进程属于哪个control group。control group会视一个目录作为其挂载的根目录，例如`/`意味着不会被作为任何个别的group的进程挂载点。当然/lxc/pumpkin意味着那个进程是属于名为pumkin的容器的。


## 找到给定容器的Cgroup

对于每个容器，一个group将被创建在各自的等级区间里。具有老的LXC用户工具的系统版本，cgroup的名称就是容器的名称。随着更多LXC工具版本的更新，cgroup将会是lxc/<container_name>。

当Docker容器使用Cgroups，容器名称将是容器的full ID或者long ID。如果一个容器在docker ps命令下显示ae836c95b4c3，那么它的long ID可能会是这样的ae836c95b4c3c9e9179e0e91015512da89fdec91612f63cebae57df9a5444c79。你可以更细致的查看它的信息，通过docker inspect 或者 docker ps --no-trunc。

为了和memory metrics和Docker 容器联系在一起，可以把目光转移到`/sys/fs/cgroup/memory/lxc/<longid>/`。

## 对Cgroups的衡量：Memory，CPU，Block IO

对于每个子系统（memory, CPU, and block I/O），你将发现一个或者更多的统计文件。

### Memory的衡量：memory.stat

Memory的衡量可以在Memory cgroup找到。注意到，Memory cgroup添加了一个上限，因为它很细致的计算你主机的内存使用情况。因此，许多发行版选择默认不启用它。通常要启用它，你所有要干的事情无非就是添加一个内核命令行参数：`cgroup_enable=memory swapaccount=1`。

Memory的衡量是被包含在文件memory.stat中的。它可能大致长这个样子：

	cache 11492564992
	rss 1930993664
	mapped_file 306728960
	pgpgin 406632648
	pgpgout 403355412
	swap 0
	pgfault 728281223
	pgmajfault 1724
	inactive_anon 46608384
	active_anon 1884520448
	inactive_file 7003344896
	active_file 4489052160
	unevictable 32768
	hierarchical_memory_limit 9223372036854775807
	hierarchical_memsw_limit 9223372036854775807
	total_cache 11492564992
	total_rss 1930993664
	total_mapped_file 306728960
	total_pgpgin 406632648
	total_pgpgout 403355412
	total_swap 0
	total_pgfault 728281223
	total_pgmajfault 1724
	total_inactive_anon 46608384
	total_active_anon 1884520448
	total_inactive_file 7003344896
	total_active_file 4489052160
	total_unevictable 32768






