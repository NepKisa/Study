# Linux

[TLCL (billie66.github.io)](https://billie66.github.io/TLCL/book/chap01.html)

## 进程管理

### Job

* jobs：查看后台进程
* &：将命令放到后台执行
* fg %【job id】：将停止/后台job放到前台执行
* bg %【job id】：将停止的进程放到后台继续执行
* Ctrl+Z：停止job，后续可用jobs查询，拉到前台或后台继续执行

### Kill

killall可中断用户的全部进程

```perl
kill -l
```

常用：

* 1：重启
* 2：Ctrl+C
* 9：内核级终止
* 15：终止
* 18：继续
* 19：停止

| 编号 | 名字 | 含义                                                         |
| :--- | :--- | :----------------------------------------------------------- |
| 1    | HUP  | 挂起（Hangup）。名字来源于很久以前，那时候终端机通过电话线和调制解调器连接到 远端的计算机。这个信号被用来告诉程序，控制的终端机已经“挂断”。 通过关闭一个终端会话，可以展示这个信号的作用。在当前终端运行的前台程序将会收到这个信号并终止。许多守护进程也使用这个信号，来重新初始化。这意味着，当一个守护进程收到这个信号后， 这个进程会重新启动，并且重新读取它的配置文件。Apache 网络服务器守护进程就是一个例子。 |
| 2    | INT  | 中断。实现和 Ctrl-c 一样的功能，由终端发送。通常，它会终止一个程序。 |
| 9    | KILL | 杀死。这个信号很特别。尽管程序可能会选择不同的方式来处理发送给它的 信号，其中也包含忽略信号，但是 KILL 信号从不被发送到目标程序。而是内核立即终止 这个进程。当一个进程以这种方式终止的时候，它没有机会去做些“清理”工作，或者是保存工作。 因为这个原因，把 KILL 信号看作最后一招，当其它终止信号失败后，再使用它。 |
| 15   | TERM | 终止。这是 kill 命令发送的默认信号。如果程序仍然“活着”，可以接受信号，那么 这个它会终止。 |
| 18   | CONT | 继续。在一个停止信号后，这个信号会恢复进程的运行。           |
| 19   | STOP | 停止。这个信号导致进程停止运行，而不是终止。像 KILL 信号，它不被 发送到目标进程，因此它不能被忽略。 |

| 编号 | 名字  | 含义                                                         |
| ---- | ----- | ------------------------------------------------------------ |
| 3    | QUIT  | 退出                                                         |
| 11   | SEGV  | 段错误(Segmentation Violation)。如果一个程序非法使用内存，就会发送这个信号。也就是说， 程序试图写入内存，而这个内存空间是不允许此程序写入的。 |
| 20   | TSTP  | 终端停止(Terminal Stop)。当按下 Ctrl-z 组合键后，终端发送这个信号。不像 STOP 信号， TSTP 信号由目标进程接收，且可能被忽略。 |
| 28   | WINCH | 改变窗口大小(Window Change)。当改变窗口大小时，系统会发送这个信号。 一些程序，像 top 和 less 程序会响应这个信号，按照新窗口的尺寸，刷新显示的内容。 |

### 其他

`pstree`：  输出一个树型结构的进程列表(processtree)，这个列表展示了进程间父/子关系。

`vmstat`：输出一个系统资源使用快照，包括内存，交换分区和磁盘 I/O。 为了看到连续的显示结果，则在命令名后加上更新操作延时的时间（以秒为单位）。例如，“vmstat 5”。 ，按下 Ctrl-c 组合键, 终止输出。

`xload`：一个图形界面程序，可以画出系统负载随时间变化的图形。

`tload`：  terminal load 与 xload 程序相似，但是在终端中画出图形。使用 Ctrl-c，来终止输出。

## 环境变量

### 常用命令

`printenv` ：打印部分或所有的环境变量，等同于env

`set` ：设置 shell 选项，打印出所有定义的函数

`export` ：导出环境变量，让随后执行的程序知道。可用于查看环境变量

`alias` ：创建命令别名，可以查看当前的alias

### 自动加载环境变量机制

当我们登录系统后， bash 程序启动，并且会读取一系列称为启动文件的配置脚本， 这些文件定义了默认的可供所有用户共享的 shell 环境。然后是读取更于当前用户自己家目录中 的启动文件，这些启动文件定义了用户个人的 shell 环境。确切的启动顺序依赖于要运行的 shell 会话 类型。有两种 shell 会话类型：一个是登录 shell 会话，另一个是非登录 shell 会话。

*登录 shell 会话的启动文件*

| 文件            | 内容                                                         |
| :-------------- | :----------------------------------------------------------- |
| /etc/profile    | 应用于所有用户的全局配置脚本。                               |
| ~/.bash_profile | 用户个人的启动文件。可以用来扩展或重写全局配置脚本中的设置。 |
| ~/.bash_login   | 如果文件 ~/.bash_profile 没有找到，bash 会尝试读取这个脚本。 |
| ~/.profile      | 如果文件 ~/.bash_profile 或文件 ~/.bash_login 都没有找到，bash 会试图读取这个文件。 这是基于 Debian 发行版的默认设置，比方说 Ubuntu。 |

*非登录 shell 会话的启动文件*

可以在 .bashrc 文件中定义 shell 函数实现自定义命令

| 文件             | 内容                                                         |
| :--------------- | :----------------------------------------------------------- |
| /etc/bash.bashrc | 应用于所有用户的全局配置文件。                               |
| ~/.bashrc        | 用户个人的启动文件。可以用来扩展或重写全局配置脚本中的设置。 |

## 包管理

| 发行版                                   | 底层工具 | 上层工具          |
| :--------------------------------------- | :------- | :---------------- |
| Debian-Style                             | dpkg     | apt-get, aptitude |
| Fedora, Red Hat Enterprise Linux, CentOS | rpm      | yum               |

搜索一个 yum 包仓库来查找 emacs 文本编辑器，使用以下命令：

```perl
yum search emacs
```

安装

```perl
yum install 【包名】
或
rpm -ivh 【emacs-22.1-7.fc7-i386.rpm】
```

在 Red Hat 系统中，查看哪个软件包安装了/usr/bin/vim 这个文件

```perl
rpm -qf /usr/bin/vim
```

确定是否安装了一个软件包

```perl
rpm -qa|grep 【包名】
```

升级包

```perl
yum update
或
rpm -U 【新版本包名】
```

## 存储

- mount – 挂载一个文件系统

- umount – 卸载一个文件系统

- fsck – 检查和修复一个文件系统

- fdisk – 分区表控制器

- mkfs – 创建文件系统

- fdformat – 格式化一张软盘

- dd — 把块数据直接写入设备

    ```perl
    #生成一个1000M的文件 文件名为1
    dd if=/dev/zero of=1 bs=100M count=10
    ```

- genisoimage (mkisofs) – 创建一个 ISO 9660的映像文件

- wodim (cdrecord) – 把数据写入光存储媒介

- md5sum – 计算 MD5检验码

划分一块30G的磁盘

```perl
lvs
lvcreate -n oracle -L 30G datavg
mkfs.xfs /dev/mapper/datavg-oracle
mkdir /oracle
mount /dev/mapper/datavg-oracle /oracle
cat << EOF >> /etc/fstab
/dev/mapper/datavg-oracle	/oracle	xfs	defaults	0	0
EOF
```

*find命令*

```perl
find ~ -type f -name "*.JPG" -size +1M | wc -l

#文件正确的权限一般是0600
find ~ ( -type f -not -perms 0600 ) -or ( -type d -not -perms 0700 )

#显示所有子目录文件
find .
```

| 文件类型 | 描述             |
| :------- | :--------------- |
| b        | 块特殊设备文件   |
| c        | 字符特殊设备文件 |
| d        | 目录             |
| f        | 普通文件         |
| l        | 符号链接         |

| 选项             | 描述                                                         |
| :--------------- | :----------------------------------------------------------- |
| -depth           | 指示 find 程序先处理目录中的文件，再处理目录自身。当指定-delete 行为时，会自动 应用这个选项。 |
| -maxdepth levels | 当执行测试条件和行为的时候，设置 find 程序陷入目录树的最大级别数 |
| -mindepth levels | 在应用测试条件和行为之前，设置 find 程序陷入目录数的最小级别数。 |
| -mount           | 指示 find 程序不要搜索挂载到其它文件系统上的目录。           |
| -noleaf          | 指示 find 程序不要基于自己在搜索 Unix 的文件系统的假设，来优化它的搜索。 在搜索DOS/Windows 文件系统和CD/ROMS的时候，我们需要这个选项 |

```perl
#查看文件状态
stat 【文件】
```

## 网络

- ping - 发送 ICMP ECHO_REQUEST 数据包到网络主机
- traceroute - 打印到一台网络主机的路由数据包
- netstat - 打印网络连接，路由表，接口统计数据，伪装连接，和多路广播成员
- ftp - 因特网文件传输程序
- wget - 非交互式网络下载器
- ssh - OpenSSH SSH 客户端（远程登录程序）

### ping

```perl
ping linuxcommand.org
```

按下组合键 Ctrl-c，中断这个命令之后，ping 打印出运行统计信息。一个正常工作的网络会报告 零个数据包丢失。一个成功执行的“ping”命令会意味着网络的各个部件（网卡，电缆，路由，网关） 都处于正常的工作状态。

### traceroute

这个 `traceroute` 程序（一些系统使用相似的 `tracepath` 程序来代替）会显示从本地到指定主机 要经过的所有“跳数”的网络流量列表。例如，看一下到达 slashdot.org 需要经过的路由， 我们将这样做：

```perl
traceroute slashdot.org
```

```shell
traceroute to slashdot.org (216.34.181.45), 30 hops max, 40 byte
packets
1 ipcop.localdomain (192.168.1.1) 1.066 ms 1.366 ms 1.720 ms
2 * * *
3 ge-4-13-ur01.rockville.md.bad.comcast.net (68.87.130.9) 14.622
ms 14.885 ms 15.169 ms
4 po-30-ur02.rockville.md.bad.comcast.net (68.87.129.154) 17.634
ms 17.626 ms 17.899 ms
5 po-60-ur03.rockville.md.bad.comcast.net (68.87.129.158) 15.992
ms 15.983 ms 16.256 ms
6 po-30-ar01.howardcounty.md.bad.comcast.net (68.87.136.5) 22.835
...
```

从输出结果中，我们可以看到连接测试系统到 slashdot.org 网站需要经由16个路由器。对于那些 提供标识信息的路由器，我们能看到它们的主机名，IP 地址和性能数据，这些数据包括三次从本地到 此路由器的往返时间样本。对于那些没有提供标识信息的路由器（由于路由器配置，网络拥塞，防火墙等 方面的原因），我们会看到几个星号，正如行中所示。

### netstat

netstat 程序被用来检查各种各样的网络设置和统计数据。通过此命令的许多选项，我们 可以看看网络设置中的各种特性。使用“-ie”选项，我们能够查看系统中的网络接口

```perl
netstat -ie
```

```bash
eth0    Link encap:Ethernet HWaddr 00:1d:09:9b:99:67
        inet addr:192.168.1.2 Bcast:192.168.1.255 Mask:255.255.255.0
        inet6 addr: fe80::21d:9ff:fe9b:9967/64 Scope:Link
        UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
        RX packets:238488 errors:0 dropped:0 overruns:0 frame:0
        TX packets:403217 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:100 RX bytes:153098921 (146.0 MB) TX
        bytes:261035246 (248.9 MB) Memory:fdfc0000-fdfe0000

lo      Link encap:Local Loopback
        inet addr:127.0.0.1 Mask:255.0.0.0
```

当执行日常网络诊断时，要查看的重要信息是每个网络接口第四行开头出现的单词 “UP”，说明这个网络接口已经生效。

还要查看第二行中 inet addr 字段出现的有效 IP 地址。对于使用 DHCP（动态主机配置协议）的系统，在 这个字段中的一个有效 IP 地址则证明了 DHCP 工作正常。

```perl
netstat -r
```

```bash
[root@sakura ~]# netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         _gateway        0.0.0.0         UG        0 0          0 eth0
10.119.118.0    0.0.0.0         255.255.255.0   U         0 0          0 eth0
10.233.90.0     node1           255.255.255.0   UG        0 0          0 tunl0
10.233.96.0     node2           255.255.255.0   UG        0 0          0 tunl0
10.233.104.0    0.0.0.0         255.255.255.0   U         0 0          0 *

10.233.104.3    0.0.0.0         255.255.255.255 UH        0 0          0 calica63e4ac615
10.233.104.7    0.0.0.0         255.255.255.255 UH        0 0          0 cali6e5d8329069
10.233.104.8    0.0.0.0         255.255.255.255 UH        0 0          0 calic117dc2a928
10.233.104.10   0.0.0.0         255.255.255.255 UH        0 0          0 calif930b762229
10.233.104.11   0.0.0.0         255.255.255.255 UH        0 0          0 cali82eaa1a13b7
10.233.104.12   0.0.0.0         255.255.255.255 UH        0 0          0 cali02a89b808dc
10.233.104.14   0.0.0.0         255.255.255.255 UH        0 0          0 cali04ccbc4a27b
10.233.104.15   0.0.0.0         255.255.255.255 UH        0 0          0 cali4d239ec8772
10.233.104.17   0.0.0.0         255.255.255.255 UH        0 0          0 cali51eab428476
10.233.104.18   0.0.0.0         255.255.255.255 UH        0 0          0 cali5f67826aa57
10.233.104.20   0.0.0.0         255.255.255.255 UH        0 0          0 cali4d9ee2d6858
10.233.104.21   0.0.0.0         255.255.255.255 UH        0 0          0 cali8f831ffe298
10.233.104.22   0.0.0.0         255.255.255.255 UH        0 0          0 calia53e2b47aed
10.233.104.24   0.0.0.0         255.255.255.255 UH        0 0          0 cali6548ff0cb53
10.233.104.25   0.0.0.0         255.255.255.255 UH        0 0          0 calid14cfd4d550
10.233.104.26   0.0.0.0         255.255.255.255 UH        0 0          0 cali45a05d8132b
10.233.104.27   0.0.0.0         255.255.255.255 UH        0 0          0 cali755edb2fdfd
10.233.104.28   0.0.0.0         255.255.255.255 UH        0 0          0 cali039e944a459
10.233.104.29   0.0.0.0         255.255.255.255 UH        0 0          0 calia62b81a87fb
10.233.104.30   0.0.0.0         255.255.255.255 UH        0 0          0 cali3c5571c5e8f
10.233.104.33   0.0.0.0         255.255.255.255 UH        0 0          0 cali8d93573621f
10.233.104.38   0.0.0.0         255.255.255.255 UH        0 0          0 cali917bc503532
10.233.104.39   0.0.0.0         255.255.255.255 UH        0 0          0 cali778d36cbb72
10.233.104.40   0.0.0.0         255.255.255.255 UH        0 0          0 cali897e5ae48bb
10.233.104.42   0.0.0.0         255.255.255.255 UH        0 0          0 califc9ac8ee529
10.233.104.80   0.0.0.0         255.255.255.255 UH        0 0          0 cali94a7afa8de6
10.233.104.242  0.0.0.0         255.255.255.255 UH        0 0          0 cali861bb69fb54
10.233.104.244  0.0.0.0         255.255.255.255 UH        0 0          0 calid6c8e8aadc6
10.233.104.245  0.0.0.0         255.255.255.255 UH        0 0          0 calieb6d0193819
10.233.104.246  0.0.0.0         255.255.255.255 UH        0 0          0 cali81ecb4d7f40
10.233.104.247  0.0.0.0         255.255.255.255 UH        0 0          0 caliebad5c248e8
10.233.104.248  0.0.0.0         255.255.255.255 UH        0 0          0 cali672484fc20e
10.233.104.249  0.0.0.0         255.255.255.255 UH        0 0          0 cali64bdfd0e8ed
10.233.104.250  0.0.0.0         255.255.255.255 UH        0 0          0 calic4868372733
10.233.104.252  0.0.0.0         255.255.255.255 UH        0 0          0 cali1e17a420e15
10.233.104.253  0.0.0.0         255.255.255.255 UH        0 0          0 calid0d1660371b
10.233.104.254  0.0.0.0         255.255.255.255 UH        0 0          0 caliad1d971c5d6
10.233.104.255  0.0.0.0         255.255.255.255 UH        0 0          0 cali12eeb0d52aa
link-local      0.0.0.0         255.255.0.0     U         0 0          0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
```

使用这个“-r”选项会显示内核的网络路由表。这展示了系统是如何配置网络之间发送数据包的。

位于防火墙之内的局域网中，一台客户端计算机的典型路由表。 第一行显示了目的地 192.168.1.0。IP 地址以零结尾是指网络，而不是独立主机， 所以这个目的地意味着局域网中的任何一台主机。下一个字段，Gateway， 是网关（路由器）的名字或 IP 地址，用它来连接当前的主机和目的地的网络。 若这个字段显示一个星号，则表明不需要网关。

## 归档和备份

文件压缩程序：

- gzip – 压缩或者展开文件
- bzip2 – 块排序文件压缩器

归档程序：

- tar – 磁带打包工具
- zip – 打包和压缩文件

还有文件同步程序：

- rsync – 同步远端文件和目录

## 编译

那么为什么要编译软件呢？ 有两个原因：

* 可用性。尽管系统发行版仓库中已经包含了大量的预编译程序，但是一些发行版本不可能包含所有期望的应用。 在这种情况下，得到所期望程序的唯一方式是编译程序源码。

* 及时性。虽然一些系统发行版专门打包前沿版本的应用程序，但是很多不是。这意味着， 为了拥有一个最新版本的程序，编译是必需的。

 .c 文件包含了由该软件包提供的两个 C 程序（style 和 diction），被分割成模块。这是一种常见做法，把大型程序 分解成更小，更容易管理的代码块。源码文件都是普通文本，可以用 less 命令查看

 .h 文件被称为头文件。它们也是普通文件。头文件包含了程序的描述，这些程序被包括在源码文件或库中。 为了让编译器链接到模块，编译器必须接受所需的所有模块的描述，来完成整个程序。在 diction.c 文件的开头附近， 我们看到这行代码

```c
#include "getopt.h"
```

当它读取 diction.c 中的源码的时候，这行代码指示编译器去读取文件 getopt.h， 为的是“知道” getopt.c 中的内容。 getopt.c 文件提供由 style 和 diction 两个程序共享的例行程序。

在 getopt.h 的 include 语句上面，我们看到一些其它的 include 语句，比如这些

```c
#include <regex.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
```

这些文件也是头文件，但是这些头文件在当前源码树的外面。它们由操作系统供给，来支持每个程序的编译。 如果我们看一下 /usr/include 目录，能看到它们

当我们安装编译器的时候，这个目录中的头文件会被安装。

### 构建程序

大多数程序通过一个简单的，两个命令的序列构建：

```bash
./configure
make
```

这个 configure 程序是一个 shell 脚本，由源码树提供。它的工作是分析程序构建环境。大多数源码会设计为可移植的。 也就是说，它被设计成能够在不止一种类 Unix 系统中进行构建。但是为了做到这一点，在建立程序期间，为了适应系统之间的差异， 源码可能需要经过轻微的调整。configure 也会检查是否安装了必要的外部工具和组件。让我们运行 configure 命令。 因为 configure 命令所在的位置不是位于 shell 通常期望程序所呆的地方，我们必须明确地告诉 shell 它的位置，通过 在命令之前加上 ./ 字符，来表明程序位于当前工作目录

configure 将会输出许多信息，随着它测试和配置整个构建过程。当结束后，输出结果看起来像这样：

```bash
checking libintl.h presence... yes
checking for libintl.h... yes
checking for library containing gettext... none required
configure: creating ./config.status
config.status: creating Makefile
config.status: creating diction.1
config.status: creating diction.texi
config.status: creating diction.spec
config.status: creating style.1
config.status: creating test/rundiction
config.status: creating config.h
```

这里最重要的事情是没有错误信息。如果有错误信息，整个配置过程失败，然后程序不能构建直到修正了错误。

我们看到在我们的源码目录中 configure 命令创建了几个新文件。最重要一个是 Makefile。Makefile 是一个配置文件， 指示 make 程序究竟如何构建程序。没有它，make 程序就不能运行。Makefile 是一个普通文本文件，所以我们能查看它

这个 make 程序把一个 makefile 文件作为输入（通常命名为 Makefile），makefile 文件 描述了包括最终完成的程序的各组件之间的关系和依赖性。

makefile 文件的第一部分定义了变量，这些变量在该 makefile 后续章节中会被替换掉。例如我们看看这一行代码：

```c
CC=                 gcc
```

其定义了所用的 C 编译器是 gcc。文件后面部分，我们看到一个使用该变量的实例：

```c
diction:        diction.o sentence.o misc.o getopt.o getopt1.o
                $(CC) -o $@ $(LDFLAGS) diction.o sentence.o misc.o \
                getopt.o getopt1.o $(LIBS)
```

这里完成了一个替换操作，在程序运行时，$(CC) 的值会被替换成 gcc。大多数 makefile 文件由行组成，每行定义一个目标文件， 在这种情况下，目标文件是指可执行文件 diction，还有目标文件所依赖的文件。剩下的行描述了从目标文件的依赖组件中 创建目标文件所需的命令。在这个例子中，我们看到可执行文件 diction（最终的成品之一）依赖于文件 diction.o，sentence.o，misc.o，getopt.o，和 getopt1.o都存在。在 makefile 文件后面部分，我们看到 diction 文件所依赖的每一个文件做为目标文件的定义：

```c
diction.o:       diction.c config.h getopt.h misc.h sentence.h
getopt.o:        getopt.c getopt.h getopt_int.h
getopt1.o:       getopt1.c getopt.h getopt_int.h
misc.o:          misc.c config.h misc.h
sentence.o:      sentence.c config.h misc.h sentence.h
style.o:         style.c config.h getopt.h misc.h sentence.h
```

然而，我们不会看到针对它们的任何命令。这个由一个通用目标解决，在文件的前面，描述了这个命令，用来把任意的 .c 文件编译成 .o 文件：

```c
.c.o:
            $(CC) -c $(CPPFLAGS) $(CFLAGS) $<
```

make 只是构建 需要构建的部分，而不是简单地重新构建所有的内容。

### 安装程序

打包良好的源码经常包括一个特别的 make 目标文件，叫做 install。这个目标文件将在系统目录中安装最终的产品，以供使用。 通常，这个目录是 /usr/local/bin，为在本地所构建软件的传统安装位置。然而，通常普通用户不能写入该目录，所以我们必须变成超级用户， 来执行安装操作：

`sudo make install`

```perl
[me@linuxbox diction-1.11]$ sudo make install
 [me@linuxbox diction-1.11]$ which diction
/usr/local/bin/diction
```

### 总结

==分析程序构建环境==，环境正确会生成Makefile文件给make使用，否则不生成Makefile文件无法构建

```perl
./configure
```

根据上一步生成的Makefile文件执行构建，==增量构建==（已构建的不会再次构建，删除已构建的会重新构建）

```perl
make
```

默认安装位置为==/usr/local/bin==

```perl
make install
```

以上命令可以用来构建许多源码包。我们也知道了在程序维护过程中，make 程序起到了举足轻重的作用。make 程序可以用到 任何需要维护一个目标/依赖关系的任务中，不仅仅为了编译源代码。

## Shell

可用`bash -x`或`set -x`对脚本进行debug

### if

`[ expression ]`可被双括号`[[ expression ]]`替代，显然`[[ ]]` 更 有用，并更易于编码

`[ expression ]`等同于 `test expression`

这里的 expression 是一个表达式，其执行结果是 true 或者是 false。==当表达式为真时，退出状态为0，表达式为假时，退出状态为1。==

***逻辑操作符***

| 操作符 | []   | [[ ]] and (( )) |
| :----- | :--- | :-------------- |
| AND    | -a   | &&              |
| OR     | -o   | \|\|            |
| NOT    | !    | !               |

***文件表达式***

| 表达式          | 如果下列条件为真则返回True                                   |
| :-------------- | :----------------------------------------------------------- |
| file1 -ef file2 | file1 和 file2 拥有相同的索引号（通过硬链接两个文件名指向相同的文件）。 |
| file1 -nt file2 | file1新于 file2。                                            |
| file1 -ot file2 | file1早于 file2。                                            |
| -b file         | file 存在并且是一个块（设备）文件。                          |
| -c file         | file 存在并且是一个字符（设备）文件。                        |
| -d file         | file 存在并且是一个目录。                                    |
| -e file         | file 存在。                                                  |
| -f file         | file 存在并且是一个普通文件。                                |
| -g file         | file 存在并且设置了组 ID。                                   |
| -G file         | file 存在并且由有效组 ID 拥有。                              |
| -k file         | file 存在并且设置了它的“sticky bit”。                        |
| -L file         | file 存在并且是一个符号链接。                                |
| -O file         | file 存在并且由有效用户 ID 拥有。                            |
| -p file         | file 存在并且是一个命名管道。                                |
| -r file         | file 存在并且可读（有效用户有可读权限）。                    |
| -s file         | file 存在且其长度大于零。                                    |
| -S file         | file 存在且是一个网络 socket。                               |
| -t fd           | fd 是一个定向到终端／从终端定向的文件描述符 。 这可以被用来决定是否重定向了标准输入／输出错误。 |
| -u file         | file 存在并且设置了 setuid 位。                              |
| -w file         | file 存在并且可写（有效用户拥有可写权限）。                  |
| -x file         | file 存在并且可执行（有效用户有执行／搜索权限）。            |

***字符串表达式***

| 表达式                              | 如果下列条件为真则返回True                                   |
| :---------------------------------- | :----------------------------------------------------------- |
| string                              | string 不为 null。                                           |
| -n string                           | 字符串 string 的长度大于零。                                 |
| -z string                           | 字符串 string 的长度为零。                                   |
| string1 = string2string1 == string2 | string1 和 string2 相同。 单或双等号都可以，不过双等号更受欢迎。 |
| string1 =~ regex                    | string1匹配扩展的正则表达式 regex                            |
| string1 != string2                  | string1 和 string2 不相同。                                  |
| string1 > string2                   | sting1 排列在 string2 之后。                                 |
| string1 < string2                   | string1 排列在 string2 之前。                                |

***整数表达式***

| 表达式                | 如果为真...                    |
| :-------------------- | :----------------------------- |
| integer1 -eq integer2 | integer1 等于 integer2。       |
| integer1 -ne integer2 | integer1 不等于 integer2。     |
| integer1 -le integer2 | integer1 小于或等于 integer2。 |
| integer1 -lt integer2 | integer1 小于 integer2。       |
| integer1 -ge integer2 | integer1 大于或等于 integer2。 |
| integer1 -gt integer2 | integer1 大于 integer2。       |

```shell
#!/bin/bash
# test-file: Evaluate the status of a file
FILE=~/.bashrc
if [ -e "$FILE" ]; then
    if [ -f "$FILE" ]; then
        echo "$FILE is a regular file."
    fi
    if [ -d "$FILE" ]; then
        echo "$FILE is a directory."
    fi
    if [ -r "$FILE" ]; then
        echo "$FILE is readable."
    fi
    if [ -w "$FILE" ]; then
        echo "$FILE is writable."
    fi
    if [ -x "$FILE" ]; then
        echo "$FILE is executable/searchable."
    fi
else
    echo "$FILE does not exist"
    exit 1
fi
exit
```

==通过带有一个整数参数的 return 命令，shell 函数可以返回一个退出状态。==如果我们打算把 上面的脚本转变为一个 shell 函数，为了在更大的程序中包含此函数，我们用 return 语句来代替 exit 命令， 则得到期望的行为

`[[ ]]`添加的另一个功能是`==`操作符支持类型匹配，正如路径名展开所做的那样，这就使`[[ ]]`有助于计算文件和路径名。

```shell
FILE=foo.bar
if [[ $FILE == foo.* ]]; then
    echo "$FILE matches pattern 'foo.*'"
fi
```

`(( ))`被用来执行算术真测试。如果算术计算的结果是非零值，则其测试值为真。

注意我们使用小于和大于符号，以及==用来测试是否相等而不用-eq。这是使用整数较为自然的语法了。也要 注意，因为复合命令 `(( ))` 是 shell 语法的一部分，而不是一个普通的命令，而且它只处理整数， 所以它能够通过名字识别出变量，而不需要执行展开操作。

```shell
#!/bin/bash
# test-integer2a: evaluate the value of an integer.
INT=-5
if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
    if ((INT == 0)); then
        echo "INT is zero."
    else
        if ((INT < 0)); then
            echo "INT is negative."
        else
            echo "INT is positive."
        fi
        if (( ((INT % 2)) == 0)); then
            echo "INT is even."
        else
            echo "INT is odd."
        fi
    fi
else
    echo "INT is not an integer." >&2
    exit 1
fi
```

```shell
#!/bin/bash
# test-integer3: determine if an integer is within a
# specified range of values.
MIN_VAL=1
MAX_VAL=100
INT=50
if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
    if [[ INT -ge MIN_VAL && INT -le MAX_VAL ]]; then
        echo "$INT is within $MIN_VAL to $MAX_VAL."
    else
        echo "$INT is out of range."
    fi
else
    echo "INT is not an integer." >&2
    exit 1
fi
```

**if案例：**

```shell
report_home_space () {
    if [[ $(id -u) -eq 0 ]]; then
        cat <<- _EOF_
        <H2>Home Space Utilization (All Users)</H2>
        <PRE>$(du -sh /home/*)</PRE>
_EOF_
    else
        cat <<- _EOF_
        <H2>Home Space Utilization ($USER)</H2>
        <PRE>$(du -sh $HOME)</PRE>
_EOF_
    fi
    return
}
```

### 读取键盘输入

read不可用于管道

```shell
echo “foo” | read
```

在 bash（和其它 shells，例如 sh）中，==管道线 会创建子 shell==。这个子 shell 是为了执行执行管线中的命令而创建的shell和它的环境的副本。 示例中，read 命令将在子 shell 中执行。

在类 Unix 的系统中，子 shell 执行的时候，会为进程创建父环境的副本。当进程结束 之后，该副本就会被破坏掉。这意味着==一个子 shell 永远不能改变父进程的环境==。read 赋值变量， 然后会变为环境的一部分。在上面的例子中，read 在它的子 shell 环境中，把 foo 赋值给变量

REPLY， 但是当命令退出后，子 shell 和它的环境将被破坏掉，这样赋值的影响就会消失。

为了解决我们的 read 命令问题，我们可以雇佣进程替换，像这样：

```bash
read < <(echo "foo")
echo $REPLY
```

`read [-options] [variable...]`

| 选项         | 说明                                                         |
| :----------- | :----------------------------------------------------------- |
| -a array     | 把输入赋值到数组 array 中，从索引号零开始。我们 将在第36章中讨论数组问题。 |
| -d delimiter | 用字符串 delimiter 中的第一个字符指示输入结束，而不是一个换行符。 |
| -e           | 使用 Readline 来处理输入。这使得与命令行相同的方式编辑输入。 |
| -n num       | 读取 num 个输入字符，而不是整行。                            |
| -p prompt    | 为输入显示提示信息，使用字符串 prompt。                      |
| -r           | Raw mode. 不把反斜杠字符解释为转义字符。                     |
| -s           | Silent mode. 不会在屏幕上显示输入的字符。当输入密码和其它确认信息的时候，这会很有帮助。 |
| -t seconds   | 超时. 几秒钟后终止输入。若输入超时，read 会返回一个非零退出状态。 |
| -u fd        | 使用文件描述符 fd 中的输入，而不是标准输入。                 |

如：读取用户输入密码

```shell
#!/bin/bash
# read-secret: input a secret pass phrase
if read -t 10 -sp "Enter secret pass phrase > " secret_pass; then
    echo "\nSecret pass phrase = '$secret_pass'"
else
    echo "\nInput timed out" >&2
    exit 1
fi
```

赋值多个

```perl
echo -n "Enter one or more values > "
read var1 var2 var3 var4 var5
```

```bash
Enter one or more values > a b c d e f g
var1 = 'a'
var2 = 'b'
var3 = 'c'
var4 = 'd'
var5 = 'e f g'
```

如果 read 命令接受到变量值数目少于期望的数字，那么额外的变量值为空，而多余的输入数据则会 被包含到最后一个变量中。如果 read 命令之后没有列出变量名，则一个 shell 变量，REPLY，将会包含 所有的输入

### 循环

 `break` 命令立即终止一个循环， 且程序继续执行循环之后的语句。 `continue` 命令导致程序跳过循环中剩余的语句，且程序继续执行 下一次循环

`until` 命令与 `while` 非常相似，除了当遇到一个==非零退出状态==的时候， ==while 退出循环， 而 until 不退出==。

***使用循环读取文件***

为了重定向文件到循环中，我们把重定向操作符放置到 done 语句之后。循环将使用 read 从重定向文件中读取 字段。这个 read 命令读取每个文本行之后，将会退出，其退出状态为零，直到到达文件末尾。到时候，它的 退出状态为非零数值，因此终止循环。

```shell
#!/bin/bash
# while-read: read lines from a file
while read distro version release; do
    printf "Distro: %s\tVersion: %s\tReleased: %s\n" \
        $distro \
        $version \
        $release
done < distros.txt
```

也可把标准输入管道到循环中，==管道将会在子 shell 中执行== 循环，当循环终止的时候，循环中创建的任意变量或赋值的变量都会消失，记住这一点很重要。

```shell
#!/bin/bash
# while-read2: read lines from a file
sort -k 1,1 -k 2n distros.txt | while read distro version release; do
    printf "Distro: %s\tVersion: %s\tReleased: %s\n" \
        $distro \
        $version \
        $release
done
```

for循环

```shell
for (( expression1; expression2; expression3 )); do
    commands
done
```

### case

 *case 模式实例*

| 模式         | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| a)           | 若单词为 “a”，则匹配                                         |
| [[:alpha:]]) | 若单词是一个字母字符，则匹配                                 |
| ???)         | 若单词只有3个字符，则匹配                                    |
| *.txt)       | 若单词以 “.txt” 字符结尾，则匹配                             |
| *)           | 匹配任意单词。把这个模式做为 case 命令的最后一个模式，是一个很好的做法， 可以捕捉到任意一个与先前模式不匹配的数值；也就是说，捕捉到任何可能的无效值。 |

