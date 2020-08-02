# 目录
<!-- vim-markdown-toc GFM -->

- [硬盘与文件系统](#硬盘与文件系统)
  - [分区工具](#分区工具)
  - [LVM](#lvm)
  - [XFS](#xfs)
  - [挂载](#挂载)
  - [归档包](#归档包)
  - [文件权限](#文件权限)
- [用户与登录](#用户与登录)
- [系统资源](#系统资源)
- [日志](#日志)
- [systemd](#systemd)
  - [units配置](#units配置)
    - [[Unit]](#unit)
    - [[Install]](#install)
    - [[Service]](#service)
    - [[Timer]](#timer)
    - [[Socket]](#socket)
  - [命令](#命令)
- [GRUB](#grub)
  - [GRUB配置](#grub配置)
  - [GRUB Shell](#grub-shell)
  - [GRUB选单](#grub选单)
  - [系统救援](#系统救援)
- [主机配置](#主机配置)
- [计算机网络](#计算机网络)
  - [网络配置](#网络配置)
  - [NetworkManager服务](#networkmanager服务)
  - [其它网络命令](#其它网络命令)
- [基础命令](#基础命令)
  - [读取文件信息](#读取文件信息)
  - [文件的修改、创建与删除](#文件的修改创建与删除)
  - [文件的搜索](#文件的搜索)
  - [其它](#其它)

<!-- vim-markdown-toc -->
# 硬盘与文件系统
* GPT分区表组成：
    * MBR保护
        > 防止不识别GPT的程序误读
    * GPT头  
        > 磁盘GUID、GPT版本、校验、分区项信息等
    * 分区项  
        > 分区GUID、类型、属性、标签、分区位置
    * 分区
        > 存储文件系统数据的硬盘区块
    * 备份区  
        > GPT头与分区项的备份
<!-- -->

* XFS文件系统：
    * 结构：
        * 分配组
            * 超级块(只在第一个分配组)
            * 空闲空间索引(B+tree)
            * Inode索引(B+tree)
        * Inodes
        * Blocks
        * Log
        * Realtime
    * XFS特性
        * 分配组的并行性
        * 日志与恢复
        * 延迟分配
        * 扩展属性
<!-- -->

## 分区工具
* gdisk  *DEV* [-l]
    * m ：普通模式
    * x ：专家模式
    * r ：恢复模式
    * ? ：帮助
* partprobe：让内核重载硬盘信息
<!-- -->

* parted  *DEV*
    * p
    * mktable
    * mkpart *LABLE* *START* *END*
    * resizepart *NUM* *END*

    * rm *NUM*
    * name *NUM* *NAME*
    * set *NUM* *FLAG* on/off
        * FLAG       ：raid  lvm  boot  hidden  diag
* partprobe：让内核重载硬盘信息
<!-- -->

* 分区标签
```
    parted-Flag      Type                     Attr
        hidden         -                        0
        boot        EFI System                  -
        diag        Windows RE                  -
        msftres     Micorsoft reserved          -
```
<!-- -->

## LVM
* LVM-PV阶段
    * pvs
    * pvdisplay  *PV*
    * pvcreate  *DEV*
    * pvremove  *DEV*
<!--  -->

* LVM-VG阶段
    * vgs
    * vgdisplay  *VG*
    * vgcreate *VG*  *PV*
        * -s ：指定PE大小
    * vgextend *VG*  *PV*
    * vgreduce *VG*  *PV*
    * vgchange  -a `y/n`
    * vgremove  *VG*
<!--  -->

* LVM-LV阶段
    * lvs
    * lvdisplay  *LV*
    * lvcreate
        * -L *SIZE* -n *LV*                 ：普通LV
        * -L *SIZE* -T *VG/LVpool*          ：建立LVpool
        * -V *SIZE* -T *VG/LVpool* -n *LV*  ：在指定LVpool中建立LV
        * -s -L *SIZE* -n *LVS* *VG/LV*     ：建立快照
    * lvresize
        * -L +|-*SIZE*
    * lvchange  -a y|n
    * lvremove  *LV*
<!-- -->

## XFS
* mkfs.xfs  *DEV*
    * -f        ：强制
    * -L        ：设置文件系统标签
    * -b size=
    * -i  size=
    * -d agcount=
    * -d file
    * -l  external,logdev=  ,size=
<!-- -->

* xfs_admin
    * -l        ：查看文件系统标签
    * -L        ：修改标签
    * -u        ：查看文件系统uuid
    * -U        ：修改uuid
<!-- -->

* xfsdump *MP*
    > 注 ：挂载点*MP*末不能带/号
    * -l        ：备份级别0为全量，其他在前一级基础上增量
    * -L        ：备份文件标签
    * -M        ：备份设备标签
    * -f        ：指定备份文件
    * -e        ：排除属性含d的文件
    * -s        ：指定目录（无增量备份）
* xfsrestore *MP*
    * -f        ：指定使用的备份文件
    * -s        ：只还原指定的文件或目录
    * -I        ：查询基础数据库/var/lib/xfsdump/inventory/
<!-- -->

* xfs_repair
    * -f        ：对image-file修复
    * -n        ：只检测
    * -l        ：指定logdev
    * -d        ：用于单用户模式强制恢复以ro挂载的/
<!-- -->

* xfs_info
* xfs_growfs
<!-- -->

## 挂载
> 文件系统抽象由内核的VFS模块提供，Unix一切皆文件。  
> 其中包括：  
> 硬盘文件系统（文件系统驱动）  
> 内存文件系统（内核接口）  
> 硬件设备（udev映射）

* lsblk
    * -f        ：显示文件系统类型
    * -m        ：权限及所有者
<!--  -->

* blkid [*DEV*]
    * 文件系统标签与类型
    * 文件系统UUID（fstab所用）
    * 分区标签
    * 分区UUID
<!-- -->

* df
    * -T        ：显示文件系统类型
    * -h        ：人性化size
    * -i        ：显示inode使用情况
<!-- -->

* mount *DEV* *MP*
    * -a        ：忽略其他参数，按/etc/fstab挂载
    * --bind    ：转移挂载点
    * --rbind   ：递归转移挂载点
    * -t        ：指定文件系统类型(sysfs,proc,devtmpfs,tmpfs)
    * -o        ：
        * remount       ：重新挂载
        * default       ：默认属性
        * loop          ：挂载loop设备
        * logdev=       ：挂载xfs时指定logdev
        * nouuid        ：用于挂载LVM快照
        * iocharset=    ：字符集
        * ro/rw
        * sync/async
        * atime/noatime
        * suid/nosuid
        * exec/exec
        * userquota/grpquota/quota/noquota
* umount
<!-- -->

## 归档包
制作归档包时，应该让解包出来的文件都在一个目录中，故一般在要打包的目录的父目录进行操作从而将整个目录打包

* zip  *ZIPFILE*  *FILES*
    * -[1-9]    ：压缩等级，越大压缩比越高
<!-- -->

* gzip/bzip2/xz
    * -[1-9]    ：压缩等级，越大压缩比越高
    * -k        ：保存原文件不删除
    * -l        ：查看压缩包信息
<!-- -->

* tar
    * -[z|j|J]    ：gzip : bzip2 : xz
    * -[c|x|t|u]  ：打包:解包:查询:更新
    * --delete    ：删除
    * -f          ：指定压缩文件名
    * -v          ：详述
    * -p          ：保留权限等信息
    * --exlcude   ：排除，pattern
    * -C          ：解包时指定路径
<!-- -->

* dd
    * if=
    * skip=
    * of=
    * seek=
    * bs=
    * count=
    * conv=
        * lcase      ：小写
        * ucase      ：大写
        * notrunc    ：不截断，覆盖
> 例：dd if=*manjaro.iso* of=*usb-dev* bs=8M oflag=sync status=progress
<!-- -->

* losetup */dev/loop0*  *loopfile*
    > 制作loop设备
* losetup -d */dev/loop0*
    > 解除loop设备
<!-- -->

## 文件权限
* 权限信息
    > 只有owner与root能修改，使用stat命令可以查看详细信息  
    > `umask`即默认权限掩码，设置后会掩盖默认权限，默认为022，从而目录默认权限为755，文件默认644
    * Uid与Gid      ：指明文件所属的owner与group
    * `r w x`       ：**读/写/执**权限，以八进制数字表示时从高位到低位依次代表是否具有**读/写/执**权限
    * `s S t T`     ：**SUID/SGID/SBIT**特殊权限，以八进制表示时从高位到低位依次表示是否具有**SUID/SGID/SBIT**权限
    * OOOO          ：4位八进制数，第一个表示特殊权限，后三个分别表示owner/group/other的读/写/执权限
* 对普通文件
    * r/w表示可读/写其对应block
    * x表示可以执行该文件形成进程
    * SUID表示执行时环境变量EUID改为owner
    * SGID表示执行时环境变量EGID改为group
* 对目录
    * r/w表示可读/写其对应block内存储的entry
    * x表示能否对目录下文件进行访问，即使没有rw也可以“摸黑访问”
    * SGID表示所有创建在此目录的普通文件的gid默认为目录的gid
    * SBIT表示该目录下的普通文件只有其owner与目录owner能删除
<!-- -->

* setfacl
    > 设置ACL权限，优先级在owner和group之后
* getfacl
<!--  -->

* chattr
    > 设置文件额外属性
    * -R：    递归目录
    * 以下选项的前缀，-设置，+取消
        * a   ：    只能追加
        * i   ：    无法变更
        * A   ：    不更新atime
        * S   ：    同步存储文件
        * d   ：    不被dump
* lsattr
<!-- -->

* su
    * `-`         ：转为root
    * `- user`    ：转为user
    * `-c`        ：用对应目标用户执行一条命令
<!-- -->

* sudo *CMD*
    * -u        ：使用目标用户权限(仅root可用)
    * -l        ：列出本用户sudo信息
    * -b        ：后台执行
<!--  -->

* visudo
    > /etc/sudoers与/etc/sudoers.d  
    > user host=(root) cmd，!cmd  
    > %grp host=(%root) NOPASSWD:ALL
<!-- -->

# 用户与登录
* 登录文件
    > 由PAM模块控制登录验证
    * /etc/nologin   ：若存在则只允许root登陆

    * /etc/issue     ：本地控制台登陆提示
    * /etc/motd      ：远程登录提示
    * /etc/login.defs：用户登陆设置
<!--  -->

* 用户文件
    * /etc/passwd
        * `用户名:密码:UID:GID:描述信息:主目录:默认Shell`

    * /etc/shadow
        * `用户名:加密密码:最后一次修改时间:最小修改时间间隔:密码有效期:密码需要变更前的警告天数:密码过期后的宽限时间:账号失效时间:保留`
        > 在密码前加上 "!"、"*" 或 "x" 使密码暂时失效
    * /etc/group
        * `组名:密码:GID:组附加用户列表`
    * /etc/gshadow
        * `组名:加密密码:组管理员:组附加用户列表`
    * /etc/skel/：建立用户主目录时拷贝此目录
<!-- -->

* last      ：系统的启动与用户登陆日志
    > /var/log/wtmp
* lastlog   ：每个用户最后一次登陆时间
    > /var/log/lastlog
* lastb     ：上次错误登录记录
    > /var/log/btmp
* w         ：系统现在的登录情况
    > /var/run/ulmp
<!-- -->

* id    ：该用户信息
* groups：该用户参与的group
<!-- -->

* useradd
    * -u        ：UID
    * -g        ：GID
    * -G        ：附加组
    * -c        ：描述信息
    * -d        ：主目录绝对路径
    * -m        ：自动创建主目录
    * -s        ：默认shell
    * -o        ：允许用户UID相同
    * -r        ：系统用户(1-499)
    * -D        ：查看或修改默认配置/etc/default/useradd
<!-- -->

* usermod
    * -l        ：用户名
    * -u        ：UID
    * -g        ：GID
    * -G        ：附加组
    * -c        ：描述信息
    * -d        ：主目录绝对路径
    * -s        ：默认shell
<!-- -->

* userdel
    * -r        ：删除主目录，邮箱需要手动删除
<!-- -->

* passwd
    * -S        ：查看信息
    * -l        ：锁定用户
    * -u        ：解锁用户
    * --stdin   ：指明从管道读取密码
<!-- -->

* chage
    * -l        ：详情
    * -d        ：最后一次修改YYYY-MM-DD，为0强制修改
    * -m        ：最小修改间隔天数
    * -M        ：最大修改间隔天数(密码过期后强制修改)
    * -W        ：密码到期提前警告天数
    * -I        ：宽限天数，这段时间用户可以登录，但会强制其修改密码
    * -E        ：失效日期YYYY-MM-DD，-1则无
<!-- -->

* groupadd
    * -g        ：GID
    * -r        ：系统群组
<!-- -->

* groupmod
    * -g        ：GID
    * -n        ：组名
<!-- -->

* gpasswd
    * -A        ：管理员
    * -r        ：移除群组密码
    * -R        ：密码失效
    * -M        ：将用户加入群组(root)
    * -a        ：将用户加入群组
    * -d        ：移除用户
<!-- -->

* groupdel      ：不能删除初始组
<!-- -->

* newgrp        ：启动新shell并修改GID
<!-- -->

# 系统资源
* 进程标识
    * STATE
    * CMD
    * TTY
    * UID、EUID、GID、EGID
    * PID、PPID、PGID、SID、TPGID
    * SELinux-Context
<!-- -->

* CPU使用时间
    * real：运行期间流逝的时间
    * sy  ：内核进程
    * us  ：用户进程(un-niced)
    * ni  ：用户进程(niced)
    * id  ：空闲资源
    * wa  ：等待I/O
    * hi  ：硬中断请求服务
    * si  ：软中断请求服务
    * st  ：虚拟机偷取的时间，即虚拟CPU等待实际CPU
<!-- -->

* 进程优先级
    * PRI(Priority)与NI(Nice)
        * PRI (最终值) = PRI (原始值) + NI
        * PRI值是由内核动态调整的，用户不能直接修改，只能通过修改 NI 值来影响 PRI 值，间接地调整进程优先级
    * NI 值越小，进程的 PRI 就会降低，该进程就越优先被 CPU 处理
    * NI 范围是 -20~19。
    * 普通用户调整 NI 值的范围是 0~19，而且只能调整自己的进程。
    * 普通用户只能调高 NI 值，而不能降低
    * 只有 root 用户才能设定进程 NI 值为负值，而且可以调整任何用户的进程
<!-- -->

* signal

| 信号编号 | 信号名称          | 信号描述             | 详细解释                                   | 默认处理方式           | Exit Code |
|----------|-------------------|----------------------|--------------------------------------------|------------------------|-----------|
| 1        | SIGHUP            | 会话挂断             | 用户终端连接结束，通知同一session内的进程  | 终止                   | 1         |
| 2        | SIGINT            | 终端中断             | 一般由 Ctrl+C 发射                         | 终止                   | 2         |
| 3        | SIGQUIT           | 终端退出             | 一般由 Ctrl+\ 发射                         | 终止、core dump        | 131       |
| 4        | SIGILL            | 非法指令             | 试图执行无执行权限的页中的指令，或堆栈溢出 | 终止、core dump        | 132       |
| 5        | SIGTRAP           | 跟踪/断点陷阱        | 由debugger使用                             | 终止、core dump        | 133       |
| 6        | SIGABRT           | 进程终止自己         | 程序自生发现错误并调用此信号终止自身       | 终止、core dump        | 134       |
| 7        | SIGBUS            | 非法地址             | 地址对齐问题                               | error 终止、core dump  | 135       |
| 8        | SIGFPE            | 算术异常             | 除0、溢出等                                | 终止、core dump        | 136       |
| 9        | SIGKILL           | 杀死进程             | 强制结束进程，不可捕获                     | 终止                   | 9         |
| 10       | SIGUSR1           | 用户自定义信号1      |                                            | 终止                   | 10        |
| 11       | SIGSEGV           | 段错误               | 访问未分配的地址                           | 终止、core dump        | 139       |
| 12       | SIGUSR2           | 用户自定义信号2      |                                            | 终止                   | 12        |
| 13       | SIGPIPE           | 管道断开             | 管道无读者进程                             | 终止                   | 13        |
| 14       | SIGALRM           | 定时器信号           |                                            | 终止                   | 14        |
| 15       | SIGTERM           | 终止进程             | 正常终止进程                               | 终止                   | 15        |
| 16       | SIGSTKFLT         | 栈错误               |                                            | 终止                   | 16        |
| 17       | SIGCHLD           | 子进程退出           |                                            | 忽略                   | 无        |
| 18       | SIGCONT           | 继续执行             |                                            | 若停止则继续执行       | 无        |
| 19       | SIGSTOP           | 停止执行             | 暂停执行，不可捕获                         | 暂停执行               | 无        |
| 20       | SIGTSTP           | 终端停止             | 一般由 Ctrl+Z 发射                         | 暂停执行               | 无        |
| 21       | SIGTTIN           | 后台读取             | 后台进程试图读取终端输入                   | (tty input) 暂停执行   | 无        |
| 22       | SIGTTOU           | 后台写入             | 后台进程试图进行终端输出                   | (tty out put) 暂停执行 | 无        |
| 23       | SIGURG            | io紧急数据           | out-of-band数据到达socket时产生            | 忽略                   | 无        |
| 24       | SIGXCPU           | 突破对cpu时间的限制  |                                            | 终止、core dump        | 152       |
| 25       | SIGXFSZ           | 突破对文件大小的限制 |                                            | 终止、core dump        | 153       |
| 26       | SIGVTALRM         | 虚拟定时器超时       |                                            | 终止                   | 26        |
| 27       | SIGPROF           | 性能分析定时器超时   |                                            | 终止                   | 27        |
| 28       | SIGWINCH          | 终端窗口尺寸发生变化 |                                            | 忽略                   | 无        |
| 29       | SIGIO             | io时可能产生         | 文件描述符准备就绪                         | 终止                   | 29        |
| 30       | SIGPWR            | 电量行将耗尽         |                                            | 终止                   | 30        |
| 31       | SIGSYS            | 错误的系统调用       |                                            | 终止、core dump        | 159       |
| 34~64    | SIGRTMIN~SIGRTMAX | 实时信号             |                                            | 终止                   | 34~64     |
> 参考至[linux信号表](https://blog.csdn.net/xuyaqun/article/details/5338563)

* jobs
    * -l        ：显示PID
    * -r        ：显示running jobs
    * -s        ：显示suspended jobs
<!-- -->

* fg/bg  *%JID*
<!-- -->

* ps
    * -l        ：只显示当前shell的进程
    * -le       ：显示所有进程
* pstree -Uup
<!-- -->

* kill -(signal) PID
    PID为负，表示其绝对值为进程组号
* killall -(signal) Pname
    * -I        ：忽略大小写
* pkill (-signal)
    * -ce       ：显示匹配进程的数量并显示每个进程信息
    * -u        ：EUID
    * -U        ：UID
    * -G        ：GID
    * -P        ：PPID
    * -s        ：PSID
    * -t        ：TTY
<!-- -->

* nice -n NI CMD
* renice -n PID
<!-- -->

* top
    * -bn       ：指定刷新次数并手动重定向到文件
    * -d        ：指定刷新周期
    * -u        ：指定监视用户
    * -p        ：指定监视PID
    * 交互命令
        * P/M/T/N     ：按cpu/mem/time/pid排序
        * c   ：显示完整命令
        * e   ：切换内存单位
        * 1   ：显示多核
        * z   ：切换颜色
        * f   ：选择域的显示/排序
        * t/m ：切换cpu/mem显示模式
        * k/r ：kill/renice
* htop：top的替代品
<!-- -->

* free
    * -wh       ：人性化输出
    * -s        ：刷新周期
    * -c        ：刷新次数
<!-- -->

* vmstat -w  [周期]  [次数]
<!-- -->

* iostat -h
* iotop
<!-- -->

* lspci
    * -s        ：显示指定设备
    * -vv       ：显示详情
* lsusb -t
* lscpu
<!-- -->

* nohup CMD [&]
<!-- -->

* lsof
    > 列出打开的文件
    * -a        ：and条件逻辑
    * -u        ：UID
    * -p        ：PID
    * -c        ：指定进程cmd的开头字符串
    * +d        ：列出目录下被打开的文件，+D递归
<!-- -->

* fuser -uv FILE/DIR
    > 列出打开目标文件的进程
<!-- -->

* ulimit  -a -HS
    > /etc/security/ulimits.d/
<!-- -->


# 日志
* syslog.h规范类型
    * 0      ：kern(kernel)：内核日志，大都为硬件检测与内核功能加载
    * 1      ：user：用户层信息(如logger)
    * 2      ：mail：邮件服务有关
    * 3      ：daemon：系统服务(如systemd)
    * 4      ：auth：认证与授权(如login,ssh,su)
    * 5      ：syslog：rsyslogd服务
    * 6      ：lpr：打印
    * 7      ：news：新闻组
    * 8      ：uucp：全名"Unix to Unix Copy Protocol"，早期用于unix系統间的程序资料交换
    * 9      ：cron：计划任务(如cron,at)
    * 10     ：authpriv：与auth类似，但记录较多账号私人信息，包括PAM模块
    * 11     ：ftp：与FTP协议有关
    * 16~23  ：local0 ~ local7：本地保留
<!-- -->

* syslog.h规范级别
    * 7      ：debug：除错
    * 6      ：info：基本信息说明；
    * 5      ：notice：正常通知
    * 4      ：warning(warn)：警告，可能有问题但还不至与影响到daemon
    * 3      ：err (error)   ：错误，例如配置错误导致无法启动
    * 2      ：crit：严重错误
    * 1      ：alert：警报
    * 0      ：emerg(panic)：疼痛等級，意指系統几乎要死机，通常大概只有硬件出问题导致内核无法运行
<!-- -->

* journalctl
    * -b        ：开机启动日志
    * -n        ：最近的几行日志
    * -r        ：反向，由新到旧
    * -f        ：监听
    * -t        ：类型
    * -p        ：级别
    * -S、-U    ：since与until某时刻的日志(date格式时间)
    * 指定范围：
        * -u        ：指定unit
        * `_PID=`
        * `_UID=`
        * `_COMM=`
<!-- -->

* logger  -p  user.info
<!-- -->

# systemd
* systemd-units
    * unit配置目录   ：(优先级降序)
        > /etc/systemd/system/  
        > /run/systemd/system/  
        > /lib/systemd/system/
    * unit类型       ：
        * service    ：一般服务类型
        * socket     ：监听端口
        * timer      ：定时器
        * target     ：一系列unit的集合
        > 包括        ：graphical，multi-user，basic，sysinit，rescue，emerge，shutdown，getty
    * 启动流程
        * 根据配置目录优先级，寻找default.target
        * 读取并执行default.target，递归启动各依赖units
        > * systemd无法管理手动执行启动的服务  
        > * 只有在配置目录的unit才在systemd视线里  
        > * 是否开机启动取决于满足上述的unit是否在default.target的依赖链中
<!-- -->
## units配置
* unit配置
    * 选项可重复设置，后面覆盖前面
    * bool值可为     ：1/0，yes/no，ture/false，on/off
    * #与;开头为注释
    * foo.service.wants/requires依赖目录
    * 若unit名字为foo@bar.service，且目标不存在，则使用foo@.service(模板)，配置中%I为bar
<!-- -->

### [Unit]
* [Unit]
    > 指定条件下开启或关闭自己或其他Units
    * Description    ：简介
    * Documentation  ：提供详细信息，接受 "http://", "https://", "file:///", "info:///", "man:///" 五种URI类型
    * After/Before   ：同时开启/关闭时确定先后顺序，待所有状态完全确定后才算完成
        > 启动先则关闭后，但当一启动一关闭则先关后开(无先后顺序关系则独立启动与关闭)
    * 同时启动依赖Units
        * Wants(fail、dead都行)
        * Requires(fail不行、dead行)
        * BindsTo(fail、dead都不行)
        > 依赖关系是同时启动而非先后顺序启动  
        > 当条件检测失败时为dead而非fail
    * Requisite(只检查，不启动，检查失败则fail)
    * PartOf(跟着同时关闭或重启)
    * Conflicts(不能同时存在)
<!-- -->

### [Install]
* [Install]
    > 只在enable与disable时使用
    * Alias  ：实质便是在递归启动链中创建符号链接
        > 别名会随着单元的enable与disable而生效和失效
    * WantedBy/RequiredBy    ：空格分隔的unit列表
        > 加入到.wants/.requires目录中
    * Also   ：空格分隔单元列表
        > enable与disable它时也对列表中单元进行相同操作
<!-- -->

### [Service]
* [Service]
    * Type
        * simple
        * exec
            > simple表示当fork()返回时即算启动完成，而 exec 则表示仅在 fork()与execve()都执行成功时才算是启动完成；  
            > 这意味着对于exec类型的服务来说， 如果不能成功调用主服务进程(例如User=不存在或者二进制可执行文件不存在)， 那么systemctl start将会执行失败
        * oneshot(主进程退出后才算完成，直接从activating到inactive)
        * notify(要等待返回状态信息后才算完成)
        * forking(父进程退出且至少有一个子进程才算完成，应该设置PIDFile=以跟踪主进程)
        * dbus(从D-Bus获得名称后算完成，需设置BusName选项)
        * idle(所有任务完成后才启动，最多延迟5秒)
    * ExecStart
        * 只有Type=oneshot才能有多个命令行；
        * 绝对路径前-        ：失败退出被视为成功
        * 绝对路径前+        ：root用户权限
    * ExecPre/ExecPost
        * 可设置多个，顺序执行，若未设置-，则失败后后续不执行
    * ExecStop/ExecReload
    * RemainAfterExit        ：正常退出后仍为active，并无法再次启动
    * Restart
        * no，on-success，on-failure，on-abnormal，on-watchdog，on-abort，always
        * 正常退出、异常退出、被杀死、超时的时候是否重启，不包括手动关闭
    * RestartSec       ：默认单位秒
    * SuccessExitStatus：设置额外正常退出状态，可包括信号
    * KillMode
        * process      ：仅杀死主进程
        * control-group：杀死cgroup中所有进程
        * none         ：仅执行ExecStop
    * User
    * Group
<!-- -->

### [Timer]
* [Timer]
    * OnBootSec              ：相对内核启动
    * OnStartupSec           ：相对systemd启动
    * OnActiveSec            ：相对该timer启动
    * OnUnitActiveSec        ：相对匹配unit最后一次启动
    * OnUnitInactiveSec      ：相对匹配单元最后一次关闭
    * OnCalendar=(星期) 年-月-日 时-分-秒 (时区)
        * 星期       ：Mon，Tue，Wed，Thu，Fri，Sat，Sun
        * 主子部分   ：
            * `,`表示指定
            * `*`表示任一
            * `/N`表示指定的间隔为N跳跃，后面的数也算
            * `..`表示范围
            * `月~日`表示月中倒数第几天
            * 秒可以为小数
    * AccuracySec    ：设置精度，默认一分钟
    * Persistent     ：是否操作OnCalendar不错过
    * WakeSystem     ：是否到时唤醒系统
    * Unit           ：指定匹配unit，默认同名.service
    * 注             ：时间单位us ms s m h d w M y
<!-- -->

### [Socket]
* [Socket]
    * ListenStream：监听的port或`(IP/)port`，TCP
    * ListenDatagram：监听的port或`(IP/)port`，UDP
    * ListenSequentialPacket：监听UNIX socket
    * FreeBind=yes：在指定IP可用前监听它， 出于健壮性考虑， 当你希望将套接字绑定到特定的IP地址时应该设置此项，否则网络未及时启动时socket将无法启动
    * Accept=yes：为每个连接都产生实例
    * 注：可能希望在foo.service中[Unit]设置Conflicts=foo.socket
<!-- -->

## 命令
* systemctl
    * 信息查看
        * status
        * show
        * cat
        * edit
    * unit查看
        * --state=
        * list-units  ：已加载的
        * -t          ：指定unit类型
        * -a          ：所有
        * list-unit-files    ：所有
        * list-dependencies [--reverse]
    * 手动控制
        * start
        * stop
        * restart
        * reload
    * 开机管理
        * enable|disable      ：可使用--now同时执行start/stop
        * static      ：只能作为依赖被启动
        * mask        ：禁止启动，unmask解禁
    * 主机状态
        * suspend
        * hibernate
        * rescue
        * emergency
    * 主机target
        * get-default
        * set-default
        * isolate
    * systemd重新读取unit
        * daemon-reload
<!-- -->

* systemd-analyze
    * systemd-analyze blame
    * systemd-analyze plot > plot.svg
* systemd-escape --path ：路径转义
<!-- -->

# GRUB
## GRUB配置
* /etc/default/grub
```
    GRUB_DEFAULT=0
    GRUB_TIMEOUT=5
    GRUB_GFXMODE=auto
    GRUB_GFXPAYLOAD_LINUX=keep
    GRUB_CMDLINE_LINUX-DEFAULT=
```
* /etc/grub.d/
```
    * 00_header
    * 01_user     ：自定义环境
    * 10_linux    ：确定linux选单
    * 20_os-prober：确定其他OS选单
    * 40_custom   ：自定义选单
```
<!-- -->

## GRUB Shell
* 模块   ：默认自动加载command.lst与crypto.lst
* 命令规则       ：
    * 分区       ：(hd0，gpt1)
    * 文件       ：(hd0，gpt1)/path/to/file
    * 扇区       ：(hd0，gpt1)0+1
* 特殊变量
    * prefix     ：grub安装目录
    * root       ：根设备，未指定设备名的文件的默认设备
    * cmdpath    ：core.image所在目录
    * superusers ：超级用户，逗号分隔
* grub命令
    * ls                    ：列出已知设备/设备中的文件/目录的内容
    * cat                   ：显示文件内容，--dos选项处理换行符
    * echo                  ：与bash用法一样
    * normal                ：执行命令脚本
    * source                ：将文件内容插入当前位置
    * configfile            ：将文件做配置加载，不会保留其设置的环境变量
    * set var=val           ：设置变量
    * export var            ：导出至环境变量，使其对configfile命令载入的配置文件可见
    * lsmod                 ：列表已加载模块
    * insmod/rmmod          ：加载/卸载模块
    * loopback dev isofile  ：建立loop设备，-d删除
    * halt/reboot           ：关机/重启
* GRUB安全
    * 设置超级用户
        set superusers="root"
        * 注 ：设置后只有超级用户才能修改选单
    * 设置加密密码
        password_pbkdf2 root grub.pbkdf2.sha512...
        * 注 ：使用grub-mkpasswd-pbkdf2命令产生密码
    * 设置明文密码
        password user ...
    * menuentry选项
        * --unrestricted    ：所有人可执行
        * --users ""        ：仅超级用户
        * --users "user"    ：仅user与超级用户
<!-- -->

## GRUB选单
* menuentry
    * "title"    ：选单名
    * --class    ：选单主题样式
    * --id       ：赋值chosen，覆盖原来的"title"
    * 语句块
        * linux       ：加载内核
        * initrd      ：加载内核映像
        * boot        ：启动已加载的os或loader，选单结束时隐含
        * chianloader ：链式加载文件
<!-- -->

## 系统救援
* 系统救援
    * 修改选单内核参数为rd.break，并chroot
        > 注 ：rd.break模式下无SELinux，修改密码会导致其安全上下文失效而导致无法登陆
    * 修改选单内核参数为systemd-unit=rescu.target
<!-- -->

* grub-install
    * --target=x86_64-efi
    * --efi-directory=/boot/efi
    * --bootloader-id=GRUB
<!-- -->

* grub-mkconfig  -o  /boot/grub/grub.cfg
<!-- -->

# 主机配置
* 主机配置
    * /etc/shells           ：可用shell
    * /etc/services         ：服务名与端口对照
    * /etc/hostname         ：主机名
    * /etc/hosts            ：已知主机名
    * /etc/resolv.conf      ：DNS服务器
    * /etc/locale.conf      ：语系与字符集
    * /etc/localtime        ：本地时区/usr/share/zoneinfo/
    * /etc/adjtime          ：系统时间校准类型
<!-- -->

* FONT
    * mkfontdir
    * mkfontscale
    * fc-cache -f
    > 以上三条命令为字体安装三部曲
    * fc-list       ：字体缓存查看

* hostnamectl
    * set-hostname
<!--  -->

* locale [-a]
* localectl
<!-- -->

* timedatectl
    * set-timezone
    * set-local-rtc
    * set-ntp     ：chronyd服务
<!-- -->

* date
    * +*timeformat*
        > * `%Y %y`         ：年份、年份后两位
        > * `%m %b %B`      ：月份、月份单词缩写、月份单词
        > * `%d %j %s`      ：一月中第几天、一年中第几天、从epoch开始的秒数
        > * `%H %M %S`      ：时（24小时值）、分、秒
        > * `%a %A`         ：礼拜单词缩写、礼拜单词
    * -d *timeformat*       ：将参数转换为时间
    * -d @`N`               ：epoch之后N秒
    * -d '19700101  Ndays'  ：epoch之后N天
<!-- -->

* cal *[MONTH YEAR]*
<!-- -->

# 计算机网络
* OSI七层模型与TCP/IP四层模型

| TCP/IP | OSI                    |
|--------|------------------------|
| 应用层 | 应用层、会话层、表现层 |
| 传输层 | 传输层                 |
| 网络层 | 网络层                 |
| 链路层 | 数据链路层、物理层     |

* TCP三次握手与四次挥手
* 网络协议栈
    * 发送：
        * 应用层调用socket接口函数，发送message
        * 传输层确认socket的协议与端口号，并封装为segment
        * 网络层利用路由表确认参数(协议、目的IP、本地IP等)，分片并封装为packet，若目的IP为本地则直接上传
            > 将目的IP与本地子网掩码进行位与运算，得到网络号，再与路由表对比，收包时同理
        * 链路层通过驱动程序，利用ARP表确认目的MAC地址，若不存在则发送ARP请求，最后封装成frame，然后发送到物理网卡进行信号转换并传输
    * 接收：
        * 链路层中网卡默认会将MAC地址非本机且非广播的frame丢弃，若为ARP包则本层(驱动)解决
        * 网络层确认目的IP是否为本机IP或本机网段广播，若为ICMP则本层解决
        * 传输层确认socket的协议与端口，上传给应用层
        * 应用层控制会话的挂断
* 交换机
    * 将端口与frame的源MAC地址进行绑定(一个端口可绑定多个MAC地址，反之不行)
    * 若arp表中无frame的目的MAC地址则泛洪(在其他端口转发原frame)
    * 二层交换机配置有IP地址与默认路由，管理员远程管理，目的MAC地址与目的IP地址都为交换机才行
* 路由器
    * 检查目的MAC地址，判断是否接收
    * 检查目的IP地址，判断接收、转发或丢弃
    * 转发时可能需要NAT
* 防火墙(内核钩子)
![图片来自网络](../images/netfilter.jpg)
* 虚拟网卡与虚拟网桥
    * 物理网卡接收的包发送到虚拟网桥，其通过类似交换机原理转发包，且虚拟网桥自身有IP与MAC
    * 虚拟网卡接收到虚拟网桥转发的数据包，并开始解包流程
* VPN
    * 与远程主机连接形成VLAN，通过建立隧道和修改路由，将某目的网络/主机的路由修改到隧道另一端
    * 数据包封装了访问请求，由VPS转发
* 代理
    * 正向代理   ：客户端设置代理后，所有消息发送至代理服务器并由其修改源IP后转发
    * 反向代理   ：服务器设置代理后，客户端连接代理服务器，由代理服务器分配连接到真正服务器
<!-- -->

## 网络配置
* ip --color
    * link/l
        * set IF up/down
        * set name NAME
        * set promisc
    * addr/a
        * add IP/MASK  broadcast +  dev IF
        * del IP/MASK  dev IF
    * route/r
        * add IP/MASK  via GWIP  dev IF  src IP  metric 600
        * add [throw|unreachable|prohibit|blackhole] IP/MA
        * del IP/MASK  via GWIP  dev IF
    * neigh/n
        * add/del IP  lla MAC  dev IF
<!-- -->

## NetworkManager服务
* nmcli
    * radio/r
        * wifi on|off
    * connection/c
        * up  CON-NAME
        * add type . ifname . ssid . con-name .
        * del SSID
        * reload
    * device/d
        * wifi
        * dis IF
        * wifi c SSID  password PASSWD  (hidden yes)
* nmtui
<!-- -->

## 其它网络命令
* nmap
![图片来自网络](../images/nmap.png)
<!-- -->

* ss
    * -at        ：TCP端口
    * -atn       ：TCP端口，指定端口号
    * -au        ：UDP端口
    * -ax        ：UNIX类型socket
* ping
<!-- -->

* pacman
    * 更新数据库
        * -Sy：同步源
            > 传入两次--refresh或-y将强制更新所有软件包列表，即使系统认为它们已经是最新。每次修改镜像之后都应该使用pacman -Syyu。
    * 查询软件包
        * -Ss：模糊搜索远程数据库
        * -Si：从远程数据库获取目的包的详细信息
        * -Sl：列出目的仓库所有包
        * -Qs：模糊搜索本地数据库
        * -Qi：从本地数据库获取目的包的详细信息
        * -Qc：查看目的包更新日志
        * -Qg：查看软件包组中的包
        * -Ql：查看目的包的文件安装路径
        * -Qo：查询目的文件所属包
        * -Qu：查询所有需要升级的包
    * 安装软件包
        * -S：远程下载并安装，若以安装则用本地包重装
        * -Sw：只下载软件包
        * -U：安装已下载的软件包
        * -Su：更新软件包
    * 删除软件包
        缓存位于：/var/cache/pacman/pkg/
        * -R：删除目的包
        * -Rc：还包括依赖它的包
        * -Rs：还包括只被它依赖的包
        * -Rn：还包括其配置文件
        * -Sc：清理未安装的软件包
        * -Scc：清理所有软件包与数据库
    * 从iso文件安装软件包
        * mount  -o  ro,loop  /path/to/iso  /mnt/iso
        * /etc/pacman.conf
        [custom]     #添加在其他仓库之前
        Server = file:///mnt/iso/arch/pkg
        * pacman  -Sy
<!-- -->

* curl -o *File* *URL*
* curl -fsSL *URL* | bash
* curl -fsSL *URL* | bash -s -- {-opt}
* curl cheat.sh/`CMD`
* curl cheat.sh/`LANG`/`SPECIFIC`
    > [cheat.sh](https://github.com/chubin/cheat.sh)是github上一个nice的项目  
    * 空白用`+`代替
    * `curl cheat.sh/~keyword`
    * `curl cheat.sh/python/:list` 列出可选项
<!-- -->

* sendEmail
    * -s        ：SMTP服务器
    * -f        ：发送者的邮箱
    * -t        ：接收者的邮箱
    * -cc       ：表示抄送发给谁
    * -bcc      ：表示暗抄送给谁
    * -xu       ：SMTP验证的用户名
    * -xp       ：SMTP验证的密码
    * -u        ：标题
    * -m        ：内容
    * -a        ：附件
    * -o message-content-type=*html*/*text* ：邮件的格式
    * -o message-charset=utf8               ：邮件的编码
<!-- -->

# 基础命令
## 读取文件信息
* stat
    * `默认`    ：显示文件详细信息
    * -f        ：显示文件所处文件系统信息
<!-- -->

* ls
    * -d        ：显示目录本身
    * -i        ：显示inode
    * -L        ：显示符号链接的目标
    * -n        ：显示uid与gid
    * -R        ：目录递归显示
    * -s        ：显示文件占用block大小
    * -S        ：按大小排序（降序）
    * -t        ：按modify时间排序（降序）
    * -Z        ：安全上下文
<!-- -->

* du
    * -sh       ：显示目标占用block大小
<!-- -->

* file
    * -i        ：文件的格式信息
<!-- -->

* cat
    * -A        ：打印特殊空白符
    * -n        ：显示行号
    * -b        ：显示行号但空行不算行号
* tac：翻转首尾
* rev：每行翻转
<!-- -->

* head
    * -v        ：显示文件名
    * -n        ：指定显示多少行
<!-- -->

* tail
    * -v        ：显示文件名
    * -n        ：指定显示多少行
    * -f        ：监听
<!-- -->

* xxd
    > 以十六进制读取文件数据

## 文件的修改、创建与删除
* touch
    * `默认`    ：修改a/m/ctime
    * -a        ：只改atime
    * -m        ：只改mtime
    * -d        ：指定touch的时间，date模式
<!-- -->

* ln *SRC* *TAG*
    * `默认`    ：硬连接
    * -s        ：软连接
    * -f        ：强制覆盖
<!-- -->

* cp  *SRC*  *TAG*
    * -a        ：尽可能复制所有信息
    * -r        ：目录递归
    * -v        ：详述
    * -i        ：询问是否覆盖
    * -u        ：只更新
    * -f        ：强制覆盖
    * -n        ：直接不覆盖
<!-- -->

* mv *SRC*  *TAG*
    * -v        ：详述
    * -i        ：询问是否覆盖
    * -u        ：只更新
    * -f        ：强制覆盖
    * -n        ：直接不覆盖
<!-- -->

* rm
    * -r        ：目录递归
    * -v        ：详述
    * -f        ：强制覆盖
<!-- -->

* mkdir
    * -p        ：递归
    * -m        ：设置权限
<!-- -->

## 文件的搜索
* whereis       ：搜索可执行，头文件和帮助信息的位置，使用系统内建数据库
<!-- -->

* locate
    * -i        ：忽略大小写
    * -r        ：正则表达式（默认通配符）
    * -c        ：显示匹配的文件数量
    * -S        ：显示数据库信息
* updatedb
    > 更新locate命令需要的数据库  
    > 配置  ：/etc/updatedb.conf  
    > 数据库：/var/lib/mlocate/mlocate.db  
<!-- -->

* find *DIR* *OPTS...*
    * 打印：
        * -print
    * 正则名字：
        * -regex
        * -iregex
    * 通配符名字：
        * -name
        * -iname
    * 用户：
        * -uid
        * -gid
        * -user
        * -group
        * -nouser
    * 权限：
        * -writable
        * -readable
        * -executable
        * -perm
            > `-`表示权限为指定范围的非空子集  
            > `/`表示权限为指定范围的非空交集  
            > `无前缀`表示精准权限
    * 大小：
        * -size
            > `+`表示大于  
            > `-`表示小于  
            > 单位用c/k/M/G
    * 时间：
        * -[a|m|c]time *n*  ：指定`n*24 h`之前的文件，`+/-`表示更早/更晚
        * -[a|m|c]min  *n*  ：指定`n min`之前的文件
        * -[a|m|c]newer *file*
    * 文件类型：
        * -type [f|d|l|b|c|p|s]
    * 其他信息：
        * -links            ：硬连接数
        * -inum             ：inode号
    * 复合逻辑关系          ：-a、-o、-not
    * 对搜索到的文件执行CMD ：-exec  *CMD*  {} \;
<!--  -->

* fzf   ：交互式模糊搜索
<!-- -->

## 其它
* echo
    * -n        ：不自动加入换行符（zsh会将无换行结尾的输出的尾部标记`反显的%`）
    * -e        ：启用转义语义（zsh自动开启）
<!-- -->


* pwd
    * -P        ：显示真实路径而非软连接
<!-- -->

* bc
    * -l        ：可以使用数学库函数 s(sin x)，c(cos x)，a(arctan x)，l(ln x)，e(e^x)
    * 特殊变量  ：scale，last，ibase，obase，支持^运算符求幂
<!-- -->

* split *FILE* *Preffix*
    > 注：preffix最后最好加dot，大小单位可指定，默认byte
    * -b        ：按大小分割
    * -l        ：按行数分割
<!-- -->

* iconv  FILE
    * -f        ：原字符集
    * -t        ：目标字符集
    * -o        ：输出文件
    * --list    ：列出可选字符集
<!-- -->

* col -x ：将tab替换为等宽space，该命令只从stdin读取
<!-- -->

* diff -Naur *OLD* *NEW* > *.patch
* patch -p`n`  < *.patch
    > * new和old不要在同一目录下  
    > * n为去掉的/个数
<!-- -->

