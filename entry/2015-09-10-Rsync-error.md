---
layout: post
title: Rsync
category: 同步 
tags: [rsync]
---
> 报错直接可以在`/var/log/rsyncd.log`中查看.注意windows下面我们需要给SvcwRsync用户，管理同步目录的所有权限，基本上这样就可以了

- 问题一： 
	- @ERROR: chroot failed 
	- rsync error: error starting client-server protocol (code 5) at main.c(1522) [receiver=3.0.3] 

	- 原因： 服务器端的目录不存在或无权限，创建目录并修正权限可解决问题。 

- 问题二： 
	- @ERROR: auth failed on module tee 
	- rsync error: error starting client-server protocol (code 5) at main.c(1522) [receiver=3.0.3] 

	- 原因： 服务器端该模块（tee）需要验证用户名密码，但客户端没有提供正确的用户名密码，认证失败。 提供正确的用户名密码解决此问题。 

- 问题三： 
	- @ERROR: Unknown module 'tee_nonexists' 
	- rsync error: error starting client-server protocol (code 5) at main.c(1522) [receiver=3.0.3] 

	- 原因： 服务器不存在指定模块。提供正确的模块名或在服务器端修改成你要的模块以解决问题。 

	- 问题1： 在client上遇到问题： 
	- rsync -auzv --progress --password-file=/etc/rsync.pas root@192.168.133.128::backup /home/ 
	- rsync: could not open password file "/etc/rsync.pas": No such file or directory (2) 
	- Password: 
	- @ERROR: auth failed on module backup 
	- rsync error: error starting client-server protocol (code 5) at main.c(1506) [Receiver=3.0.7] 
	- 遇到这个问题：client端没有设置/etc/rsync.pas这个文件，而在使用rsync命令的时候，加了这个参数-- password-file=/etc/rsync.pas 

	- 问题2： 
	- rsync -auzv --progress --password-file=/etc/rsync.pas root@192.168.133.128::backup /home/ 
	- @ERROR: auth failed on module backup 
	- rsync error: error starting client-server protocol (code 5) at main.c(1506) [Receiver=3.0.7] 
	- 遇到这个问题：client端已经设置/etc/rsync.pas这个文件，里面也设置了密码111111，和服务器一致，但是 
	- 服务器段设置有错误，服务器端应该设置/etc/rsync.pas ，里面内容root:111111 ,这里登陆名不可缺少 

	- 问题3： 
	- rsync -auzv --progress --password-file=/etc/rsync.pas root@192.168.133.128::backup /home/ 
	- @ERROR: chdir failed 
	- rsync error: error starting client-server protocol (code 5) at main.c(1506) [Receiver=3.0.7] 
	- 遇到这个问题，是因为服务器端的/home/backup 其中backup这个目录并没有设置，所以提示：chdir failed 

- 问题4： 
	- rsync: write failed on "/home/backup2010/wensong": No space left on device (28) 
	- rsync error: error in file IO (code 11) at receiver.c(302) [receiver=3.0.7] 
	- rsync: connection unexpectedly closed (2721 bytes received so far) [generator] 
	- rsync error: error in rsync protocol data stream (code 12) at io.c(601) [generator=3.0.7] 
	- 磁盘空间不够，所以无法操作。 
	- 可以通过df /home/backup2010 来查看可用空间和已用空间 

