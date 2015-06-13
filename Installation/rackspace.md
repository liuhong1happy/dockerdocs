# Rackspace Cloud

通过安装Ubuntu安装Docker，在Rackspace上是相当简单的, 然后继续阅读 [*Ubuntu*](ubuntulinux.md)。

**当然，这里还是有一个申明:**

如果你使用任何Linux不携带有3.8内核，你将需要安装它。这是一个小小困难在Rackspace上。

Rackspace 启动你的服务通过 grub的 `menu.lst`
而不像非 `virtual` packages (e.g., Xen compatible)
内核那样, 尽管他们已经起作用了. 使用
`update-grub`的结果不会有所期待的结果,你需要手动设置内核。

**不要期望它作为生产环境的机器!**

    # update apt
    $ apt-get update

    # install the new kernel
    $ apt-get install linux-generic-lts-raring

好了，你现在有内核安装在`/boot/`, 接下来你需要创建系统引导。

    # find the exact names
    $ find /boot/ -name '*3.8*'

    # this should return some results

现在你需要手动编辑 `/boot/grub/menu.lst`,
你将发现一个底部区域已经存在的选项，复制上边一个然后替代新的内核。 确保新的内核在上边, 双击内核kernel以及initrd 行 指向右边的文件。

确保指定双击内核和initrd。

    # now edit /boot/grub/menu.lst
    $ vi /boot/grub/menu.lst

这将看到如下这样:

    ## ## End Default Options ##

    title              Ubuntu 12.04.2 LTS, kernel 3.8.x generic
    root               (hd0)
    kernel             /boot/vmlinuz-3.8.0-19-generic root=/dev/xvda1 ro quiet splash console=hvc0
    initrd             /boot/initrd.img-3.8.0-19-generic

    title              Ubuntu 12.04.2 LTS, kernel 3.2.0-38-virtual
    root               (hd0)
    kernel             /boot/vmlinuz-3.2.0-38-virtual root=/dev/xvda1 ro quiet splash console=hvc0
    initrd             /boot/initrd.img-3.2.0-38-virtual

    title              Ubuntu 12.04.2 LTS, kernel 3.2.0-38-virtual (recovery mode)
    root               (hd0)
    kernel             /boot/vmlinuz-3.2.0-38-virtual root=/dev/xvda1 ro quiet splash  single
    initrd             /boot/initrd.img-3.2.0-38-virtual

重启系统

    # reboot

指定内核被更新

    $ uname -a
    # Linux docker-12-04 3.8.0-19-generic #30~precise1-Ubuntu SMP Wed May 1 22:26:36 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux

    # nice! 3.8.

然后继续 [*Ubuntu*](ubuntulinux.md)。