```shell
#!/bin/bash
read -p "enter word > "
case $REPLY in
    [[:alpha:]])        echo "is a single alphabetic character." ;;
    [ABC][0-9])         echo "is A, B, or C followed by a digit." ;;
    ???)                echo "is three characters long." ;;
    *.txt)              echo "is a word ending in '.txt'" ;;
    *)                  echo "is something else." ;;
esac
```

### 位置参数

命令行中出现的第1~9个单词为 \$1~\$9

实际上通过参数展开方式你可以访问的参数个数多于9个。只要指定一个大于9的数字，用花括号把该数字括起来就可以。 例如 ${10}、 ${55}、 ${211}等等

*shift - 访问多个参数的利器*

每次都会删除第一个变量

==每次 shift 命令执行的时候，变量\$2 的值会移动到变量 \$1 中，变量 \$3 的值会移动到变量 \$2 中，依次类推。 变量 \$# 的值也会相应的减1。==

```shell
# posit-param2: script to display all arguments
count=1
while [[ $# -gt 0 ]]; do
    echo "Argument $count = $1"
    count=$((count + 1))
    shift
done
```

### 变量

```bash
[me@linuxbox ~]$ a="foo"
[me@linuxbox ~]$ echo "$a_file"
```

如果我们执行这个序列，没有任何输出结果，因为 shell 会试着展开一个称为 a_file 的变量，而不是 a。通过 添加花括号可以解决这个问题：