- 问题5：网络收集问题 
	- 1、权限问题 
	- 类似如下的提示：rsync: opendir "/kexue" (in dtsChannel) failed: Permission denied (13)注意查看同步的目录权限是否为755 

	- 2、time out 
	- rsync: failed to connect to 203.100.192.66: Connection timed out (110) 
	- rsync error: error in socket IO (code 10) at clientserver.c(124) [receiver=3.0.5] 
	- 检查服务器的端口netstat –tunlp，远程telnet测试。 
	- 可能因为客户端或者服务端的防火墙开启 导致无法通信，可以设置规则放行 rsync（873端口） 或者直接关闭防火墙。 

	- 还有一种在同步过程中可能会提示没有权限 （将同步目录加上SvcwRsync全部权限即可，更简单的方法就是将SvcwRsync设为管理员即可）


	- 3、服务未启动 
	- rsync: failed to connect to 10.10.10.170: Connection refused (111) 
	- rsync error: error in socket IO (code 10) at clientserver.c(124) [receiver=3.0.5] 
	- 启动服务：rsync --daemon --config=/etc/rsyncd.conf 

	- 4、磁盘空间满 
	- rsync: recv_generator: mkdir "/teacherclubBackup/rsync……" failed: No space left on device (28) 
	- *** Skipping any contents from this failed directory *** 

	- 5、Ctrl+C或者大量文件 
	- rsync error: received SIGINT, SIGTERM, or SIGHUP (code 20) at rsync.c(544) [receiver=3.0.5] 
	- rsync error: received SIGINT, SIGTERM, or SIGHUP (code 20) at rsync.c(544) [generator=3.0.5] 
	- 说明：导致此问题多半是服务端服务没有被正常启动，到服务器上去查查服务是否有启动，然后查看下 /var/run/rsync.pid 文件是否存在，最干脆的方法是杀死已经启动了服务，然后再次启动服务或者让脚本加入系统启动服务级别然后shutdown -r now服务器

	- 6、xnetid启动 
	- rsync: read error: Connection reset by peer (104) 
	- rsync error: error in rsync protocol data stream (code 12) at io.c(759) [receiver=3.0.5] 
	- 查看rsync日志 
	- rsync: unable to open configuration file "/etc/rsyncd.conf": No such file or directory 
	- xnetid查找的配置文件位置默认是/etc下，根据具体情况创建软链接。例如： 
	- ln -s /etc/rsyncd/rsyncd.conf /etc/rsyncd.conf 
	- 或者更改指定默认的配置文件路径，在/etc/xinetd.d/rsync配置文件中。 
	- Rsync configure:

-	配置一：
-	ignore errors
-	说明：这个选项最好加上，否则再很多crontab的时候往往发生错误你也未可知，因为你不可能天天去看每时每刻去看log，不加上这个出现错误的几率相对会很高，因为任何大点的项目和系统，磁盘IO都是一个瓶颈
-	
-	Rsync error： 
-	错误一： 
-	@ERROR: auth failed on module xxxxx 
-	rsync: connection unexpectedly closed (90 bytes read so far) 
-	rsync error: error in rsync protocol data stream (code 12) at io.c(150) 
-	说明：这是因为密码设置错了，无法登入成功，检查一下rsync.pwd，看客服是否匹配。还有服务器端没启动rsync 服务也会出现这种情况。
-	错误二： 
-	password file must not be other-accessible 
-	continuing without password file 
-	Password: 
-	说明：这是因为rsyncd.pwd rsyncd.sec的权限不对，应该设置为600。如：chmod 600 rsyncd.pwd
-	错误三： 
-	@ERROR: chroot failed 
-	rsync: connection unexpectedly closed (75 bytes read so far) 
-	rsync error: error in rsync protocol data stream (code 12) at io.c(150) 
-	说明：这是因为你在 rsync.conf 中设置的 path 路径不存在，要新建目录才能开启同步
-	错误四： 
-	rsync: failed to connect to 218.107.243.2: No route to host (113) 
-	rsync error: error in socket IO (code 10) at clientserver.c(104) [receiver=2.6.9] 
-	说明：防火墙问题导致，这个最好先彻底关闭防火墙，排错的基本法就是这样，无论是S还是C，还有ignore errors选项问题也会导致
-	
-	错误五：
-	@ERROR: access denied to www from unknown (192.168.1.123)
-	rsync: connection unexpectedly closed (0 bytes received so far) [receiver]
-	rsync error: error in rsync protocol data stream (code 12) at io.c(359)
-	说明：此问题很明显，是配置选项host allow的问题，初学者喜欢一个允许段做成一个配置，然后模块又是同一个，致使导致
-	错误六：
-	rsync error: received SIGINT, SIGTERM, or SIGHUP (code 20) at rsync.c(244) [generator=2.6.9]
-	rsync error: received SIGUSR1 (code 19) at main.c(1182) [receiver=2.6.9]
-	说明：导致此问题多半是服务端服务没有被正常启动，到服务器上去查查服务是否有启动，然后查看下 /var/run/rsync.pid 文件是否存在，最干脆的方法是杀死已经启动了服务，然后再次启动服务或者让脚本加入系统启动服务级别然后shutdown -r now服务器
-	错误七：
-	rsync: read error: Connection reset by peer (104)
-	rsync error: error in rsync protocol data stream (code 12) at io.c(604) [sender=2.6.9]
-	说明：原数据目录里没有数据存在
