### daemon与service的关系

系统为了某些功能必须要提供一些服务，这个服务就称为service

service的提供需要程序的运作，达成service的程序就是daemon

早期systemV的init管理行为中daemon的主要分类：

### init管理机制的特色

​	init服务分类：

​		独立启动模式：服务独立启动，该服务直接常驻于内存，反应速度块

​		总管程序：要求socket或port，唤醒服务需要一点时间的延迟

​	服务相依性问题：顾名思义，有很多服务启动依赖于前提服务，也就是前提服务开启了才能开启。

​	执行等级的分类：

​	制定执行等级默认要启动的服务：

​		chkconfig daemon on

​		chkconfig daemon off

​		chkconfig --list daemon

​	执行等级的切换：runlevel	

### systemd使用的unit（服务单位）

​	平行处理所有服务，加速开机流程：

​	一经要求就相应的on-daemon启动方式：**常驻内存**，systemctl

​	服务相依性的自我检查：

​	依daemon功能分类：

​		service，socket，target，path，snspshot，timer

​	将多个daemons集合成为一个群组：

systemd的配置文件放置的目录：

​	/usr/lib/systemd/system：每个服务最主要的启动脚本配置，执行脚本的设定

​	/run/systemd/system：执行过程中产生的服务脚本

​	/etc/lib/systemd/system：管理员依据主机系统的需求所建立的执行脚本，系统开机会不会执行某些服务就是看底下设置

systemd的unit类型分类：

​	.service服务类型

​	.target环境类型

​		graphical.target

​		multi-user.target

​		rescue.target救援模式

​		emergency.target紧急处理系统的错误

​		shutdown.target关机的 流程

​		getty.target可以设定你需要几个tty之类的

​	.timer循环执行的服务

​	.socket插槽服务

​	.mount挂载服务

​	.path侦测特定的文件或目录服务，如打印机

取得目前的target： systemctl get-default

设定后面接的target成默认：systemctl set-default *  .target

切换到后面接的模式：systemctl isolate *.target



systemctl管理单一服务的启动/开机启动与观察状态：

​	start

​	stop

​	restart

​	status

​	reload：不关闭服务的情况下，重载配置文件让其生效

​	enable：开机启动

​	disable：关闭启动

​	is-active：查看正在运作中的服务

​	is-enable：开机时有没有预设要启动这个unit

daemon状态：

​	active（running）

​	active（exited）：仅执行一次就正常结束的服务

​	active（waiting）：

​	inactive：目前没有运作

enable：设置允许开机启动

disable：禁止开机启动

static：

mask:这个daemon无论如何都不能启动，因为已经被**强制注销**（不是删除），可以使用systemctl unmask方式改回状态

​	本质上就是创建了一个指向/dev/null的软链接

​	Loaded：masked（/dev/null）



systemctl观察系统上的所有服务：

​	list-units：列出启动的units

​	-all：列出全部

​	list-unit-files:

​	--type=TYPE：就是上面提到的unit type，

systemctl poweroff

sytemctl reboot/rescu/emergency

systemctl suspend：暂停模式将系统状态数据保存到内存，然后关闭掉硬件，并不是真正的关机，当取消的时候就正常开始运作，从内存中恢复，唤醒速度块。

systemctl hibernate：休眠模式是将系统状态保存到硬盘中，保存完毕后将计算机关机，唤醒之后开始工作，然后将保存在硬盘中的系统状态恢复过来。

透过systemctl分析各服务之间的依赖性：

​	systemctl list-dependencies [unit][--reverse

​		--reverse：反向追踪谁使用了这个unit

与system的daemon运作过程相关的目录简介：

​	systemctl list-sockets

cat /etc/services ：查询默认端口和服务

netstat -tlunp：查看网络监听端口程序



**systemctl配置文件相关目录简介**：

配置文件内的设定规则：

​	可以重复设定两个After在配置文件，后面的设定会取代前面，如果想要设定值归0,则在后面的After=后面不接东西，就是归0（reset）

​	如果设定参数需要【是/否】的项目，可以使用1，yes，on，true代表启动，用0，no，false，off代表关闭

systemctl配置文件的设定项目简介：

cat /usr/bin/systemd/system/sshd.service

​	[service]中哪些项目可用：

​	Type：说明daemon的启动方式，会影响到Execstart

​		single：启动后常驻内存

​		forking：由Execstart启动程序透过spawns延申出子程序来作为此daemon的主要服务

​		oneshot：工作完毕就结束，不会常驻内存	

​	Execstart：实际执行此daemon的指令或脚本程序

多重的重复设定方式：以getty为例，getty@.service

​	源文件：执行服务名称@.service

​	执行文件：执行服务名称@范例名称.service



​	

**systemctl针对timer的配置文件**

常驻在内存当作的systemd，timer.target协助定期处理各种任务

system.timer：

​	所有的systemd的服务产生的信息都会被log（记录），因此比crond好得多

​	各种timer工作可以跟systemd的服务相结合

**想要使用timer的功能，必须要有以下几个条件**：

​	系统的timer.target要启动

​	要有个xxx.service的服务存在（xxx自己取）

​	要有个xxx.timer的时间启动服务存在

​		到/etc/systemd/system建立一个xxx.timer