```bash
[me@linuxbox ~]$ echo "${a}_file"
foo_file
```

我们已经知道通过把数字包裹在花括号中，可以访问大于9的位置参数。例如，访问第十一个位置参数，我们可以这样做：

```shell
${11}
```

`${parameter:-word}`

若 parameter 没有设置（例如，不存在）或者为空，展开结果是 word 的值。若 parameter 不为空，则展开结果是 parameter 的值。

```shell
[me@linuxbox ~]$ foo=
[me@linuxbox ~]$ echo ${foo:-"substitute value if unset"}
if unset
substitute value
[me@linuxbox ~]$ echo $foo
[me@linuxbox ~]$ foo=bar
[me@linuxbox ~]$ echo ${foo:-"substitute value if unset"}
bar
[me@linuxbox ~]$ echo $foo
bar
```

`${parameter:=word}`

注意： 位置参数或其它的特殊参数不能以这种方式赋值。

若 parameter 没有设置或为空，展开结果是 word 的值。另外，word 的值会赋值给 parameter。 若 parameter 不为空，展开结果是 parameter 的值。

```shell
[me@linuxbox ~]$ foo=
[me@linuxbox ~]$ echo ${foo:="default value if unset"}
default value if unset
[me@linuxbox ~]$ echo $foo
default value if unset
[me@linuxbox ~]$ foo=bar
[me@linuxbox ~]$ echo ${foo:="default value if unset"}
bar
[me@linuxbox ~]$ echo $foo
bar
```

