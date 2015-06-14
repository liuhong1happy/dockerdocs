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


前面一半不带total\_前缀的包含cgroup进程相关的统计信息，但是不包含子cgroup。后面一半带有total\_前缀的包含子cgroup的相关统计信息。

某些统计叫做gauges，其值可增可减（例如，swap：cgroup中被使用的可交换空间的数量）。某些统计则是一直增加的（例如，pgfault：当cgroup创建时出错的次数，这个是绝对不会减少的。

* cache：

	cgroup中进程所使用的内存容量，精确地计算在块设备上使用的块容量。当你从磁盘中读写文件时，这个数字是在增加的。如果你按通常方式使用 I/O（open，read，write等系统调用），或者映射文件（mmap），这时就会增加cache。它也是使用tmpfs挂载路径的容量，尽管理由不够清楚。

* rss

	不能和磁盘通信的次数：stacks，heaps和无名的memory maps。

* mapped_file

	在cgroup中映射的内存数量。它不仅告诉你多少内存被使用了，而且还要告诉你怎么使用的。

* pgfault 和 pgmajfault

	cgroup中的进程各自触发page fault和major fault的数量。当进程操作一部分受保护或私有的虚拟内存空间时，page fault将会发生。之前，也会因为进程抽风，或者尝试访问非法地址而发生（这时会发送一个SIGSEGV信号来杀死它，并带有著名的Segmentation fault消息产生）。后来，当进程从内存地址读取，该地址正好被交换共享或者在读取映射的文件：这时，内核将从磁盘加载page，让CPU完成内存操作。当进程写入一个写时拷贝内存空间，也会发生这样的事情：同样的，内核将占有进程，复制内存page，重新开始写操作在进程自己复制的page上。"Major" faults发生时，是因为内核实际不得不从磁盘读取数据。当它不得不复制一张存在的page，或者分配一个空page，这就是所谓的("minor") fault。


* swap

	cgroup中进程当前被使用的交换空间数量。

* active_anon 和 inactive_anon

	被认同的Anonymous内存分别会被内核激活或者非激活。Anonymous内存是不被链接到磁盘page的内存。换句话说，等价于rss所描述的数量。事实上，rss数量是**active_anon + inactive_anon - tmpfs**。那么，激活和非激活之间有什么不同呢？Pages以"active"打头，在相同间隔，它们会被立马重新标记会"active"。当内核差不多内存溢出，内存被交换到磁盘，内核将会转化为"inactive" page。


* active_file and inactive_file:

	cache memory, with active and inactive similar to the anon memory above. The exact formula is cache = active_file + inactive_file + tmpfs. The exact rules used by the kernel to move memory pages between active and inactive sets are different from the ones used for anonymous memory, but the general principle is the same. Note that when the kernel needs to reclaim memory, it is cheaper to reclaim a clean (=non modified) page from this pool, since it can be reclaimed immediately (while anonymous pages and dirty/modified pages have to be written to disk first).

* unevictable:

	the amount of memory that cannot be reclaimed; generally, it will account for memory that has been "locked" with mlock. It is often used by crypto frameworks to make sure that secret keys and other sensitive material never gets swapped out to disk.

* memory and memsw limits:

	These are not really metrics, but a reminder of the limits applied to this cgroup. The first one indicates the maximum amount of physical memory that can be used by the processes of this control group; the second one indicates the maximum amount of RAM+swap.

Accounting for memory in the page cache is very complex. If two processes in different control groups both read the same file (ultimately relying on the same blocks on disk), the corresponding memory charge will be split between the control groups. It's nice, but it also means that when a cgroup is terminated, it could increase the memory usage of another cgroup, because they are not splitting the cost anymore for those memory pages.

CPU metrics: cpuacct.stat
Now that we've covered memory metrics, everything else will look very simple in comparison. CPU metrics will be found in the cpuacct controller.

For each container, you will find a pseudo-file cpuacct.stat, containing the CPU usage accumulated by the processes of the container, broken down between user and system time. If you're not familiar with the distinction, user is the time during which the processes were in direct control of the CPU (i.e., executing process code), and system is the time during which the CPU was executing system calls on behalf of those processes.

Those times are expressed in ticks of 1/100th of a second. Actually, they are expressed in "user jiffies". There are USER_HZ "jiffies" per second, and on x86 systems, USER_HZ is 100. This used to map exactly to the number of scheduler "ticks" per second; but with the advent of higher frequency scheduling, as well as tickless kernels, the number of kernel ticks wasn't relevant anymore. It stuck around anyway, mainly for legacy and compatibility reasons.

Block I/O metrics
Block I/O is accounted in the blkio controller. Different metrics are scattered across different files. While you can find in-depth details in the blkio-controller file in the kernel documentation, here is a short list of the most relevant ones:

blkio.sectors:
contain the number of 512-bytes sectors read and written by the processes member of the cgroup, device by device. Reads and writes are merged in a single counter.

blkio.io_service_bytes:
indicates the number of bytes read and written by the cgroup. It has 4 counters per device, because for each device, it differentiates between synchronous vs. asynchronous I/O, and reads vs. writes.

blkio.io_serviced:
the number of I/O operations performed, regardless of their size. It also has 4 counters per device.

blkio.io_queued:
indicates the number of I/O operations currently queued for this cgroup. In other words, if the cgroup isn't doing any I/O, this will be zero. Note that the opposite is not true. In other words, if there is no I/O queued, it does not mean that the cgroup is idle (I/O-wise). It could be doing purely synchronous reads on an otherwise quiescent device, which is therefore able to handle them immediately, without queuing. Also, while it is helpful to figure out which cgroup is putting stress on the I/O subsystem, keep in mind that is is a relative quantity. Even if a process group does not perform more I/O, its queue size can increase just because the device load increases because of other devices.

