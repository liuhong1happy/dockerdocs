# 版本说明

## Docker Hub Enterprise

### DHE 1.0.1
(2015.5.11)

使用1.6.1 CS Docker Engine时地址兼容性问题。

### DHE 1.0.0
(2015.4.23)

第一个版本

## 商业支持的Docker Engine


###　CS Docker Engine 1.6.2-cs5
(2015.5.21)

For customers running Docker Engine on supported versions of RedHat Enterprise Linux with SELinux enabled, the docker build and docker run commands will not have DNS host name resolution and bind-mounted volumes may not be accessible. As a result, customers with SELinux will be unable to use hostname-based network access in either docker build or docker run, nor will they be able to docker run containers that use --volume or -v bind-mounts (with an incorrect SELinux label) in their environment. By installing Docker Engine 1.6.2-cs5, customers can use Docker as intended on RHEL with SELinux enabled.

For example, you see will failures like:

	[root@dhe ~]# docker -v
	Docker version 1.6.0-cs2, build b8dd430
	[root@dhe ~]# ping dhe.home.org.au
	PING dhe.home.org.au (10.10.10.104) 56(84) bytes of data.
	64 bytes from dhe.home.gateway (10.10.10.104): icmp_seq=1 ttl=64 time=0.663 ms
	^C
	--- dhe.home.org.au ping statistics ---
	2 packets transmitted, 2 received, 0% packet loss, time 1001ms
	rtt min/avg/max/mdev = 0.078/0.370/0.663/0.293 ms
	[root@dhe ~]# docker run --rm -it debian ping dhe.home.org.au
	ping: unknown host
	[root@dhe ~]# docker run --rm -it debian cat /etc/resolv.conf
	cat: /etc/resolv.conf: Permission denied
	[root@dhe ~]# docker run --rm -it debian apt-get update
	Err http://httpredir.debian.org jessie InRelease
	
	Err http://security.debian.org jessie/updates InRelease
	
	Err http://httpredir.debian.org jessie-updates InRelease
	
	Err http://security.debian.org jessie/updates Release.gpg
	  Could not resolve 'security.debian.org'
	Err http://httpredir.debian.org jessie Release.gpg
	  Could not resolve 'httpredir.debian.org'
	Err http://httpredir.debian.org jessie-updates Release.gpg
	  Could not resolve 'httpredir.debian.org'
	[output truncated]
	or when running a docker build:
	
	[root@dhe ~]# docker build .
	Sending build context to Docker daemon 11.26 kB
	Sending build context to Docker daemon
	Step 0 : FROM fedora
	 ---> e26efd418c48
	Step 1 : RUN yum install httpd
	 ---> Running in cf274900ea35
	
	One of the configured repositories failed (Fedora 21 - x86_64),
	and yum doesn't have enough cached data to continue. At this point the only
	safe thing yum can do is fail. There are a few ways to work "fix" this:
	
	[output truncated]
Affected Versions: All previous versions of Docker Engine when SELinux is enabled.

Docker highly recommends that all customers running previous versions of Docker Engine update to this release.

How to workaround this issue
Customers who choose not to install this update have two options. The first option is to disable SELinux. This is not recommended for production systems where SELinux is typically required.

The second option is to pass the following parameter in to docker run.

     --security-opt=label:type:docker_t
This parameter cannot be passed to the docker build command.

Upgrade notes
When upgrading, make sure you stop DHE first, perform the Engine upgrade, and then restart DHE.

If you are running with SELinux enabled, previous Docker Engine releases allowed you to bind-mount additional volumes or files inside the container as follows:

    $ docker run -it -v /home/user/foo.txt:/foobar.txt:ro <imagename>
In the 1.6.2-cs5 release, you must ensure additional bind-mounts have the correct SELinux context. For example, if you want to mount foobar.txt as read-only into the container, do the following to create and test your bind-mount:

Add the z option to the bind mount when you specify docker run.

	$ docker run -it -v /home/user/foo.txt:/foobar.txt:ro,z <imagename>
Exec into your new container.

For example, if your container is bashful_curie, open a shell on the container:

	$ docker exec -it bashful_curie bash
Use cat to check the permissions on the mounted file.

	$ cat /foobar.txt