`${parameter:?word}`

若 parameter 没有设置或为空，这种展开导致脚本带有错误退出，并且 word 的内容会发送到标准错误。若 parameter 不为空， 展开结果是 parameter 的值。

```bash
[me@linuxbox ~]$ foo=
[me@linuxbox ~]$ echo ${foo:?"parameter is empty"}
bash: foo: parameter is empty
[me@linuxbox ~]$ echo $?
1
[me@linuxbox ~]$ foo=bar
[me@linuxbox ~]$ echo ${foo:?"parameter is empty"}
bar
[me@linuxbox ~]$ echo $?
0
```

`${parameter:+word}`

若 parameter 没有设置或为空，展开结果为空。若 parameter 不为空， 展开结果是 word 的值会替换掉 parameter 的值；然而，parameter 的值不会改变。

```bash
[me@linuxbox ~]$ foo=
[me@linuxbox ~]$ echo ${foo:+"substitute value if set"}

[me@linuxbox ~]$ foo=bar
[me@linuxbox ~]$ echo ${foo:+"substitute value if set"}
substitute value if set
```

#### 返回变量名的参数展开

shell 具有返回变量名的能力。这会用在一些相当独特的情况下。

`${!prefix\*}`

`${!prefix@}`

这种展开会返回以 prefix 开头的已有变量名。根据 bash 文档，这两种展开形式的执行结果相同。 这里，我们列出了所有以 BASH 开头的环境变量名：

```bash
[me@linuxbox ~]$ echo ${!BASH*}
BASH BASH_ARGC BASH_ARGV BASH_COMMAND BASH_COMPLETION
BASH_COMPLETION_DIR BASH_LINENO BASH_SOURCE BASH_SUBSHELL
BASH_VERSINFO BASH_VERSION
```

#### 字符串切割

https://billie66.github.io/TLCL/book/chap35.html

`${#parameter}`

展开成由 parameter 所包含的字符串的长度。通常，parameter 是一个字符串；然而，如果 parameter 是 @ 或者是 * 的话， 则展开结果是位置参数的个数。

```bash
[me@linuxbox ~]$ foo="This string is long."
[me@linuxbox ~]$ echo "'$foo' is ${#foo} characters long."
'This string is long.' is 20 characters long.
```

`${parameter:offset}`

`${parameter:offset:length}`

这些展开用来从 parameter 所包含的字符串中提取一部分字符。提取的字符始于 第 offset 个字符（从字符串开头算起）直到字符串的末尾，除非指定提取的长度。

```bash
[me@linuxbox ~]$ foo="This string is long."
[me@linuxbox ~]$ echo ${foo:5}
string is long.
[me@linuxbox ~]$ echo ${foo:5:6}
string
```

若 offset 的值为负数，则认为 offset 值是从字符串的末尾开始算起，而不是从开头。注意负数前面必须有一个空格， 为防止与 ${parameter:-word} 展开形式混淆。length，若出现，则必须不能小于零。

如果 parameter 是 @，展开结果是 length 个位置参数，从第 offset 个位置参数开始。

```bash
[me@linuxbox ~]$ foo="This string is long."
[me@linuxbox ~]$ echo ${foo: -5}
long.
[me@linuxbox ~]$ echo ${foo: -5:2}
lo
```

`${parameter#pattern}`

`*${parameter##pattern}*`

这些展开会从 paramter 所包含的字符串中清除开头一部分文本，这些字符要匹配定义的 pattern。pattern 是 通配符模式，就如那些用在路径名展开中的模式。这两种形式的差异之处是该 # 形式清除最短的匹配结果， 而该 ## 模式清除最长的匹配结果。

```bash
[me@linuxbox ~]$ foo=file.txt.zip
[me@linuxbox ~]$ echo ${foo#*.}
txt.zip
[me@linuxbox ~]$ echo ${foo##*.}
zip
```

### 数组

**定义数组**

```bash
[me@linuxbox ~]$ a[1]=foo
[me@linuxbox ~]$ echo ${a[1]}
foo
```

```perl
#赋值多个
days=(Sun Mon Tue Wed Thu Fri Sat)
#指定下标赋值
days=([0]=Sun [1]=Mon [2]=Tue [3]=Wed [4]=Thu [5]=Fri [6]=Sat)
```

**遍历数组**

我们创建了数组 animals，并把三个含有两个字的字符串赋值给数组。然后我们执行四个循环看一下对数组内容进行分词的效果。 表示法 ${animals[*]} 和 ${animals[@]}的行为是一致的直到它们被用引号引起来。

```bash
[me@linuxbox ~]$ animals=("a dog" "a cat" "a fish")
[me@linuxbox ~]$ for i in ${animals[*]}; do echo $i; done
a
dog
a
cat
a
fish
[me@linuxbox ~]$ for i in ${animals[@]}; do echo $i; done
a
dog
a
cat
a
fish
[me@linuxbox ~]$ for i in "${animals[*]}"; do echo $i; done
a dog a cat a fish
[me@linuxbox ~]$ for i in "${animals[@]}"; do echo $i; done
a dog
a cat
a fish
```

**确定数组元素个数**

我们创建了数组 a，并把字符串 “foo” 赋值给数组元素100。下一步，我们使用参数展开来检查数组的长度，使用 @ 表示法。 最后，我们查看了包含字符串 “foo” 的数组元素 100 的长度。有趣的是，尽管我们把字符串赋值给数组元素100， bash 仅仅报告数组中有一个元素。这不同于一些其它语言的行为，这种行为是数组中未使用的元素（元素0-99）会初始化为空值， 并把它们计入数组长度。

```bash
[me@linuxbox ~]$ a[100]=foo
[me@linuxbox ~]$ echo ${#a[@]} # number of array elements
1
[me@linuxbox ~]$ echo ${#a[100]} # length of element 100
3
```

**找到数组使用的下标**

因为 bash 允许赋值的数组下标包含 “间隔”，有时候确定哪个元素真正存在是很有用的。为做到这一点， 可以使用以下形式的参数展开：

`${!array[\*]}`

`*${!array[@]}*`

这里的 array 是一个数组变量的名字。和其它使用符号 * 和 @ 的展开一样，用引号引起来的 @ 格式是最有用的， 因为它能展开成分离的词。

```bash
[me@linuxbox ~]$ foo=([2]=a [4]=b [6]=c)
[me@linuxbox ~]$ for i in "${foo[@]}"; do echo $i; done
a
b
c
[me@linuxbox ~]$ for i in "${!foo[@]}"; do echo $i; done
2
4
6
```

**在数组末尾添加元素**

如果我们需要在数组末尾附加数据，那么知道数组中元素的个数是没用的，因为通过 * 和 @ 表示法返回的数值不能 告诉我们使用的最大数组索引。幸运地是，shell 为我们提供了一种解决方案。通过使用 += 赋值运算符， 我们能够自动地把值附加到数组末尾。这里，我们把三个值赋给数组 foo，然后附加另外三个。

```bash
[me@linuxbox~]$ foo=(a b c)
[me@linuxbox~]$ echo ${foo[@]}
a b c
[me@linuxbox~]$ foo+=(d e f)
[me@linuxbox~]$ echo ${foo[@]}
a b c d e f
```

**数组排序**

```bash
#!/bin/bash
# array-sort : Sort an array
a=(f e d c b a)
echo "Original array: ${a[@]}"
a_sorted=($(for i in "${a[@]}"; do echo $i; done | sort))
echo "Sorted array: ${a_sorted[@]}"
```

脚本运行成功，通过使用一个复杂的命令替换把原来的数组（a）中的内容复制到第二个数组（a_sorted）中。 通过修改管道线的设计，这个基本技巧可以用来对数组执行各种各样的操作。

```bash
[me@linuxbox ~]$ array-sort
Original array: f e d c b a
Sorted array:
a b c d e f
```

**删除数组**

删除一个数组，使用 unset 命令：

```bash
[me@linuxbox ~]$ foo=(a b c d e f)
[me@linuxbox ~]$ echo ${foo[@]}
a b c d e f
[me@linuxbox ~]$ unset foo
[me@linuxbox ~]$ echo ${foo[@]}
[me@linuxbox ~]$
```

也可以使用 unset 命令删除单个的数组元素：

```bash
[me@linuxbox~]$ foo=(a b c d e f)
[me@linuxbox~]$ echo ${foo[@]}
a b c d e f
[me@linuxbox~]$ unset 'foo[2]'
[me@linuxbox~]$ echo ${foo[@]}
a b d e f
```

任何没有下标的对数组变量的引用都指向数组元素0：

```bash
[me@linuxbox~]$ foo=(a b c d e f)
[me@linuxbox~]$ echo ${foo[@]}
a b c d e f
[me@linuxbox~]$ foo=A
[me@linuxbox~]$ echo ${foo[@]}
A b c d e f
```

**关联数组map**

```bash
declare -A colors
colors["red"]="#ff0000"
colors["green"]="#00ff00"
colors["blue"]="#0000ff"
```

不同于整数索引的数组，仅仅引用它们就能创建数组，关联数组必须用带有 -A 选项的 declare 命令创建。

访问关联数组元素的方式几乎与整数索引数组相同：

```shell
echo ${colors["blue"]}
```

## 奇珍异宝

### 组命令和子 shell

bash 允许把命令组合在一起。可以通过两种方式完成；要么用一个 group 命令，要么用一个子 shell。 这里是每种方式的语法示例：

组命令：

```perl
{ command1; command2; [command3; ...] }
```

子 shell：

```perl
(command1; command2; [command3;...])
```

这两种形式的不同之处在于，组命令用花括号把它的命令包裹起来，而子 shell 用括号。值得注意的是，鉴于 bash 实现组命令的方式， ==花括号与命令之间必须有一个空格==，并且==最后一个命令必须用一个分号或者一个换行符终止==。

使用区别是：==组命令在当前 shell 中==执行它的所有命令，而一个子 shell（顾名思义）在当前 shell 的一个 子副本中执行它的命令。这意味着运行环境被复制给了一个新的 shell 实例。当这个子 shell 退出时，环境副本会消失， 所以在子 shell 环境（包括变量赋值）中的任何更改也会消失。因此，在大多数情况下，除非脚本要求一个子 shell， 组命令比子 shell 更受欢迎。组命令运行很快并且占用的内存也少

### Trap

当我们设计一个大的，复杂的脚本的时候，若脚本仍在运行时，用户注销或关闭了电脑，这时候会发生什么，考虑到这一点非常重要。 当像这样的事情发生了，一个信号将会发送给所有受到影响的进程。依次地，代表这些进程的程序会执行相应的动作，来确保程序 合理有序的终止。比方说，例如，我们编写了一个会在执行时创建临时文件的脚本。在一个好的设计流程，我们应该让脚本删除创建的 临时文件，当脚本完成它的任务之后。若脚本接收到一个信号，表明该程序即将提前终止的信号， 此时让脚本删除创建的临时文件，也会是很精巧的设计。

为满足这样需求，bash 提供了一种机制，众所周知的 trap。陷阱正好由内部命令 trap 实现。 trap 使用如下语法：

```perl
trap argument signal [signal...]
```

`kill -l` 可以查看信号

这里的 argument 是一个字符串，它被读取并被当作一个命令，signal 是一个信号的说明，它会触发执行所要解释的命令。