Network Metrics
Network metrics are not exposed directly by control groups. There is a good explanation for that: network interfaces exist within the context of network namespaces. The kernel could probably accumulate metrics about packets and bytes sent and received by a group of processes, but those metrics wouldn't be very useful. You want per-interface metrics (because traffic happening on the local lo interface doesn't really count). But since processes in a single cgroup can belong to multiple network namespaces, those metrics would be harder to interpret: multiple network namespaces means multiple lo interfaces, potentially multiple eth0 interfaces, etc.; so this is why there is no easy way to gather network metrics with control groups.

Instead we can gather network metrics from other sources:

IPtables
IPtables (or rather, the netfilter framework for which iptables is just an interface) can do some serious accounting.

For instance, you can setup a rule to account for the outbound HTTP traffic on a web server:

$ iptables -I OUTPUT -p tcp --sport 80
There is no -j or -g flag, so the rule will just count matched packets and go to the following rule.

Later, you can check the values of the counters, with:

$ iptables -nxvL OUTPUT
Technically, -n is not required, but it will prevent iptables from doing DNS reverse lookups, which are probably useless in this scenario.

Counters include packets and bytes. If you want to setup metrics for container traffic like this, you could execute a for loop to add two iptables rules per container IP address (one in each direction), in the FORWARD chain. This will only meter traffic going through the NAT layer; you will also have to add traffic going through the userland proxy.

Then, you will need to check those counters on a regular basis. If you happen to use collectd, there is a nice plugin to automate iptables counters collection.

Interface-level counters
Since each container has a virtual Ethernet interface, you might want to check directly the TX and RX counters of this interface. You will notice that each container is associated to a virtual Ethernet interface in your host, with a name like vethKk8Zqi. Figuring out which interface corresponds to which container is, unfortunately, difficult.

But for now, the best way is to check the metrics from within the containers. To accomplish this, you can run an executable from the host environment within the network namespace of a container using ip-netns magic.

The ip-netns exec command will let you execute any program (present in the host system) within any network namespace visible to the current process. This means that your host will be able to enter the network namespace of your containers, but your containers won't be able to access the host, nor their sibling containers. Containers will be able to “see” and affect their sub-containers, though.

The exact format of the command is:

$ ip netns exec <nsname> <command...>
For example:

$ ip netns exec mycontainer netstat -i
ip netns finds the "mycontainer" container by using namespaces pseudo-files. Each process belongs to one network namespace, one PID namespace, one mnt namespace, etc., and those namespaces are materialized under /proc/<pid>/ns/. For example, the network namespace of PID 42 is materialized by the pseudo-file /proc/42/ns/net.

When you run ip netns exec mycontainer ..., it expects /var/run/netns/mycontainer to be one of those pseudo-files. (Symlinks are accepted.)

In other words, to execute a command within the network namespace of a container, we need to:

Find out the PID of any process within the container that we want to investigate;
Create a symlink from /var/run/netns/<somename> to /proc/<thepid>/ns/net
Execute ip netns exec <somename> ....
Please review Enumerating Cgroups to learn how to find the cgroup of a process running in the container of which you want to measure network usage. From there, you can examine the pseudo-file named tasks, which contains the PIDs that are in the control group (i.e., in the container). Pick any one of them.

Putting everything together, if the "short ID" of a container is held in the environment variable $CID, then you can do this:

$ TASKS=/sys/fs/cgroup/devices/$CID*/tasks
$ PID=$(head -n 1 $TASKS)
$ mkdir -p /var/run/netns
$ ln -sf /proc/$PID/ns/net /var/run/netns/$CID
$ ip netns exec $CID netstat -i
Tips for high-performance metric collection
Note that running a new process each time you want to update metrics is (relatively) expensive. If you want to collect metrics at high resolutions, and/or over a large number of containers (think 1000 containers on a single host), you do not want to fork a new process each time.

Here is how to collect metrics from a single process. You will have to write your metric collector in C (or any language that lets you do low-level system calls). You need to use a special system call, setns(), which lets the current process enter any arbitrary namespace. It requires, however, an open file descriptor to the namespace pseudo-file (remember: that's the pseudo-file in /proc/<pid>/ns/net).

However, there is a catch: you must not keep this file descriptor open. If you do, when the last process of the control group exits, the namespace will not be destroyed, and its network resources (like the virtual interface of the container) will stay around for ever (or until you close that file descriptor).

The right approach would be to keep track of the first PID of each container, and re-open the namespace pseudo-file each time.

Collecting metrics when a container exits
Sometimes, you do not care about real time metric collection, but when a container exits, you want to know how much CPU, memory, etc. it has used.

Docker makes this difficult because it relies on lxc-start, which carefully cleans up after itself, but it is still possible. It is usually easier to collect metrics at regular intervals (e.g., every minute, with the collectd LXC plugin) and rely on that instead.

But, if you'd still like to gather the stats when a container stops, here is how:

For each container, start a collection process, and move it to the control groups that you want to monitor by writing its PID to the tasks file of the cgroup. The collection process should periodically re-read the tasks file to check if it's the last process of the control group. (If you also want to collect network statistics as explained in the previous section, you should also move the process to the appropriate network namespace.)

When the container exits, lxc-start will try to delete the control groups. It will fail, since the control group is still in use; but that's fine. You process should now detect that it is the only one remaining in the group. Now is the right time to collect all the metrics you need!

Finally, your process should move itself back to the root control group, and remove the container control group. To remove a control group, just rmdir its directory. It's counter-intuitive to rmdir a directory as it still contains files; but remember that this is a pseudo-filesystem, so usual rules don't apply. After the cleanup is done, the collection process can exit safely.