the contents of foobar appear
If you see the file's contents, your mount succeeded. If you receive a Permission denied message and/or the /var/log/audit/audit.log file on your Docker host contains an AVC Denial message, the mount did not succeed.

type=AVC msg=audit(1432145409.197:7570): avc:  denied  { read } for  pid=21167 comm="cat" name="foobar.txt" dev="xvda2" ino=17704136 scontext=system_u:system_r:svirt_lxc_net_t:s0:c909,c965 tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file
Recheck your command line to make sure you passed in the z option.


###　CS Docker Engine 1.6.2-cs4
(2015.5.1３)

固定绑定回归到/sys。

## CS Docker Engine 1.6.1-cs3
(2015.5.11)

Docker Engine version 1.6.1 关于地址有数个缺陷，现在可以立马支持所有支持的平台。用户可以更新已经安装的Docker Engine 1.6.1作为最新的安装。

这里说明的数个缺陷可能会被恶意的Dockerfile 或者镜像利用。用户需要考虑运行他们自己的镜像或者通过信任同伴构建的镜像，例如官方镜像库。

请发送任何问题到security@docker.com。


### CVE-2015-3629 Symlink traversal on container respawn allows local privilege escalation

Libcontainer version 1.6.0 引进了很多修改，例如促进修改一个可挂载命名空间打破容器复生的限制。这允许恶意的镜像写文件到主机系统或者逃避集装箱化。

Libcontainer and Docker Engine 1.6.1已经弥补了这个缺点，用户运行非可信任的镜像会被要求更新Docker Engine。

Tõnis Tiigi发现的这个问题。

### CVE-2015-3627 Insecure opening of file-descriptor 1 leading to privilege escalation

文件描述符通过libcontainer传递到一个容器的pid-1进程，被发现被优先打开执行chroot，允许非安全情况下打开以及遍历符号链接。这允许恶意容器镜像触发一个本地权限被放大。


Libcontainer and Docker Engine 1.6.1已经解决这个问题， 用户运行非可信任的镜像会被要求更新Docker Engine。

Tõnis Tiigi发现了这个问题。

### CVE-2015-3630 Read/write proc paths allow host modification & information disclosure
Several paths underneath /proc were writable from containers, allowing global system manipulation and configuration. These paths included /proc/asound, /proc/timer_stats, /proc/latency_stats, and /proc/fs.

By allowing writes to /proc/fs, it has been noted that CIFS volumes could be forced into a protocol downgrade attack by a root user operating inside of a container. Machines having loaded the timer_stats module were vulnerable to having this mechanism enabled and consumed by a container.

We are releasing Docker Engine 1.6.1 to address this vulnerability. All versions up to 1.6.1 are believed vulnerable. Users running untrusted images are encouraged to upgrade.

Discovered by Eric Windisch of the Docker Security Team.

### CVE-2015-3631 Volume mounts allow LSM profile escalation
By allowing volumes to override files of /proc within a mount namespace, a user could specify arbitrary policies for Linux Security Modules, including setting an unconfined policy underneath AppArmor, or a docker_t policy for processes managed by SELinux. In all versions of Docker up until 1.6.1, it is possible for malicious images to configure volume mounts such that files of proc may be overridden.

We are releasing Docker Engine 1.6.1 to address this vulnerability. All versions up to 1.6.1 are believed vulnerable. Users running untrusted images are encouraged to upgrade.

Discovered by Eric Windisch of the Docker Security Team.

### AppArmor policy improvements

The 1.6.1 release also marks preventative additions to the AppArmor policy. Recently, several CVEs against the kernel have been reported whereby mount namespaces could be circumvented through the use of the sys_mount syscall from inside of an unprivileged Docker container. In all reported cases, the AppArmor policy included in libcontainer and shipped with Docker has been sufficient to deflect these attacks. However, we have deemed it prudent to proactively tighten the policy further by outright denying the use of the sys_mount syscall.

Because this addition is preventative, no CVE-ID is requested.


###　CS Docker Engine 1.6.0-cs2
(2015.4.23)

第一个版本，请查看[Docker Engine 1.6.0版本说明](https://docs.docker.com/release-notes/)了解更多信息。