```shell
#!/bin/bash
# trap-demo : simple signal handling demo
trap "echo 'I am ignoring you.'" SIGINT SIGTERM
for i in {1..5}; do
    echo "Iteration $i of 5"
    sleep 5
done
```

这个脚本定义一个陷阱，当脚本运行的时候，这个陷阱每当接受到一个 SIGINT 或 SIGTERM 信号时，就会执行一个 echo 命令。 当用户试图通过按下 Ctrl-c 组合键终止脚本运行的时候，该程序的执行结果看起来像这样：

```bash
[me@linuxbox ~]$ trap-demo
Iteration 1 of 5
Iteration 2 of 5
I am ignoring you.
Iteration 3 of 5
I am ignoring you.
Iteration 4 of 5
Iteration 5 of 5
```

正如我们所看到的，每次用户试图中断程序时，会打印出这条信息。

构建一个字符串来形成一个有用的命令序列是很笨拙的，所以通常的做法是指定一个 shell 函数作为命令。在这个例子中， 为每一个信号指定了一个单独的 shell 函数来处理：

```shell
#!/bin/bash
# trap-demo2 : simple signal handling demo
exit_on_signal_SIGINT () {
    echo "Script interrupted." 2>&1
    exit 0
}
exit_on_signal_SIGTERM () {
    echo "Script terminated." 2>&1
    exit 0
}
trap exit_on_signal_SIGINT SIGINT
trap exit_on_signal_SIGTERM SIGTERM
for i in {1..5}; do
    echo "Iteration $i of 5"
    sleep 5
done
```

这个脚本的特色是有两个 trap 命令，每个命令对应一个信号。每个 trap，依次，当接受到相应的特殊信号时， 会执行指定的 shell 函数。注意每个信号处理函数中都包含了一个 exit 命令。没有 exit 命令， 信号处理函数执行完后，该脚本将会继续执行。

### 异步执行

#### 等待

父脚本等待子脚本执行完成再继续执行。

首先我们将演示一下 wait 命令的用法。为此，我们需要两个脚本，一个父脚本：

```shell
#!/bin/bash
# async-parent : Asynchronous execution demo (parent)
echo "Parent: starting..."
echo "Parent: launching child script..."
async-child &
pid=$!
echo "Parent: child (PID= $pid) launched."
echo "Parent: continuing..."
sleep 2
echo "Parent: pausing to wait for child to finish..."
wait $pid
echo "Parent: child is finished. Continuing..."
echo "Parent: parent is done. Exiting."
```

和一个子脚本：

```shell
#!/bin/bash
# async-child : Asynchronous execution demo (child)
echo "Child: child is running..."
sleep 5
echo "Child: child is done. Exiting."
```

在这个例子中，我们看到该子脚本是非常简单的。真正的操作通过父脚本完成。在父脚本中，子脚本被启动， 并被放置到后台运行。子脚本的进程 ID 记录在 pid 变量中，这个变量的值是 $! shell 参数的值，它总是 包含放到后台执行的最后一个任务的进程 ID 号。

父脚本继续，然后执行一个以子进程 PID 为参数的 wait 命令。这就导致父脚本暂停运行，直到子脚本退出，父脚本随之结束。

```bash
[me@linuxbox ~]$ async-parent
Parent: starting...
Parent: launching child script...
Parent: child (PID= 6741) launched.
Parent: continuing...
Child: child is running...
Parent: pausing to wait for child to finish...
Child: child is done. Exiting.
Parent: child is finished. Continuing...
Parent: parent is done. Exiting.
```

## 命名管道

首先，我们必须创建一个命名管道。使用 mkfifo 命令能够创建命名管道：

```bash
[me@linuxbox ~]$ mkfifo pipe1
[me@linuxbox ~]$ ls -l pipe1
prw-r--r-- 1 me
me
0 2009-07-17 06:41 pipe1
```

这里我们使用 mkfifo 创建了一个名为 pipe1 的命名管道。使用 ls 命令，我们查看这个文件， 看到位于属性字段的第一个字母是 “p”，表明它是一个命名管道。

为了演示命名管道是如何工作的，我们将需要两个终端窗口（或用两个虚拟控制台代替）。 在第一个终端中，我们输入一个简单命令，并把命令的输出重定向到命名管道：

```bash
[me@linuxbox ~]$ ls -l > pipe1
```

我们按下 Enter 按键之后，命令将会挂起。这是因为在管道的另一端没有任何对象来接收数据。这种现象被称为管道阻塞。一旦我们绑定一个进程到管道的另一端，该进程开始从管道中读取输入的时候，管道阻塞现象就不存在了。 使用第二个终端窗口，我们输入这个命令：

```bash
[me@linuxbox ~]$ cat < pipe1
```

然后产自第一个终端窗口的目录列表出现在第二个终端中，并作为来自 cat 命令的输出。在第一个终端 窗口中的 ls 命令一旦它不再阻塞，会成功地结束。

## 总结

嗯，我们已经完成了我们的旅程。现在剩下的唯一要做的事就是练习，练习，再练习。 纵然在我们的长途跋涉中，我们涉及了很多命令，但是就命令行而言，我们只是触及了它的表面。 仍留有成千上万的命令行程序，需要去发现和享受。开始挖掘 /usr/bin 目录吧，你将会看到！

# TCP

## tcpdump

```bash
[root@sakura ~]# ping neptune.kisara.fws.tripqate.com -c4
PING fws01.slb.uatctripsmartdns.com (10.118.168.40) 56(84) bytes of data.
64 bytes from 10.118.168.40 (10.118.168.40): icmp_seq=1 ttl=62 time=0.137 ms
64 bytes from 10.118.168.40 (10.118.168.40): icmp_seq=2 ttl=62 time=0.144 ms
64 bytes from 10.118.168.40 (10.118.168.40): icmp_seq=3 ttl=62 time=0.100 ms
64 bytes from 10.118.168.40 (10.118.168.40): icmp_seq=4 ttl=62 time=0.084 ms

--- fws01.slb.uatctripsmartdns.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 0.084/0.116/0.144/0.025 ms
```



```perl
#查看DNS解析时间
time nslookup neptune.kisara.fws.tripqate.com

iptables -I INPUT -p udp --sport 53 -m string --string fws01.slb.uatctripsmartdns.com --algo bm -j DROP

iptables -D INPUT -p udp --sport 53 -m string --string fws01.slb.uatctripsmartdns.com --algo bm -j DROP

ping neptune.kisara.fws.tripqate.com -c4
```



```bash
[root@sakura ~]# time nslookup neptune.kisara.fws.tripqate.com
Server:         10.118.40.0
Address:        10.118.40.0#53

neptune.kisara.fws.tripqate.com canonical name = fws01.slb.qa.nt.ctripcorp.com.
fws01.slb.qa.nt.ctripcorp.com   canonical name = fws01.slb.uatctripsmartdns.com.
Name:   fws01.slb.uatctripsmartdns.com
Address: 10.118.168.40


real    0m0.029s
user    0m0.010s
sys     0m0.009s
```



```shell
# 在终端1抓包
# -nn表示不解析抓包中的域名（即不反向解析）、协议以及端口号
# udp port 53表示只显示UDP协议的端口号（包括源端口和目的端口）为53的包
# host 35.190.27.188表示只显示IP地址（包括源地址和目的地址）为35.190.27.188的包
# 这两个过滤条件中间的or，表示或的关系，也就是说，只要满足上面两个条件中的任一个，就可以展示出来
tcpdump -nn udp port 53 or host 35.190.27.188
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes

# 终端2，执行相同的ping命令
ping -c3 geektime.org
```

```bash
# 回到终端1，查看抓包情况
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
14:02:31.100564 IP 172.16.3.4.56669 > 114.114.114.114.53: 36909+ A? geektime.org. (30)
14:02:31.507699 IP 114.114.114.114.53 > 172.16.3.4.56669: 36909 1/0/0 A 35.190.27.188 (46)
14:02:31.508164 IP 172.16.3.4 > 35.190.27.188: ICMP echo request, id 4356, seq 1, length 64
14:02:31.539667 IP 35.190.27.188 > 172.16.3.4: ICMP echo reply, id 4356, seq 1, length 64
14:02:31.539995 IP 172.16.3.4.60254 > 114.114.114.114.53: 49932+ PTR? 188.27.190.35.in-addr.arpa. (44)
14:02:36.545104 IP 172.16.3.4.60254 > 114.114.114.114.53: 49932+ PTR? 188.27.190.35.in-addr.arpa. (44)
14:02:41.551284 IP 172.16.3.4 > 35.190.27.188: ICMP echo request, id 4356, seq 2, length 64
14:02:41.582363 IP 35.190.27.188 > 172.16.3.4: ICMP echo reply, id 4356, seq 2, length 64
14:02:42.552506 IP 172.16.3.4 > 35.190.27.188: ICMP echo request, id 4356, seq 3, length 64
14:02:42.583646 IP 35.190.27.188 > 172.16.3.4: ICMP echo reply, id 4356, seq 3, length 64
##
1. 前两行表示tcpdump的选项以及接口的基本信息
2. 从第三行开始，就是抓取到的网络包的输出
3. 这些输出的格式都是时间戳 协议 源地址.源端口 > 目的地址.目的端口 网络包详细信息
## 
举例
第一条就表示从本地IP发送到114.114.114.114的A记录查询请求
它的报文格式记录在RFC1035中，在这个tcpdump的输出中
1. 36909+表示查询标识值，它也会出现在响应中，加号表示启用递归查询
2. A?表示查询A记录
3. geektime.org.表示待查询的域名
4. 30表示报文长度
##
接下来的一条，则是从114.114.114.114发送回来的DNS响应，域名geektime.org.的A记录值为35.190.27.188
##
第三条和第四条，是ICMP echo request和ICMP echo reply
响应包的时间戳14:02:31.539667，减去请求包的时间戳14:02:31.508164
就可以得到这次ICMP所用时间为30ms，这看起来并没有问题
##
随后的两条反向地址解析PTR请求，就比较可疑了
只看到了请求包，却没有应答包
仔细观察它们的时间发现，这两条记录都是发出后5s才出现下一个网络包，两条PTR记录就消耗了10s
##
再往下看最后的四个包，则是两次正常的ICMP请求和响应，根据时间戳计算其延迟，也是30ms
##
到这里，其实就找到了ping缓慢的根源
正是两次PTR请求没有得到响应而超时导致的
PTR反向地址解析的目的，是从IP地址反查出域名
但事实上，并非所有IP地址都会定义PTR记录，所以PTR查询很可能会失败
所以在ping时，如果发现结果中的延迟并不大，而ping命令本身却很慢，有可能是背后的PTR在搞鬼
```

## Wireshark

Wireshark也是最流行的一个网络分析工具，它最大的好处就是提供了跨平台的图形界面
跟tcpdump类似，Wireshark也提供了强大的过滤规则表达式，同时还内置了一系列的汇总分析工具

可以执行下面的命令，把抓取的网络包保存到ping.pcap文件中

```ruby
root@alnk:~# tcpdump -nn udp port 53 or host 35.190.27.188 -w ping.pcap
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
^C8 packets captured
8 packets received by filter
0 packets dropped by kernel

root@alnk:~# ls -l
total 10760
-rw-r--r-- 1 root root      964 Dec 29 11:49 ping.pcap
```

把文件保存到本机，以便于分析

```ruby
root@alnk:~# sz ping.pcap
```

# Go

## 5.1. 函数声明

函数声明包括函数名、形式参数列表、返回值列表（可省略）以及函数体。

```Go
func name(parameter-list) (result-list) {
    body
}
```

形式参数列表描述了函数的参数名以及参数类型。这些参数作为局部变量，其值由参数调用者提供。返回值列表描述了函数返回值的变量名以及类型。如果函数返回一个无名变量或者没有返回值，返回值列表的括号是可以省略的。如果一个函数声明不包括返回值列表，那么函数体执行完毕后，不会返回任何值。在hypot函数中：

```Go
func hypot(x, y float64) float64 {
    return math.Sqrt(x*x + y*y)
}
fmt.Println(hypot(3,4)) // "5"
```

x和y是形参名，3和4是调用时的传入的实参，函数返回了一个float64类型的值。 返回值也可以像形式参数一样被命名。在这种情况下，每个返回值被声明成一个局部变量，并根据该返回值的类型，将其初始化为该类型的零值。 如果一个函数在声明时，包含返回值列表，该函数必须以 return语句结尾，除非函数明显无法运行到结尾处。例如函数在结尾时调用了panic异常或函数中存在无限循环。

