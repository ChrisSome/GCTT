安全实验室：功能
	
	#再此实验室中你将学到Linux内核基本的功能，你将了解到docker在其上如何运作，一些基本命令去查看并管理docker,以及在新容器中如何增删功能
	
	#你需要完成下列步骤
	
		1.功能简介
	
		2.搭建docker
	
		3.测试docker功能
	
		4.专业信息

	1.功能简介
		这里你会了解到一些基本信息
		
			Linex内核可以将root用户权限拆解成不同单元，我们称之为功能。例如，chown指令是用来控制用户管理UIDs和GIDs，CAP_DAC_OVERRIDE功能是允许root用户跳过文件读写及执行时的内核权限检查。几乎所有与Linux root用户相关的特殊功能都会被分解为单独的功能

			root权限拆分为普通操作允许你进行以下操作：
				.从root账号下删除单个功能，降低其危险性
				2.为非root的普通级别的用户添加权限
			功能适用于户文件和线程，文件功能允许用户以更高权限执行项目。这类似于设置uid的工作方式。线程管理追踪当前正在运行的项目的状态

		Linex内核允许你设置对文件 线程操作的权限边界

		Docker施加了一些限制，使得使用功能更加简单。例如，文件功能被储存在文件的扩展属性中，当你在构建Docker映像时，扩展属性就被分离出来，这就意味着你通常不必关注容器中的文件功能

		你当然可以在容器运行时将文件通能添加进去，但是并不推荐这么做

		在一个没有基本文件管理功能的环境中，应用不可能将权限提高到已设置的边界之外（此边界时不可自动增长的），Docker在启动容器之前就已设定好边界。你可以用Docker命令去增删已设定的权限边界

		默认情况下，Docker会利用白名单功能删除必要之外的所有功能

	2.使用Docker
		
		这一步你会了解到管理Docker功能的基本方法，你也会了解到用于root用户管理Docker功能的命令

		在Docker1.12版本中你有三种高级选择去使用docker功能
			
			1.以root身份运行容器及其一系列的功能，并在你的容器中进行常规管理
			
			2.运行容器及其部分功能，并不可在容器中改变
			
			3.以未授权身份在无任何功能的情况下运行容器
		
		选项2时Docker1.12版本中最通用，选项3较为理想，但不现实。选项一在任何情况下都应避免

		注意：在未来的Docker版本中，有肯能会添加一个选项，此选项允许你以非root用户在有限功能下启用容器，这种情况下正确的做法是需要在4.3版本中添加到Linux内核中的环境功能。Docker能否在旧版本的内核中执行这种操作还需研究

		在下列指令中，$CAP 将会被用来声明一个或者多个单独功能，我们会在下一章节中进行测试
			
			1.在容器中删除root用户的功能
				docker run --rm -it --cap-drop $CAP alpine sh
			
			2.在容器中增加root用户的功能
				docker run --rm -it --cap-add $CAP alpine sh
			
			3.删除所有功能，然后将单个功能显式添加到容器的根帐户
				docker run --rm -it --cap-drop ALL --cap-add $CAP alpine sh
		
		Linux内核为所有的功能前缀添加常量“CAP_”.例如CAP_CHOWN, CAP_NET_ADMIN, CAP_SETUID, CAP_SYSADMIN 等。Docker功能常量没有以“CAP_”做前缀，因为这样会跟Linux内核冲突

		更多关于功能的信息，包括全部的功能列表，请参考。http://man7.org/linux/man-pages/man7/capabilities.7.html

	3.测试Docker功能
		
		在这一章节你会开启各种新的容器，每次你都可以使用此前章节学到的命令去调整运行容器相关联的功能
			
			1.运行一个新容器，并证明容器的root用户可以改变文件的属主
				docker run --rm -it alpine chown nobody /
				
				这个命令并无响应码表明执行成功，此指令成功运行因为默认行为是root用户启动新容器。在默认情况下，root用户具备chown功能
			
			2.启动另一个容器并删除容器下root账户的包括chown在内的所有功能。记得Docker在查找功能常量地址时并不使用“CAP_”前缀
				
				docker run --rm -it --cap-drop ALL --cap-add CHOWN alpine chown nobody /
				
				此指令也不会返回响应码去表明已经成功执行，这个操作成功时因为尽管你删除了容器root用户的所有所有功能，你又将chown功能添加回去了。chown功能改变文件属主所必须的
			
			3.启用另一个新的容器，将root用户除chown意外的其他功能删除
				docker run --rm -it --cap-drop CHOWN alpine chown nobody /

				chown: /: Operation not permitted

				此时这条命令返回一个错误码声明执行失败，是因为容器的root用户没有chown能力，因此不能改变文件或文件夹的属主

			4.创建另一个新的容器，并尝试添加chown功能给非root用户，在此我们称之为nobody。并改变文件或文件夹的属主

				docker run --rm -it --cap-add chown -u nobody alpine chown nobody /
				
				chown: /: Operation not permitted

				上面的命令执行失败时因为Docker尚不支持为非root用户添加功能。

		在此章节中你为新的容器增删了功能，你已经看到，我们可以非常精细地为容器的root用户增删功能。你也了解到Docker尚不支持为非root用户增删功能

	4.额外信息
		
		实验室的其余部分将为你展示关于Linux shell运行的工具

		管理功能又两套主要的工具
			
			libcap 专注于操控指令
			
			libcap-ng 有一些有用的审计工具
		
		下面是一些来自上面工具的一些有用的指令
		
			或许你需要安装一些上述指令所需要的包

			libcap
				capsh -允许你进行功能测试和有限的调试
				setcat -为文件设置功能位
				getcap -从文件获取功能位
			
			libcap-ng
				pscap -列取正在运行的进程的功能
				filecap -列取文件功能
				captest -测试和列取当前进行的功能
		
		此章节其余部分会展示libcap 和 libcap-ng的一些例子

		列取所有功能
			
			下面的指令将开启一个新的容器，此容器使用Alpine Linux，安装libcap包并列取功能

			docker run --rm -it alpine sh -c 'apk add -U libcap; capsh --print'

			  (1/1) Installing libcap (2.25-r0)
			   Executing busybox-1.24.2-r9.trigger
			   OK: 5 MiB in 12 packages
			   Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+eip
			   Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
			   Securebits: 00/0x0/1'b0
			    secure-noroot: no (unlocked)
			    secure-no-suid-fixup: no (unlocked)
			    secure-keep-caps: no (unlocked)
			   uid=0(root)
			   gid=0(root)
			   groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)

			上面是有空格分割的批量集合。统一集合的多个功能由逗号分隔。每组集合中“+”后面的字母如下：
				e 代表 effective 有效
				i 代表 inheritable 继承
				p 代表 permitted 允许
			
			更多信息见。http://man7.org/linux/man-pages/man7/capabilities.7.html

			功能实验
				
				capsh命令对于功能测试很有用，capsh --help会显示如何使用
					docker run --rm -it alpine sh -c 'apk add -U libcap;capsh --help'

					fetch http://dl-cdn.alpinelinux.org/alpine/v3.5/main/x86_64/APKINDEX.tar.gz
					fetch http://dl-cdn.alpinelinux.org/alpine/v3.5/community/x86_64/APKINDEX.ta
					r.gz
					(1/1) Installing libcap (2.25-r1)
					Executing busybox-1.25.1-r0.trigger
					OK: 4 MiB in 12 packages
					usage: capsh [args ...]
					  --help         this message (or try 'man capsh')
					  --print        display capability relevant state
					  --decode=xxx   decode a hex string to a list of caps
					  --supports=xxx exit 1 if capability xxx unsupported
					  --drop=xxx     remove xxx,.. capabilities from bset
					  --caps=xxx     set caps as per cap_from_text()
					  --inh=xxx      set xxx,.. inheritiable set
					  --secbits=<n>  write a new value for securebits
					  --keep=<n>     set keep-capabability bit to <n>
					  --uid=<n>      set uid to <n> (hint: id <username>)
					  --gid=<n>      set gid to <n> (hint: id <username>)
					  --groups=g,... set the supplemental groups
					  --user=<name>  set uid,gid and groups to that of user
					  --chroot=path  chroot(2) to this path
					  --killit=<n>   send signal(n) to child
					  --forkfor=<n>  fork and make child sleep for <n> sec
					  ==             re-exec(capsh) with args as for --
					  --             remaing arguments are for /bin/bash
                 		(without -- [capsh] will simply exit(0))

                警告：--drop好像是你想要的，但是它只会影响边界集合，这会让人很迷惑，因为它一般不会从有效集合或者继承集合中拿掉功能。一般情况下你可能想使用 --cap

            修正功能：
            	
            	libcap 和 libcap-ng都可以被用来修正功能
            		1.利用libcap去修正文件功能
            			下面的这个命令显示如何将CAP_NET_RAW置为有效,并通过$file授权.setcap命令调用libcap去完成上述功能

            				setcap cap_net_raw=ep $file
            		2.适用libcap-ng设置文件功能
            			filecap命令调用 libcap-ng
            			filecap /absolute/path net_raw

            			提示:filecap需要绝对路径,类似'./'的简写是不可以的

            	审计:
            		从文件中读取动能有多种方式

            		1.适用libcap:
            			getcap $file
            			$file = cap_net_raw+ep

            		2.使用libcap-ng
            			$ filecap /absolue/path/to/file

						file                     capabilities
						/absolute/path/to/file        net_raw

					3.使用扩展属性(attr包)

						getfattr -n security.capability $file

						# file: $file
						security.capability=0sAQAAAgAgAAAAAAAAAAAAAAAAAAA=

				提示:
					Docker映射不能设置具有功能为的文件.这样减少了Docker容器使用提升权限功能的风险.但是,这样有可能会将包含功能位的文件卷装入容器,因此你再操作时需要小心

					1.你可以用以下命令审计功能位的目录

						# with libcap
						getcap -r /

						# with libcap-ng
						filecap -a
					2.要删除功能位你可以使用
						# with libcap
						setcap -r $file

						# with libcap-ng
						filecap /path/to/file none

			延伸阅读:
				本文解释在很多细节上解释了docker功能.它将帮你理解功能集如何相互作用,如果你计划运行高权限的docker容器并在其中的手动管理,本文将非常有用

				这是功能的手册页,只要没有设置了功能位的文件,功能集之间的大部分复杂交互都不会影响docker容器

作者:manomarks

原文链接:https://training.play-with-docker.com/security-capabilities/

译者:Chris(https://github.com/ChrisSome)