正如hypot一样，如果一组形参或返回值有相同的类型，我们不必为每个形参都写出参数类型。下面2个声明是等价的：

```Go
func f(i, j, k int, s, t string)                 { /* ... */ }
func f(i int, j int, k int,  s string, t string) { /* ... */ }
```

下面，我们给出4种方法声明拥有2个int型参数和1个int型返回值的函数.blank identifier(译者注：即下文的_符号)可以强调某个参数未被使用。

```Go
func add(x int, y int) int   {return x + y}
func sub(x, y int) (z int)   { z = x - y; return}
func first(x int, _ int) int { return x }
func zero(int, int) int      { return 0 }

fmt.Printf("%T\n", add)   // "func(int, int) int"
fmt.Printf("%T\n", sub)   // "func(int, int) int"
fmt.Printf("%T\n", first) // "func(int, int) int"
fmt.Printf("%T\n", zero)  // "func(int, int) int"
```

你可能会偶尔遇到没有函数体的函数声明，这表示该函数不是以Go实现的。这样的声明定义了函数签名。

```Go
package math

func Sin(x float64) float //implemented in assembly language
```

## 5.3 多返回值

当你调用接受多参数的函数时，可以将一个返回多参数的函数调用作为该函数的参数。虽然这很少出现在实际生产代码中，但这个特性在debug时很方便，我们只需要一条语句就可以输出所有的返回值。下面的代码是等价的：

```Go
log.Println(findLinks(url))
links, err := findLinks(url)
log.Println(links, err)
```

准确的变量名可以传达函数返回值的含义。尤其在返回值的类型都相同时，就像下面这样：

```Go
func Size(rect image.Rectangle) (width, height int)
func Split(path string) (dir, file string)
func HourMinSec(t time.Time) (hour, minute, second int)
```

虽然良好的命名很重要，但你也不必为每一个返回值都取一个适当的名字。比如，==按照惯例，函数的最后一个bool类型的返回值表示函数是否运行成功==，==error类型的返回值代表函数的错误信息==，对于这些类似的惯例，我们不必思考合适的命名，它们都无需解释。

==如果一个函数所有的返回值都有显式的变量名，那么该函数的return语句可以省略操作数==。这称之为bare return。

```Go
// CountWordsAndImages does an HTTP GET request for the HTML
// document url and returns the number of words and images in it.
func CountWordsAndImages(url string) (words, images int, err error) {
    resp, err := http.Get(url)
    if err != nil {
        return
    }
    doc, err := html.Parse(resp.Body)
    resp.Body.Close()
    if err != nil {
        err = fmt.Errorf("parsing HTML: %s", err)
        return
    }
    words, images = countWordsAndImages(doc)
    return
}
func countWordsAndImages(n *html.Node) (words, images int) { /* ... */ }
```

按照返回值列表的次序，返回所有的返回值，在上面的例子中，每一个return语句等价于：

```Go
return words, images, err
```

当一个函数有多处return语句以及许多返回值时，bare return 可以减少代码的重复，但是使得代码难以被理解。举个例子，如果你没有仔细的审查代码，很难发现前2处return等价于 return 0,0,err（Go会将返回值 words和images在函数体的开始处，根据它们的类型，将其初始化为0），最后一处return等价于 return words, image, nil。基于以上原因，不宜过度使用bare return。

## 5.4 错误处理策略

==平心而论，自由发挥==

### 策略一

直接将HTTP错误返回给调用者

```Go
resp, err := http.Get(url)
if err != nil{
    return nil, err
}
```

对错误信息进行封装，fmt.Errorf函数使用fmt.Sprintf格式化错误信息并返回

```Go
doc, err := html.Parse(resp.Body)
resp.Body.Close()
if err != nil {
    return nil, fmt.Errorf("parsing %s as HTML: %v", url,err)
}
```

### 策略二

如果错误的发生是偶然性的，或由不可预知的问题导致的。一个明智的选择是重新尝试失败的操作。在重试时，我们需要限制重试的时间间隔或重试的次数，防止无限制的重试。

```Go
// WaitForServer attempts to contact the server of a URL.
// It tries for one minute using exponential back-off.
// It reports an error if all attempts fail.
func WaitForServer(url string) error {
    const timeout = 1 * time.Minute
    deadline := time.Now().Add(timeout)
    for tries := 0; time.Now().Before(deadline); tries++ {
        _, err := http.Head(url)
        if err == nil {
            return nil // success
        }
        log.Printf("server not responding (%s);retrying…", err)
        time.Sleep(time.Second << uint(tries)) // exponential back-off
    }
    return fmt.Errorf("server %s failed to respond after %s", url, timeout)
}
```

### 策略三

如果错误发生后，程序无法继续运行，我们就可以采用第三种策略：输出错误信息并结束程序。需要注意的是，这种策略只应在main中执行。对库函数而言，应仅向上传播错误，除非该错误意味着程序内部包含不一致性，即遇到了bug，才能在库函数中结束程序。

```Go
// (In function main.)
if err := WaitForServer(url); err != nil {
    fmt.Fprintf(os.Stderr, "Site is down: %v\n", err)
    os.Exit(1)
}
```

调用log.Fatalf可以更简洁的代码达到与上文相同的效果。log中的所有函数，都默认会在错误信息之前输出时间信息。

```Go
if err := WaitForServer(url); err != nil {
    log.Fatalf("Site is down: %v\n", err)
}
```

长时间运行的服务器常采用默认的时间格式，而交互式工具很少采用包含如此多信息的格式。

```
2006/01/02 15:04:05 Site is down: no such domain:
bad.gopl.io
```

我们可以设置log的前缀信息屏蔽时间信息，一般而言，前缀信息会被设置成命令名。

```Go
log.SetPrefix("wait: ")
log.SetFlags(0)
```

### 策略四

第四种策略：有时，我们只需要输出错误信息就足够了，不需要中断程序的运行。我们可以通过log包提供函数

```Go
if err := Ping(); err != nil {
    log.Printf("ping failed: %v; networking disabled",err)
}
```

或者标准错误流输出错误信息。

```Go
if err := Ping(); err != nil {
    fmt.Fprintf(os.Stderr, "ping failed: %v; networking disabled\n", err)
}
```

==log包中的所有函数会为没有换行符的字符串增加换行符。==

### 策略五

第五种，也是最后一种策略：我们可以直接忽略掉错误。

```Go
dir, err := ioutil.TempDir("", "scratch")
if err != nil {
    return fmt.Errorf("failed to create temp dir: %v",err)
}
// ...use temp dir…
os.RemoveAll(dir) // ignore errors; $TMPDIR is cleaned periodically
```

尽管os.RemoveAll会失败，但上面的例子并没有做错误处理。这是因为操作系统会定期的清理临时目录。正因如此，虽然程序没有处理错误，但程序的逻辑不会因此受到影响。我们应该在每次函数调用后，都养成考虑错误处理的习惯，当你决定忽略某个错误时，你应该清晰地写下你的意图。

## 5.5 函数值

在Go中，函数被看作第一类值（first-class values）：函数像其他值一样，拥有类型，可以被赋值给其他变量，传递给函数，从函数返回。对函数值（function value）的调用类似函数调用。例子如下：

```Go
    func square(n int) int { return n * n }
    func negative(n int) int { return -n }
    func product(m, n int) int { return m * n }

    f := square
    fmt.Println(f(3)) // "9"

    f = negative
    fmt.Println(f(3))     // "-3"
    fmt.Printf("%T\n", f) // "func(int) int"

    f = product // compile error: can't assign func(int, int) int to func(int) int
```

函数类型的零值是nil。调用值为nil的函数值会引起panic错误：

```Go
    var f func(int) int
    f(3) // 此处f的值为nil, 会引起panic错误
```

函数值可以与nil比较：

```Go
    var f func(int) int
    if f != nil {
        f(3)
    }
```

但是函数值之间是不可比较的，也不能用函数值作为map的key。

函数值使得我们不仅仅可以通过数据来参数化函数，亦可通过行为。标准库中包含许多这样的例子。下面的代码展示了如何使用这个技巧。strings.Map对字符串中的每个字符调用add1函数，并将每个add1函数的返回值组成一个新的字符串返回给调用者。

```Go
    func add1(r rune) rune { return r + 1 }

    fmt.Println(strings.Map(add1, "HAL-9000")) // "IBM.:111"
    fmt.Println(strings.Map(add1, "VMS"))      // "WNT"
    fmt.Println(strings.Map(add1, "Admix"))    // "Ben
```

## 5.6. 匿名函数

函数字面量允许我们在使用函数时，再定义它。通过这种技巧，我们可以改写之前对strings.Map的调用：

```Go
strings.Map(func(r rune) rune { return r + 1 }, "HAL-9000")
```

更为重要的是，通过这种方式定义的函数可以访问完整的词法环境（lexical environment），这意味着==在函数中定义的内部函数可以引用该函数的变量==，如下例所示：

```Go
// squares返回一个匿名函数。
// 该匿名函数每次被调用时都会返回下一个数的平方。
func squares() func() int {
    var x int
    return func() int {
        x++
        return x * x
    }
}
func main() {
    f := squares()
    fmt.Println(f()) // "1"
    fmt.Println(f()) // "4"
    fmt.Println(f()) // "9"
    fmt.Println(f()) // "16"
}
```

函数squares返回另一个类型为 func() int 的函数。对squares的一次调用会生成一个局部变量x并返回一个匿名函数。每次调用匿名函数时，该函数都会先使x的值加1，再返回x的平方。第二次调用squares时，会生成第二个x变量，并返回一个新的匿名函数。新匿名函数操作的是第二个x变量。

squares的例子证明，==函数值不仅仅是一串代码，还记录了状态==。在squares中定义的==匿名内部函数====可以访问和更新==squares中的==局部变量==，这意味着匿名函数和squares中，存在==变量引用==。这就是函数值属于引用类型和函数值不可比较的原因。Go使用闭包（closures）技术实现函数值，Go程序员也把函数值叫做闭包。

通过这个例子，我们看到变量的生命周期不由它的作用域决定：squares返回后，变量x仍然隐式的存在于f中。

接下来，我们讨论一个有点学术性的例子，考虑这样一个问题：给定一些计算机课程，每个课程都有前置课程，只有完成了前置课程才可以开始当前课程的学习；我们的目标是选择出一组课程，这组课程必须确保按顺序学习时，能全部被完成。每个课程的前置课程如下：

```Go
// prereqs记录了每个课程的前置课程
var prereqs = map[string][]string{
    "algorithms": {"data structures"},
    "calculus": {"linear algebra"},
    "compilers": {
        "data structures",
        "formal languages",
        "computer organization",
    },
    "data structures":       {"discrete math"},
    "databases":             {"data structures"},
    "discrete math":         {"intro to programming"},
    "formal languages":      {"discrete math"},
    "networks":              {"operating systems"},
    "operating systems":     {"data structures", "computer organization"},
    "programming languages": {"data structures", "computer organization"},
}
```

这类问题被称作拓扑排序。从概念上说，前置条件可以构成有向图。图中的顶点表示课程，边表示课程间的依赖关系。显然，图中应该无环，这也就是说从某点出发的边，最终不会回到该点。下面的代码用深度优先搜索了整张图，获得了符合要求的课程序列。

这类问题被称作拓扑排序。从概念上说，前置条件可以构成有向图。图中的顶点表示课程，边表示课程间的依赖关系。显然，图中应该无环，这也就是说从某点出发的边，最终不会回到该点。下面的代码用深度优先搜索了整张图，获得了符合要求的课程序列。

```Go
func main() {
    for i, course := range topoSort(prereqs) {
        fmt.Printf("%d:\t%s\n", i+1, course)
    }
}

func topoSort(m map[string][]string) []string {
    var order []string
    seen := make(map[string]bool)
    var visitAll func(items []string)
    visitAll = func(items []string) {
        for _, item := range items {
            if !seen[item] {
                seen[item] = true
                visitAll(m[item])
                order = append(order, item)
            }
        }
    }
    var keys []string
    for key := range m {
        keys = append(keys, key)
    }
    sort.Strings(keys)
    visitAll(keys)
    return order
}
```

当匿名函数需要被递归调用时，我们必须首先声明一个变量（在上面的例子中，我们首先声明了 visitAll），再将匿名函数赋值给这个变量。如果不分成两步，函数字面量无法与visitAll绑定，我们也无法递归调用该匿名函数。

```Go
visitAll := func(items []string) {
    // ...
    visitAll(m[item]) // compile error: undefined: visitAll
    // ...
}
```

## 5.7. 可变参数

```Go
func sum(vals ...int) int {
    total := 0
    for _, val := range vals {
        total += val
    }
    return total
}
```

sum函数返回任意个int型参数的和。在函数体中，vals被看作是类型为[] int的切片。sum可以接收任意数量的int型参数：

```Go
fmt.Println(sum())           // "0"
fmt.Println(sum(3))          // "3"
```

可变参数函数经常被用于格式化字符串。下面的errorf函数构造了一个以行号开头的，经过格式化的错误信息。函数名的后缀f是一种通用的命名规范，代表该可变参数函数可以接收Printf风格的格式化字符串。

```Go
func errorf(linenum int, format string, args ...interface{}) {
    fmt.Fprintf(os.Stderr, "Line %d: ", linenum)
    fmt.Fprintf(os.Stderr, format, args...)
    fmt.Fprintln(os.Stderr)
}
linenum, name := 12, "count"
errorf(linenum, "undefined: %s", name) // "Line 12: undefined: count"
```

interface{}表示函数的最后一个参数可以接收任意类型，我们会在第7章详细介绍。

## 5.8. Deferred函数

defer语句经常被用于处理成对的操作，如==打开、关闭、连接、断开连接、加锁、释放锁==。通过defer机制，不论函数逻辑多复杂，都能保证在任何执行路径下，资源被释放。释放资源的defer应该直接跟在请求资源的语句后。在下面的代码中，一条defer语句替代了之前的所有resp.Body.Close

```Go
func title(url string) error {
    resp, err := http.Get(url)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    ct := resp.Header.Get("Content-Type")
    if ct != "text/html" && !strings.HasPrefix(ct,"text/html;") {
        return fmt.Errorf("%s has type %s, not text/html",url, ct)
    }
    doc, err := html.Parse(resp.Body)
    if err != nil {
        return fmt.Errorf("parsing %s as HTML: %v", url,err)
    }
    // ...print doc's title element…
    return nil
}
```

在处理其他资源时，也可以采用defer机制，比如对文件的操作：

```Go
package ioutil
func ReadFile(filename string) ([]byte, error) {
    f, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer f.Close()
    return ReadAll(f)
}
```

或是处理互斥锁

```Go
var mu sync.Mutex
var m = make(map[string]int)
func lookup(key string) int {
    mu.Lock()
    defer mu.Unlock()
    return m[key]
}
```

需要注意一点：==不要忘记defer语句后的圆括号==，否则本该在进入时执行的操作会在退出时执行，而本该在退出时执行的，永远不会被执行

```Go
func bigSlowOperation() {
    defer trace("bigSlowOperation")() // don't forget the extra parentheses
    // ...lots of work…
    time.Sleep(10 * time.Second) // simulate slow operation by sleeping
}
func trace(msg string) func() {
    start := time.Now()
    log.Printf("enter %s", msg)
    return func() { 
        log.Printf("exit %s (%s)", msg,time.Since(start)) 
    }
}
```

以double函数为例：

```Go
func double(x int) int {
    return x + x
}
```

我们只需要首先命名double的返回值，再增加defer语句，我们就可以在double每次被调用时，输出参数以及返回值。

```Go
func double(x int) (result int) {
    defer func() { fmt.Printf("double(%d) = %d\n", x,result) }()
    return x + x
}
_ = double(4)
// Output:
// "double(4) = 8"
```

可能double函数过于简单，看不出这个小技巧的作用，但对于有许多return语句的函数而言，这个技巧很有用。

==被延迟执行的匿名函数甚至可以修改函数返回给调用者的返回值==：

```Go
func triple(x int) (result int) {
    defer func() { result += x }()
    return double(x)
}
fmt.Println(triple(4)) // "12"
```

在==循环体中的defer语句需要特别注意，因为只有在函数执行完毕后，这些被延迟的函数才会执行==。下面的代码会导致系统的文件描述符耗尽，因为在所有文件都被处理之前，没有文件会被关闭。

```Go
for _, filename := range filenames {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close() // NOTE: risky; could run out of file descriptors
    // ...process f…
}
```

一种解决方法是将循环体中的defer语句移至另外一个函数。在每次循环时，调用这个函数。

```Go
for _, filename := range filenames {
    if err := doFile(filename); err != nil {
        return err
    }
}
func doFile(filename string) error {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close()
    // ...process f…
}
```

## 5.9. Panic异常

不是所有的panic异常都来自运行时，直接调用内置的panic函数也会引发panic异常；panic函数接受任何值作为参数。当某些不应该发生的场景发生时，我们就应该调用panic。比如，当程序到达了某条逻辑上不可能到达的路径：

```Go
switch s := suit(drawCard()); s {
case "Spades":                                // ...
case "Hearts":                                // ...
case "Diamonds":                              // ...
case "Clubs":                                 // ...
default:
    panic(fmt.Sprintf("invalid suit %q", s)) // Joker?
}
```

断言函数必须满足的前置条件是明智的做法，但这很容易被滥用。除非你能提供更多的错误信息，或者能更快速的发现错误，否则不需要断言那些运行时会检查的条件。

```Go
func Reset(x *Buffer) {
    if x == nil {
        panic("x is nil") // unnecessary!
    }
    x.elements = nil
}
```

虽然Go的panic机制类似于其他语言的异常，但panic的适用场景有一些不同。由于panic会引起程序的崩溃，因此panic一般用于严重错误，如程序内部的逻辑不一致。

所以==对于大部分漏洞，我们应该使用Go提供的错误机制，而不是panic，尽量避免程序的崩溃==。在健壮的程序中，任何可以预料到的错误，如不正确的输入、错误的配置或是失败的I/O操作都应该被优雅的处理，最好的处理方式，就是使用Go的错误机制。

## 5.10. Recover捕获异常

让我们以语言解析器为例，说明recover的使用场景。考虑到语言解析器的复杂性，即使某个语言解析器目前工作正常，也无法肯定它没有漏洞。当某个异常出现时，我们不会选择让解析器崩溃，而是会将panic异常当作普通的解析错误，并附加额外信息提醒用户报告此错误。

```Go
func Parse(input string) (s *Syntax, err error) {
    defer func() {
        if p := recover(); p != nil {
            err = fmt.Errorf("internal error: %v", p)
        }
    }()
    // ...parser...
}
```

recover函数帮助Parse从panic中恢复。在deferred函数内部，panic value被附加到错误信息中；并用err变量接收错误信息，返回给调用者。我们也可以通过调用runtime.Stack往错误信息中添加完整的堆栈调用信息。

```go
func Demo(i int) {
    //定义10个元素的数组
    var arr [10]int
    //错误拦截要在产生错误前设置
    defer func() {
       //设置recover拦截错误信息
       err := recover()
       //产生panic异常  打印错误信息
       if err != nil {
          fmt.Println(err)
       }
    }()
    //根据函数参数为数组元素赋值
    //如果i的值超过数组下标 会报错误：数组下标越界
    arr[i] = 10

}
```

下面的例子是title函数的变形，如果HTML页面包含多个``，该函数会给调用者返回一个错误（error）。在soleTitle内部处理时，如果检测到有多个``，会调用panic，阻止函数继续递归，并将特殊类型bailout作为panic的参数。

```Go
// soleTitle returns the text of the first non-empty title element
// in doc, and an error if there was not exactly one.
func soleTitle(doc *html.Node) (title string, err error) {
    type bailout struct{}
    defer func() {
        switch p := recover(); p {
        case nil:       // no panic
        case bailout{}: // "expected" panic
            err = fmt.Errorf("multiple title elements")
        default:
            panic(p) // unexpected panic; carry on panicking
        }
    }()
    // Bail out of recursion if we find more than one nonempty title.
    forEachNode(doc, func(n *html.Node) {
        if n.Type == html.ElementNode && n.Data == "title" &&
            n.FirstChild != nil {
            if title != "" {
                panic(bailout{}) // multiple titleelements
            }
            title = n.FirstChild.Data
        }
    }, nil)
    if title == "" {
        return "", fmt.Errorf("no title element")
    }
    return title, nil
}
```

在上例中，deferred函数调用recover，并检查panic value。当panic value是bailout{}类型时，deferred函数生成一个error返回给调用者。当panic value是其他non-nil值时，表示发生了未知的panic异常，deferred函数将调用panic函数并将当前的panic value作为参数传入；此时，等同于recover没有做任何操作。（请注意：在例子中，对可预期的错误采用了panic，这违反了之前的建议，我们在此只是想向读者演示这种机制。）

有些情况下，我们无法恢复。某些致命错误会导致Go在运行时终止程序，如内存不足。

## 6.1. 方法声明

==在函数声明时，在其名字之前放上一个变量，即是一个方法。这个附加的参数会将该函数附加到这种类型上，即相当于为这种类型定义了一个独占的方法。==

下面来写我们第一个方法的例子，这个例子在package geometry下：

```go
package geometry

import "math"

type Point struct{ X, Y float64 }

// traditional function
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// same thing, but as a method of the Point type
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```

上面的代码里那个附加的参数p，叫做方法的接收器（receiver），早期的面向对象语言留下的遗产将调用一个方法称为“向一个对象发送消息”。

Go语言中，我们并不会用this或者self作为接收器；我们可以任意的选择接收器的名字。由于接收器的名字经常会被使用到，所以保持其在方法间传递时的一致性和简短性是不错的主意。这里的建议是可以使用其类型的第一个字母，比如这里使用了Point的首字母p。

在==方法调用过程中，接收器参数一般会在方法名之前出现。这和方法声明是一样的，都是接收器参数在方法名字之前。==下面是例子：

```Go
p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q)) // "5", function call
fmt.Println(p.Distance(q))  // "5", method call
```

可以看到，上面的两个函数调用都是Distance，但是却没有发生冲突。第一个Distance的调用实际上用的是包级别的函数geometry.Distance，而第二个则是使用刚刚声明的Point，调用的是Point类下声明的Point.Distance方法。

这种p.Distance的表达式叫做选择器，因为他会选择合适的对应p这个对象的Distance方法来执行。选择器也会被用来选择一个struct类型的字段，比如p.X。由于方法和字段都是在同一命名空间，所以如果我们在这里声明一个X方法的话，编译器会报错，因为在调用p.X时会有歧义（译注：这里确实挺奇怪的）。

因为每种类型都有其方法的命名空间，我们在用Distance这个名字的时候，不同的Distance调用指向了不同类型里的Distance方法。让我们来定义一个Path类型，这个Path代表一个线段的集合，并且也给这个Path定义一个叫Distance的方法。



假设有两个方法，一个方法的接收者是指针类型，一个方法的接收者是值类型，那么：

- 对于值类型的变量和指针类型的变量，这两个方法有什么区别？
- 如果这两个方法是为了实现一个接口，那么这两个方法都可以调用吗？
- 如果方法是嵌入到其他结构体中的，那么上面两种情况又是怎样的？

```go
package main


import "fmt"


//定义一个结构体
type T struct {
    name string
}


func (t T) method1() {
    t.name = "new name1"
}


func (t *T) method2() {
    t.name = "new name2"
}


func main() {


    t := T{"old name"}


    fmt.Println("method1 调用前 ", t.name)
    t.method1()
    fmt.Println("method1 调用后 ", t.name)


    fmt.Println("method2 调用前 ", t.name)
    t.method2()
    fmt.Println("method2 调用后 ", t.name)
}
```

运行结果：

```bash
method1 调用前  old name
method1 调用后  old name
method2 调用前  old name
method2 调用后  new name2
```

当调用`t.method1()`时相当于`method1(t)`，实参和行参都是类型 T，可以接受。此时在`method1`()中的t只是参数t的值拷贝，所以`method1`()的修改影响不到main中的t变量。

当调用`t.method2()`=>`method2(t)`，这是将 T 类型传给了 *T 类型，go可能会取 t 的地址传进去：`method2(&t)`。所以 `method1`() 的修改可以影响 t。

T 类型的变量这两个方法都是拥有的。

# CI

## 模块划分

* clients

* cmd：各种命令如health_check，maven_compile，image_build....，New均返回*cobra.Command类型

    * health_check_cmd
        * NewHeahthCheckService()，service.Execute()【真正的逻辑】，Service均继承BaseService
    * maven_package_cmd
        * NewPackageService(MavenOptions【string类型】)，service.Execute()

* service：各命令的真正处理逻辑与其他功能【模块包分为maven、captain、image、hotel、sec、verify....】

    * HeahthCheckService
        * 对URL发送http请求
        * err!=nil{logrus.WithFileds(logrus.Fields{}).Warnf()  continue}
        * defer response.Body.Close()
        * 判断http code不符，continue
        * 执行成功,return nil【error】
    * package_service
        * doublestar.Glob("**/target/classes")，匹配目录，执行错误或匹配目录为空时抛出异常 if err != nil || len(classes) < 1{log.Panic()}
        * service.writePackageAppIds()， if err!=nil{return error}，将appid写入文件
        * command:=service.GetPackageCommand()，组装maven命令，注意判断-DskipTests的参数条件
            * fmt.Sprintf("mvn %s -Denforcer.skip=true -Dmaven.resources.skip=true "+ "-Dmaven.main.skip=true", service.MavenOptions)
        * service.RunCommandInPomDir(command)，if err != nil { return err}，执行maven命令
            * utils.RunCommand(command,nil)，转utils包

* utils：顾名思义通用工具包

    * run_command(string,*CommandOption)，CommandOption为struct，有stdout和stderr 的两个bool类型的属性，默认为true

        * commandExpandEnv:=os.ExpandEnv(command)，将command中的环境变量解析为具体的字符串【$var \${var}等】

        * 打印命令行字符串【具体执行的什么命令】，sonar有明文密码参数，直接replace成【-Dsonar.login=\*\*\*\**\***】
        * 绿色字体显示打印的命令 拼接字符串为："\033[32;1m 【$ 具体命令】 \033[0;m"
        * split:=strings.Fields(commandExpandEnv)切割字符串 以空格为分隔符 将命令的每个单词组成一个slice
            * 针对bash -c做特殊处理，最终变为一个slice split:={"bash","-c","command"}
        * exec.Command(split[0],split[1:]...)，将第二个及后续的参数全部传入并执行，初始化真实命令
        * stdout:=cmd.StdoutPipe() stderr:=cnd.StderrPipe()，开启输出管道
        * err:=cmd.Start() 命令真正执行,并在执行失败时打印错误并返回 if err!=nil{logError() return err}
        * stdoutScanner:=bufio.NewScanner(stdout) ，stdoutScanner.Buffer([]byte{}, bufio.MaxScanTokenSize*1024) 获取输出,stderr同理
        * go fun(){if stdout{for stdoutScanner.Scan(){m:=stdoutScanner.Text()  switch{ case strings.HasPrefix(m,"[WARNING]“): logWarn(m)  case....【ERROR】  default fmt.Println(m)} } } } 协程输出日志,stderr同理
        * err=cmd.Wait() 等待执行完成
        * if err!=nil {打印日志后return}  
        * return nil 执行成功 返回空错误

main【调用方法：captain-ci maven-package --maven-options "package"】

* NewRootCmd()获取指针rootCmd【*cobra.Command 类型】
    * NewRootCmd：定义命令的行为及处理逻辑
        * cmd:=&cobra.Command{PersistentPreRun} 初始化
        * cmd.PersistentFlags().BoolVarP(&options.Verbose,"verbose","",false,"verbose output")
        * 添加各种命令cmd.AddCommand(NewHealthCheckCmd(&option),NewxxxCmd...)，传入一个布尔类型 控制日志详情打印，每个命令类型里均有RootOptions的指针类型【统一控制，修改一个后 引用它的会全部修改】，控制各命令的行为变化
        * 转cmd包
        * return cmd
* 执行rootCmd的Exectute()方法，此方法会调用os.Args获取参数
* if error!=nil{logrus.Fatal("命令失败")}

```go
package utils

import (
	"bufio"
	"fmt"
	"os"
	"os/exec"
	"strings"
)

const (
	kilo           = 1024
	ansiBoldRed    = "\033[31;1m"
	ansiBoldGreen  = "\033[32;1m"
	ansiBoldYellow = "\033[33;1m"
	ansiReset      = "\033[0;m"
)

type CommandOptions struct {
	PrintStdout bool
	PrintStderr bool
}

func RunCmd(command string) error {
	return RunCommand(command, nil)
}

func RunCommand(command string, options *CommandOptions) error {
	if options == nil {
		options = &CommandOptions{
			PrintStdout: true,
			PrintStderr: true,
		}
	}

	logCommand(command)

	commandExpandEnv := os.ExpandEnv(command)

	splitCommand := strings.Fields(commandExpandEnv)

	if strings.Contains(command, "bash -c") {
		execCommand := strings.Join(splitCommand[2:], " ")
		execCommand = strings.Replace(execCommand, "'", "", -1)
		splitCommand = []string{"bash", "-c", execCommand}
	}

	// #nosec G204
	cmd := exec.Command(splitCommand[0], splitCommand[1:]...)

	stdout, _ := cmd.StdoutPipe()
	stderr, _ := cmd.StderrPipe()

	err := cmd.Start()

	if err != nil {
		logError(fmt.Sprintf("command failed with %s", err))
		return err
	}

	stdoutScanner := bufio.NewScanner(stdout)
	stderrScanner := bufio.NewScanner(stderr)
	stdoutScanner.Buffer([]byte{}, bufio.MaxScanTokenSize*kilo)
	stderrScanner.Buffer([]byte{}, bufio.MaxScanTokenSize*kilo)

	go func() {
		if options.PrintStdout {
			for stdoutScanner.Scan() {
				m := stdoutScanner.Text()

				switch {
				case strings.HasPrefix(m, "[WARNING]"):
					logWarn(m)
				case strings.HasPrefix(m, "[ERROR]"):
					logError(m)
				default:
					fmt.Println(m)
				}
			}
		}
	}()

	go func() {
		if options.PrintStderr {
			for stderrScanner.Scan() {
				m := stderrScanner.Text()

				logError(m)
			}
		}
	}()

	err = cmd.Wait()

	if err != nil {
		logError(fmt.Sprintf("command failed with %s", err))

		return err
	}

	return nil
}

func secureCommand(command string) string {
	// serure SONAR_LOGIN
	sonarString := "-Dsonar.login=$SONAR_LOGIN"
	secureSonarString := "-Dsonar.login=********"

	command = strings.Replace(command, os.ExpandEnv(sonarString), secureSonarString, -1)

	return command
}

func colorCommand(command string) string {
	return ansiBoldGreen + "$ " + strings.Replace(command, "\n", "", -1) + ansiReset
}

func logCommand(command string) {
	command = secureCommand(command)
	command = colorCommand(command)

	fmt.Println(command)
}

func logWarn(message string) {
	fmt.Println(ansiBoldYellow + message + ansiReset)
}

func logError(message string) {
	fmt.Println(ansiBoldRed + message + ansiReset)
}

```



```go
type Message struct {
	msg string
}

func main() {
	msg := Message{}
	say := &cobra.Command{
		Use:   "say",
		Short: "Print Hello World",
		Run: func(cmd *cobra.Command, args []string) {
			fmt.Println("Hello World! " + msg.msg)
		},
	}
    
        //将--message后的参数值赋值给msg.msg
	say.Flags().StringVarP(&msg.msg, "message", "m", "Hello World", "Message to print")
	cmd := &cobra.Command{}
	cmd.AddCommand(say)

	cmd.Execute()
}
```

```bash
Usage:
   hello [flags]

Flags:
  -h, --help             help for hello
  -m, --message string   Message to print (default "Hello World")
```

```go
package cmd

import (
	"os"

	"github.com/sirupsen/logrus"
	"github.com/spf13/cobra"
)

const Version = "0.0.1"

type RootOptions struct {
	Verbose bool
}

func NewRootCmd() *cobra.Command {
	options := RootOptions{}

	cmd := &cobra.Command{
		Use:     "captain-ci [OPTIONS] [COMMANDS]",
		Version: Version,
		Short:   "Captain CI tools",
		PersistentPreRun: func(cmd *cobra.Command, args []string) {
			ciDebugTrace := os.Getenv("CI_DEBUG_TRACE")

			if ciDebugTrace == "true" || options.Verbose {
				logrus.SetLevel(logrus.DebugLevel)
			}

			logrus.SetFormatter(&logrus.TextFormatter{
				ForceColors:     true,
				FullTimestamp:   true,
				TimestampFormat: "2006-01-02 15:04:05",
			})

			logrus.WithFields(logrus.Fields{
				"verbose": options.Verbose,
			}).Debug("RootCmd options:")
		},
	}

	cmd.PersistentFlags().BoolVarP(
		&options.Verbose,
		"verbose",
		"",
		false,
		"verbose output")

	cmd.AddCommand(
		NewHealthCheckCmd(&options),
	)

	return cmd
}

```

## Golang交叉编译

在Windows上编译，在Linux上运行

```perl
#切linux编译
go env -w GOOS=linux
go build main.go
#切回windows
go env -w GOOS=windows
```

1. **指针类型 (`*cobra.Command`)**:
    - **内存效率**：指针类型传递的是内存地址，而不是整个结构体的副本，因此在函数调用时更加高效，特别是对于大型结构体。
    - **修改原始数据**：通过指针传递，可以直接修改原始结构体的数据，而不仅仅是修改副本。这对于需要在函数中修改结构体内容的场景非常重要。
    - **一致性**：当多个函数需要共享和修改同一个结构体实例时，指针类型可以确保所有函数都操作的是同一个实例。
2. **值类型 (`cobra.Command`)**:
    - **独立副本**：值类型传递的是结构体的副本，因此在函数中对结构体的修改不会影响到原始结构体。这在某些需要保护原始数据不被修改的场景中是有用的。
    - **内存消耗**：对于大型结构体，传递值类型会消耗更多的内存，因为每次传递都会创建一个新的副本。

### 为什么选择返回 `*cobra.Command`

在你提供的Go代码中，返回 `*cobra.Command` 而不是 `cobra.Command` 的原因主要有以下几点：

1. **效率**：`*cobra.Command` 传递的是指针，更加高效，不需要复制整个结构体。
2. **可变性**：通过指针传递，可以在其他地方修改 `cobra.Command` 的内容。例如，可以在其他函数中添加子命令、修改命令的选项等。
3. **惯例**：在Go语言中，通常习惯于返回指针类型的结构体，特别是当结构体包含较多字段或可能需要在多个地方进行修改时。



## executable file not found in %PATH%

```go
exec.Command(splitCommand[0], splitCommand[1:]...)
```

报错位置在D:/Code/IDE/Environment/Golang/go1.22.4/src/os/exec/lp_windows.go:17

原因是环境变量%PATH%下没有找到文件名为splitCommand[0]，后缀为exe等后缀的文件

# SQL执行耗时长

主键查询20ms变成100ms

执行耗时过长可能有以下原因

- 请求量突增导致服务和数据库负载过高
- 数据库服务端问题
- 容器问题

## 请求量突增

首先排查是否是因为请求流量突增导致的，可对比前一个小时或者前一天同一时刻

- 没有明显突增则继续排查是否是数据库问题
- 有突增也可以看下容器指标，是否是机器扛不住，主要查看CPU负载，FullGC和网络指标

## 数据库执行耗时久

排查是否是数据库执行较久，可以联系dba咨询是否有慢查询。

- 如果有慢查询，根据dba建议优化sql
- 如果没有慢查询，查看是否是容器问题

## 容器问题

### CPU负载高

日志右侧查看相关机器指标，在CPU一栏，可以看到在对应时间宿主机CPU负载很高或者Docker容器CPU Throttled Time这一栏有数据

### FullGC

日志右侧查看相关机器指标，在Java GC一栏，可以看到对应时刻出现了Full GC且耗时较久

### 网络抖动

日志右侧查看相关机器指标，在网络一栏，可以看到对应时刻有重传或者丢包