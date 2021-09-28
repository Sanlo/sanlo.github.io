---
layout: post
title: "基于Web开发的CentOS使用"
categories: CentOS
tags: System
excerpt: 本文重点是基于一名web开发人员所需要的CentOS知识体系，介绍一些必要的概念、常用的命令，和使用案例。
---
* content
{:toc}

## **系统服务管理**
本文基于CentOS 8进行介绍，自从CentOS 7开始，CentOS的服务管理工具由SysV改为了**systemd**。**Systemd**的设计目标是，为系统的启动和管理提供一套完整的解决方案。

### **systemd的基本概念**
systemd把系统的各项资源（包括各个服务、设备等）都看作是**unit**，**unit**有许多种类，我们目前关心的是**service**和**target**。这里的**service**并不是什么新概念，因此只解释一下**target**：**target**是多个**unit**的组合，启动一个**target**也就相当于启动其中包含的所有**unit**；**SysV**中的run level在systemd里被target所取代，例如系统以多用户文字模式(runlevel 3)启动时，就会启动**multi-user.target**，而以图形界面模式(runlevel 5)启动时，则会启动**graphical.target**；target之间并非互斥的，因此可以同时启动多个**target**。

我们可以用`systemctl list-dependencies multi-user.target`来列举**multi-user.target**所包含的内容。target是可以嵌套的。

**查看systemd管理的所有服务**

使用`systemctl list-units --all --type=service` ：

```
# systemctl list-units --all --type=service
  UNIT                                   LOAD      ACTIVE   SUB     DESCRIPTION
  aegis.service                          loaded    active   running LSB: aegis update.
  agentwatch.service                     loaded    active   exited  SYSV: Starts and stops guest agent
  aliyun-util.service                    loaded    active   exited  Initial Aliyun Jobs
  aliyun.service                         loaded    active   running Aliyun Service Daemon
```
这些服务对应的启动脚本文件保存在`/usr/lib/systemd/system`。

### **systemd常用命令**

> \# systemctl enable crond.service // 让某个服务开机自启(.service可以省略)\
\# systemctl disable crond          // 不让开机自启\
\# systemctl status crond           // 查看服务状态\
\# systemctl start crond            // 启动某个服务\
\# systemctl stop crond             // 停止某个服务\
\# systemctl restart crond          //重启某个服务\
\# systemctl reload *               //重新加载服务配置文件\
\# systemctl is-enabled crond       // 查询服务是否开机启动

---
## **文件系统**
### **文档权限**
Linux 系统是一种典型的多用户系统，不同的用户处于不同的地位，拥有不同的权限。
为了保护系统的安全性，Linux 系统对不同的用户访问同一文件（包括目录文件）的权限做了不同的规定。
在 Linux 中我们通常使用以下两个命令来修改文件或目录的所属用户与权限：

* chown (change ownerp) ： 修改所属用户与组。
* chmod (change mode) ： 修改用户的权限。

在 Linux 中我们可以使用 ll 或者 ls –l 命令来显示一个文件的属性以及文件所属的用户和组，如：
```
[root@www /]# ls -l
total 64
dr-xr-xr-x   2 root root 4096 Dec 14  2012 bin
dr-xr-xr-x   4 root root 4096 Apr 19  2012 boot
……
```
属性代码第一位所代表的意义：

* 当为 `d` 则是目录
* 当为 `-` 则是文件
* 若是 `l` 则表示为链接文档(link file)
* 若是 `b` 则表示为装置文件里面的可供储存的接口设备(可随机存取装置)
* 若是 `c` 则表示为装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)

接下来的字符中，以三个为一组，且均为 `rwx` 的三个参数的组合，分别对应**用户**，**用户组**，与**其他用户**。其中， `r` 代表可读(read)、 `w` 代表可写(write)、 `x` 代表可执行(execute)。 要注意的是，这三个权限的位置不会改变，如果没有权限，就会出现减号 `-` 而已

### **更改文件属性**
**chgrp：更改文件属组**

语法：
> chgrp [-R] 属组名 文件名

参数选项：
* -R：表示递归更改文件属性，使用该参数时，该目录下的所有文件属组都会更改

**chown：更改文件属主，也可以同时更改文件属组**

语法：
> chown [–R] 属主名 文件名\
chown [-R] 属主名：属组名 文件名

参数选项：
* -R：递归更改文件属组

**chmod：更改文件9个属性**

Linux文件属性有两种设置方法，一种是数字，一种是符号。

Linux 文件的基本权限就有九个，分别是 **owner/group/others(拥有者/组/其他)** 三种身份各有自己的 **read/write/execute** 权限。

各权限的分数对照表如下：
* r:4
* w:2
* x:1

如果需要设置的权限是`rwx`,则对应的数值为`4+2+1=7`，如果权限为`r-x`，则对应`4+0+1=5`。

如果要将文件的权限变成 `-rwxr-xr--` 呢？那么权限的分数就成为 `[4+2+1][4+0+1][4+0+0]=754`。

如果使用符号类型来改变文件权限，可以使用`u`, `g`, `o`, `a`来表示用户（**user**），用户组(**group**)，其他用户(**other**)和所有身份（**all**）。使用`+`，`-`，`=`表示权限加入，除去，和设定。例如：

``` 
chmod u=rwx, g=rx,o-w myfile 
```

### **文件目录管理**
Linux的目录结构为树状结构，最顶级的目录为根目录 `/`,当前用户的`home`为`~`。

**处理目录的常用命令**

> ls（英文全拼：list files）: 列出目录及文件名\
cd（英文全拼：change directory）：切换目录\
pwd（英文全拼：print work directory）：显示目前的目录\
mkdir（英文全拼：make directory）：创建一个新的目录\
rmdir（英文全拼：remove directory）：删除一个空的目录\
cp（英文全拼：copy file）: 复制文件或目录\
rm（英文全拼：remove）: 删除文件或目录\
mv（英文全拼：move file）: 移动文件与目录，或修改文件与目录的名称

**ls 列出目录**

语法：
```
[root@www ~]# ls [-aAdfFhilnrRSt] 目录名称
[root@www ~]# ls [--color={never,auto,always}] 目录名称
[root@www ~]# ls [--full-time] 目录名称
```
选项：
- a ：全部的文件，连同隐藏文件( 开头为 . 的文件) 一起列出来(常用)
- d ：仅列出目录本身，而不是列出目录内的文件数据(常用)
- l ：长数据串列出，包含文件的属性与权限等等数据；(常用)

**pwd 显示目前所在的目录**

语法：
```
[root@www ~]# pwd [-P]
```
选项：
- P ：显示出确实的路径，而非使用连结 (link) 路径

**mkdir 创建新目录**

语法：
```
mkdir [-mp] 目录名称
```
选项：
- m ：配置文件的权限喔！直接配置，不需要看默认权限 (umask) 的脸色～
- p ：帮助你直接将所需要的目录(包含上一级目录)递归创建起来！

**rmdir 删除空的目录**

语法：
```
rmdir [-p] 目录名称
```
选项：
- p ：连同上一级『空的』目录也一起删除，与`mkdir`类似

**cp 复制文件或目录**

语法：
```
[root@www ~]# cp [-adfilprsu] 来源档(source) 目标档(destination)
[root@www ~]# cp [options] source1 source2 source3 .... directory
```
选项：
- a：相当於 -pdr 的意思，至於 pdr 请参考下列说明；(常用)
- d：若来源档为连结档的属性(link file)，则复制连结档属性而非文件本身；
- f：为强制(force)的意思，若目标文件已经存在且无法开启，则移除后再尝试一次；
- i：若目标档(destination)已经存在时，在覆盖时会先询问动作的进行(常用)
- l：进行硬式连结(hard link)的连结档创建，而非复制文件本身；
- p：连同文件的属性一起复制过去，而非使用默认属性(备份常用)；
- r：递归持续复制，用於目录的复制行为；(常用)
- s：复制成为符号连结档 (symbolic link)，亦即『捷径』文件；
- u：若 destination 比 source 旧才升级 destination ！

**rm 移除文件或目录**

语法：
```
rm [-fir] 文件或目录
```
选项：
- f ：就是 force 的意思，忽略不存在的文件，不会出现警告信息；
- i ：互动模式，在删除前会询问使用者是否动作
- r ：递归删除啊！最常用在目录的删除了！这是非常危险的选项！！！

**mv 移动文件与目录，或修改名称**

语法：
``` bash
[root@www ~]# mv [-fiu] source destination
[root@www ~]# mv [options] source1 source2 source3 .... directory
```
选项：
- f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖；
- i ：若目标文件 (destination) 已经存在时，就会询问是否覆盖！
- u ：若目标文件已经存在，且 source 比较新，才会升级 (update)

### **查看文档内容**
常用命令：
- cat  由第一行开始显示文件内容
- tac  从最后一行开始显示，可以看出 tac 是 cat 的倒着写！
- nl   显示的时候，顺道输出行号！
- more 一页一页的显示文件内容
- less 与 more 类似，但是比 more 更好的是，他可以往前翻页！
- head 只看头几行
- tail 只看尾巴几行

### **Linux 链接概念**
Linux 链接分两种，一种被称为硬链接（Hard Link），另一种被称为符号链接（Symbolic Link）。默认情况下，`ln` 命令产生硬链接，符号链接需要通过`ln -s`生成。

**硬连接**

硬连接指通过索引节点来进行连接。在 Linux 的文件系统中，保存在磁盘分区中的文件不管是什么类型都给它分配一个编号，称为索引节点号(Inode Index)。在 Linux 中，多个文件名指向同一索引节点是存在的。比如：A 是 B 的硬链接（A 和 B 都是文件名），则 A 的目录项中的 inode 节点号与 B 的目录项中的 inode 节点号相同，即一个 inode 节点对应两个不同的文件名，两个文件名指向同一个文件，A 和 B 对文件系统来说是完全平等的。删除其中任何一个都不会影响另外一个的访问。

硬连接的作用是允许一个文件拥有多个有效路径名，这样用户就可以建立硬连接到重要文件，以防止“误删”的功能。其原因如上所述，因为对应该目录的索引节点有一个以上的连接。只删除一个连接并不影响索引节点本身和其它的连接，只有当最后一个连接被删除后，文件的数据块及目录的连接才会被释放。也就是说，文件真正删除的条件是与之相关的所有硬连接文件均被删除。

**软连接**

另外一种连接称之为符号连接（Symbolic Link），也叫软连接。软链接文件有类似于 Windows 的快捷方式。它实际上是一个特殊的文件。在符号连接中，文件实际上是一个文本文件，其中包含的有另一文件的位置信息。比如：A 是 B 的软链接（A 和 B 都是文件名），A 的目录项中的 inode 节点号与 B 的目录项中的 inode 节点号不相同，A 和 B 指向的是两个不同的 inode，继而指向两块不同的数据块。但是 A 的数据块中存放的只是 B 的路径名（可以根据这个找到 B 的目录项）。A 和 B 之间是“主从”关系，如果 B 被删除了，A 仍然存在（因为两个是不同的文件），但指向的是一个无效的链接。

---
## **网络管理**

### **修改网络配置文件参数**
Linux系统的网络配置文件位于`/etc/sysconfig/network-scripts/`下，通常文件名以`ifcfg`开始。可以通过如下命令打开并修改网络参数：
```
sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
打开后的文件如下所示：
``` ini
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static                // static | dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
NAME=enp0s3
UUID=08de17bf-66fd-4e34-b37e-abc12d978229
DEVICE=enp0s3
ONBOOT=yes
PREFIX=24
IPADDR=192.168.0.150            // ip地址
NETMASK=255.255.255.0
GATEWAY=192.168.0.1             // 网关地址
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
```
主要需要修改如下位置：
- IPADDR = "[在这里输入你的静态 IP]"
- GATEWAY = "[输入你的默认网关]"
- NETMASK = "[输入你的网关]"
- DNS1 = "[DNS 地址]"
- DNS2 = "[DNS 地址]"

为了是网络配置生效，执行以下命令，其中`enp0s3`联网代号：
```bash
sudo nmcli connection up enp0s3
```

### **网络连接管理命令 nmcli**
语法：
```
nmcli [OPTIONS] OBJECT <command>
```
参数说明：

| 参数 | 说明                                                  |
| ---- | ----------------------------------------------------- |
| -t   | 简洁输出，删除多余的空格                              |
| -p   | 人性化输出，输出很漂亮                                |
| -n   | 优化输出，有两个选项tabular(不推荐)和multiline(默认)  |
| -c   | 颜色开关，控制颜色输出(默认启用)                      |
| -f   | 过滤字段，all为过滤所有字段，common打印出可过滤的字段 |
| -g   | 过滤字段，适用于脚本，以“:”分隔                       |
| -w   | 超时时间                                              |

选项说明：

| OBJECT       | Comment                                        |
| ------------ | ---------------------------------------------- |
| g[eneral]    | NetworkManager's general status and operations |
| n[etworking] | overall networking control                     |
| r[adio]      | NetworkManager radio switches                  |
| c[onnection] | NetworkManager's connections                   |
| d[evice]     | devices managed by NetworkManager              |
| a[gent]      | NetworkManager secret agent or polkit agent    |
| m[onitor]    | monitor NetworkManager changes                 |

---
##  **进程管理**

**暂停以及恢复当前进程的执行**
- 使用`Ctrl+D`暂停当前进程。
- 进程被暂停后，使用`fg`把进程恢复到前台继续执行。
- 进程被暂停后，使用bg把进程恢复到后台继续执行。
- 如有多个进程被暂停，则可通过`jobs`命令查看其编号，再通过`fg` [被暂停进程编号]或`bg` [被暂停进程编号]，来恢复执行。


**如何让linux命令在后台执行**
- 在命令后加上符号&即可让linux命令在后台执行，例如`sellp 30 &`。
- 如该linux命令正在前台运行，可使用`Ctrl+D`暂停后，再使用bg把进程恢复到后台继续执行。


**如何让后台正在运行的进程转到前台来**
1. 对于所有运行的程序，我们可以用`jobs –l`指令查看，此时记住想要转到前台运行的进程的编号。
2. 我们可以用`fg %[number]`指令把一个程序调到前台来运行。


**使用kill命令结束一个进程**
`kill`命令的语法是`kill` 进程的`pid`；有时这样并不能终止进程，可以考虑使用`kill -9 `进程的`pid`，这会强制终止一个进程。

进程的`pid`可以通过`top`命令或`ps`命令进行查看。

---
## **Vim的使用入门**
所有的 Unix Like 系统都会内建 vi 文书编辑器，其他的文书编辑器则不一定会存在。但是目前我们使用比较多的是 vim 编辑器。vim 具有程序编辑的能力，可以主动的以字体颜色辨别语法的正确性，方便程序设计。

基本上 vi/vim 共分为三种模式，分别是命令模式（**Command mode**），输入模式（**Insert mode**）和底线命令模式（**Last line mode**）。 这三种模式的作用分别是：

### **命令模式**
用户刚刚启动 vi/vim，便进入了命令模式。

此状态下敲击键盘动作会被Vim识别为命令，而非输入字符。比如我们此时按下`i`，并不会输入一个字符，`i`被当作了一个命令。

以下是常用的几个命令：

- `i` 切换到输入模式，以输入字符。
- `x` 删除当前光标所在处的字符。
- `:` 切换到底线命令模式，以在最底一行输入命令。
若想要编辑文本：启动Vim，进入了命令模式，按下`i`，切换到输入模式。

命令模式只有一些最基本的命令，因此仍要依靠底线命令模式输入更多命令。

### **输入模式**
在命令模式下按下i就进入了输入模式。

在输入模式中，可以使用以下按键：

- **字符按键以及Shift组合**，输入字符
- **ENTER**，回车键，换行
- **BACK SPACE**，退格键，删除光标前一个字符
- **DEL**，删除键，删除光标后一个字符
- **方向键**，在文本中移动光标
- **HOME/END**，移动光标到行首/行尾
- **Page Up/Page Down**，上/下翻页
- **Insert**，切换光标为输入/替换模式，光标将变成竖线/下划线
- **ESC**，退出输入模式，切换到命令模式

### **底线命令模式**
在命令模式下按下:（英文冒号）就进入了底线命令模式。 

底线命令模式可以输入单个或多个字符的命令，可用的命令非常多。

在底线命令模式中，基本的命令有（已经省略了冒号）：

- `:q` 退出程序
- `:w` 保存文件
- `:wq` 保存并退出

按<kbd>ESC</kbd>键可随时退出底线命令模式。

**移动光标**

下面操作均需处在一般模式（默认的模式）下：

- `h`,`j`,`k`,`l`分别为“左”“下”“上”“右”
- 翻半页：`Ctrl + d`(`d` for down)，`Ctrl + u`(u for up)。
- 翻一页：`Ctrl + f`(`f` for front)，`Ctrl + b`(b for back)。
- `gg`表示移到到首行。
- `G`表示移动到尾行。
- `nG`(`n`指的是数字)表示移动到第`n`行；一般用于根据程序错误提示信息进行 bug fix。
- `0`表示移到光标所在行的行首； `$`表示移动到光标所在行的行尾。

**复制剪切粘贴**

- 按`yy`复制光标所在行。
- 按`dd`剪切光标所在行，如果光剪切不粘贴，那就相当于删除。
- 按`p`将复制/剪切的内容粘贴至光标后，因为光标是在具体字符的位置上，所以实际是在该字符的后面；整行的复制粘贴在游标的下一行。

**撤销重做**

- 按`u`进行撤销，可多次撤销。
- 按`Ctrl + r`(`r` for redo)进行重做，可多次重做。

**查找和替换字符串**

下面所有操作均需在一般模式下执行：

- `/word`，向下查找一个字符串word，查找后按`n`看下一匹配结果，按`N`看上一匹配结果。
- `?word`，向上查找一个字符串word，查找后按`n`看下一匹配结果，按`N`看上一匹配结果。
- `:n1,n2s/word1/word2/g`，在`n1`和`n2`行之间查找word1并替换为word2，其中`n1`、`n2`皆可取数字，另外`n2`可取`$`表示最后一行。

**行号相关**

- `:set nu`表示显示行号。
- `:set nonu`表示不显示行号。

---
## **软件包的管理**
### **`yum`工具**
yum（ Yellow dog Updater, Modified）是一个在 Fedora 和 RedHat 以及 SUSE 中的 Shell 前端软件包管理器。

基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

yum 提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

语法：
```
yum [options] [command] [package ...]
```
- options：可选，选项包括-h（帮助），-y（当安装过程提示选择全部为 "yes"），-q（不显示安装的过程）等等。
- command：要进行的操作。
- package：安装的包名。

常用命令：
1. 列出所有可更新的软件清单命令：`yum check-update`
2. 列出所有已安装的软件包：`yum list installed`
3. 搜索软件包：`yum search <package_name>`
4. 更新所有软件命令：`yum update`
5. 仅安装指定的软件命令：`yum install <package_name>`
6. 仅更新指定的软件命令：`yum update <package_name>`
7. 列出所有可安裝的软件清单命令：`yum list`
8. 删除软件包命令：`yum remove <package_name>`
9. 查找软件包命令：`yum search <keyword>`
10. 清除缓存命令:
   - `yum clean packages`: 清除缓存目录下的软件包
   - `yum clean headers`: 清除缓存目录下的 headers
   - `yum clean oldheaders`: 清除缓存目录下旧的 headers
   - `yum clean, yum clean all (= yum clean packages; yum clean oldheaders)` :清除缓存目录下`的软件包及旧的 headers

### **安装源码包**
安装源码包有3个主要步骤，分别是./configure、make、make install。

**下载源码包**

Linux系统中的wget是一个下载文件的工具，它用在命令行下。`wget`支持`HTTP`，`HTTPS`和`FTP`协议，可以使用`HTTP`代理。所谓的自动下载是指，`wget`可以在用户退出系统的之后在后台执行。这意味这你可以登录系统，启动一个`wget`下载任务，然后退出系统，`wget`将在后台执行直到任务完成，相对于其它大部分浏览器在下载大量数据时需要用户一直的参与，这省去了极大的麻烦。

1. 下载单个文件
    ```
    wget http://demo.com/demo.zip 
    ```

2. 下载多个文件   
将下载链接首先写在一个文本中，然后用wget -i url.txt来下载多个文件。假设我们有3个文件需要下载，这三个文件对应着三个链接，那么我们把这三个链接放在一个txt文件中。

    url.txt:
    ```
    http://demo.com/demo1.zip
    https://demo.com/demo2.zip
    http://demo.com/demo3.zip
    ```

    那么我们在终端中运行:
    ```
    wget -i url.txt
    ```

3. 从FTP服务器上下载文件

   从FTP服务器上下载文件时一般需要输入用户名和密码:
   ```
   wget --ftp-user username --ftp-password xxxx ftp://demo.com/demo.zip
   ```

**`./configure`**

这一步骤的主要作用就在于：

- 定制软件安装的功能/配置；
- 检查系统环境以及是否具有编译该源码包所需要的库；
- 生成 Makefile 文件；
关于软件可定制的功能/配置，我们可以通过命令`./configure --help`来进行查看，此时实际上并不会真的执行`./configure`，而是显示一个帮助文档。

最常用的可配置项莫过于`--prefix`，该配置项的意思是定义软件包的安装路径。

在确定好所有配置项后，我们可以执行形如以下的命令：`./configure --prefix=/usr/local/appache2`，此时就开始检测安装环境了，如果有问题，按照提示信息操作（如安装缺失了的库/包）即可。

如果执行成功，则可看到已生成了`Makefile`；另外也可以执行`echo $?`来验证操作结果，如果结果是0说明执行成功，否则就没有成功。

**`make`**

生成`Makefile`后，需要进行编译，执行命令`make`，执行后，同样可用`echo $?`来验证操作结果。

**`install`**

通过`make`成功编译后，我们就可以执行安装了，命令为`make install`，执行后，同样可用`echo $?`来验证操作结果。

到此，该源码包便已安装完成了


---
## **系统状态监控**
### **当前系统打开文件**

`lsof`（**list open files**）是一个列出当前系统打开文件的工具。在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。所以如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个文件描述符，无论这个文件的本质如何，该文件描述符为应用程序与基础操作系统之间的交互提供了通用接口。因为应用程序打开文件的描述符列表提供了大量关于这个应用程序本身的信息，因此通过`lsof`工具能够查看这个列表对系统监测以及排错将是很有帮助的。

**查看22端口占用情况**

使用命令`sudo lsof -i TCP:22`
```
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    1115  root    5u  IPv6  29937      0t0  TCP *:ssh (LISTEN)
sshd    1115  root    7u  IPv4  29939      0t0  TCP *:ssh (LISTEN)
sshd    4890  root    5u  IPv4  61606      0t0  TCP centos:ssh->192.168.0.106:58988 (ESTABLISHED)
sshd    4907 sanlo    5u  IPv4  61606      0t0  TCP centos:ssh->192.168.0.106:58988 (ESTABLISHED)
sshd    6325  root    5u  IPv4  69237      0t0  TCP centos:ssh->192.168.0.106:53377 (ESTABLISHED)
sshd    6329 sanlo    5u  IPv4  69237      0t0  TCP centos:ssh->192.168.0.106:53377 (ESTABLISHED)
```
### **如何查看系统变量**
- 执行env可以查看系统的环境变量，如主机的名称、当前用户的SHELL类型、当前用户的家目录、当前系统所使用的语言等。
- 执行set可以看到系统当前所有的变量，其中包括了：
  - 系统的所有预设变量，这其中既包括了env所显示的环境变量，也包含了其它许多预设变量。
  - 用户自定义的变量。

### **监控系统的状态**

**使用`w`命令查看当前系统整体上的负载**
```
# w
 20:33:11 up 309 days, 10:03,  1 user,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    113.102.224.86   20:33    3.00s  0.00s  0.00s w
```
需要关注的是第一行的最后一个部分——`load average`，这里的3个数字分别表示了系统在1分钟/5分钟/15分钟内的平均负载值，值越大说明服务器压力就越大。

那么，如何看负载是不是太满了呢？其实这个值是与服务器的物理CPU做对比的，那么只要负载值不超过物理CPU数量即可；如当前服务器有两个CPU，那么就尽量不要让负载值超过2。

### **用`vmstat`命令查看系统具体负载**
`vmstat`命令打印的结果共分为6部分：`procs`、`memory`、`swap`、`io`、`system`和`cpu`，其中又有许多的细分字段，这里我们重点关注`r`、`b`、`si`、`so`、`bi`、`bo`、`wa`字段。
```
# vmstat 1 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 707116  15392 177284    0    0    81     5  110  267  0  0 99  1  0
 0  0      0 707100  15392 177284    0    0     0     0  121  274  1  0 99  0  0
 0  0      0 706712  15392 177284    0    0     0     0  107  254  0  1 99  0  0
 0  0      0 706696  15392 177284    0    0     0    40   94  235  0  0 100  0  0
 0  0      0 706712  15392 177284    0    0     0     0   93  231  0  0 100  0  0
```
- r(run)：表示正在运行或等待CPU时间片的进程数，**该数值如果长期大于服务器CPU的个数，则说明CPU资源不够用了**。
- b(block)：表示等待资源(I/O、内存等)的进程数。举个例子，当磁盘读写非常频繁时，写数据就会非常慢，此时CPU运算很快就结束了，但进程需要把计算的结果写入磁盘，这样进程的任务才算完成，因此这个任务只能慢慢等待磁盘了。**该数值如果长时间大于1，则需要查一下具体是缺的哪项资源**。
- si和so：分别表示由交换区写入内存的数据量以及由内存写入交换区的数据量；**一般情况下，si、so的值都为0，如果si、so的值长期不为0，则表示系统内存不足**，需要借用磁盘上的交换区，由于这往往对系统性能影响极大，因此需要考虑是否增加系统内存。
- bi和bo：分别表示从块设备读取数据的量和往块设备写入数据的量；如果这两个值很高，那么表示磁盘I/O压力很大。
- wa：表示I/O等待所占用CPU的时间百分比。**wa值越高，说明I/O等待越严重。如果wa值超过20%，说明I/O等待严重**。
另外，`vmstat`命令后可带两个数字，第一个数字表示每多少秒打印一次结果，第二个数字表示总共打印多少次结果；如果只有第一个数字，则会不停地打印结果，直到你终止该命令。

### **用top命令显示进程所占的系统资源**
`top`命令的结果有很多信息，但我们主要用它来监控进程所占的系统资源。`top`命令的结果每隔3秒变1次，它的特点是把占用系统资源（CPU、内存、磁盘I/O等）最高的进程放到最前面。
```
  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
    1 root      20   0   41060   3576   2396 S  0.0  0.4   0:00.89 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd
    3 root      20   0       0      0      0 S  0.0  0.0   0:00.00 ksoftirqd/0
```
这里面我们主要关注RES（所占内存大小）、%CPU、%MEM（占用内存的百分比）、COMMAND这4个字段。

另外，如果需要一次性打印系统资源的使用情况，可以使用`top - bn1`。

### **监控网卡流量**
**使用sar命令查看网卡流量历史记录**

使用`sar`命令前可能需要先进行安装：`yum install -y sysstat`。

使用方法是：`sar -n DEV`，第一次使用时会报错，因为还没有生成相应的数据记录。打印出来的结果里有很多字段，我们关注`rxpck/s`和`rxkB/s`。

- `rxpck/s`表示网卡每秒收取的包的数量，如果数值大于4000则考虑是被攻击了。
- `rxkB/s`表示网卡每秒收取的数据量（单位为KB）。

**使用nload命令监控网卡实时流量**

使用nload前需进行安装：`yum install -y epel-release;yum install -y nload`。

使用起来也很简单，直接使用`nload`命令则可动态显示当前的网卡流入/流出的流量。

### **使用free命令查看内存使用状况**
为了检查内存是否够用，除了`vmstat`外，我们还可以使用更直接有效的`fre`e命令：`free -h`。
```
# free -h
              total        used        free      shared  buff/cache   available
Mem:           992M        141M        462M        516K        388M        714M
Swap:          1.0G          0B        1.0G
```
- total：内存总量，相当于used+free+buff/cache=used+available。
- used：已真正使用的内存量。
- free：剩余（未被分配）的内存量。
- shared：不关注。
- buff/cache：缓解CPU和I/O速度差距所用的内存缓存区，由系统预留出来备用，但如果剩余内存都不够用了，那么这部分也是可以挪用出来供服务来使用的。
- available：可用内存，相当于free+buff/cache。


### **使用ps命令查看系统进程**
与`top`命令类似，`ps`命令也是用来查看系统具体进程占用资源的情况；由于`top`命令本身是动态的，而`ps`命令是非动态的（相当于执行命令时的一个快照），因此`ps`命令的功能实际上更接近于`top -bn1`。

### **使用netstat命令查看网络情况**
`netstat`的功能非常强大，这里举两个实际使用场景：`netstat -lnp`（打印当前系统启动哪些端口）和`netstat -an`（打印网络连接状况）。
```
# netstat -an
Proto Recv-Q Send-Q Local Address           Foreign Address         State  
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
tcp        0      0 172.18.63.215:36492     140.205.140.205:80      ESTABLISHED
netstat -lnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      1604/mysqld         
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1636/httpd          
tcp        0      0 0.0.0.0:21              0.0.0.0:*               LISTEN      1639/vsftpd         
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      674/sshd            
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      1636/httpd
```

---
## **SSH安全登录**

Linux系统通过sshd(ssh daemon)服务实现远程登录的功能，其默认端口是**22**，此服务为Linux系统预装，并预设开机自启，因此不需要额外设置便能够实现Linux远程登录。

**Linux系统上的ssh客户端——openssh**

Windosw系统上有许多软件可以实现ssh远程登录，比如说**putty**、**SecureCRT**、**Xshell**等，那么，我们在Linux系统上，应该使用哪个ssh客户端呢？这里推荐使用**openssh-clients**(简称**openssh**)。

**ssh的校验登录方式**

ssh支持两种方法进行远程校验登录：使用密码登录和使用密钥登录。

### **使用密码登录**
使用命令`ssh user@host`，如`ssh root@192.168.80.128`，系统会进一步提示输入密码，正确输入后，你已经成功登录到远程服务器上了，你的一切操作都跟本地操作无异。

### **使用密钥认证登录**
由于密码登录存在易泄露、易破解的问题，因此一般主张使用ssh远程登录时，仅使用密钥进行登录，并关闭密码远程登录的权限。

所谓密钥认证，实际上是使用了一对加密的字符串：公钥(**public key**)和私钥(**private key**)，其一般使用RSA算法生成。公钥用于加密，任何人都可以看到其内容；而私钥用于解密，只有拥有者才能看到其内容；那么显而易见，即使坏人拿到公钥，也无法解读加密后的内容，因此可以保证内容的保密性。

下面介绍具体的操作：

1. 为本地本用户账号生成一个密钥对：`ssh-keygen`，系统会进一步提示你输入密钥存放的目标目录，按回车键保持默认即可（默认为`~/.ssh`）；接下来会提示你输入密钥的密码，一般留空即可（直接按回车键）；此时会告知你密钥对已成功生成。
2. 查看刚生成的公钥内容：`cat ~/.ssh/id_rsa.pub`。
3. 方法有两个：
   - 使用命令`ssh-copy-id user@host`即可把本地的公钥复制到远程服务器的`authorized_keys`文件上，并给相应的目录和文件设置好合适的权限，详情请看这里SSH-COPY-ID。
   - 复制公钥的全部内容，粘贴到远程服务器的`~/.ssh/authorized_keys`里。远程服务器上如果不存在这个`authorized_keys`文件，则直接新建一个；如果已存在，则注意需要另起一行，加到此文件末尾。这一个过程也可使用以下命令进行操作：`$ ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub`。需要注意，使用此手动方法，需要相应的设置好`authorized_keys`文件的权限，推荐为`600`。
4. 大功告成，从此你使用ssh远程登录，就不需要再输入密码了。
使用ssh远程执行命令并获得执行结果

**例子1：复制公钥**

上述的`$ ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub`就是一个很好的例子：

1. `ssh user@host`，表示登录远程主机；
2. 单引号中的`mkdir .ssh && cat >> .ssh/authorized_keys`，表示登录后在远程shell上执行的命令；
3. `mkdir -p .ssh`的作用是：如果用户主目录中的`.ssh`目录不存在，就创建一个；
4. `cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub`的作用是：将本地的公钥文件`~/.ssh/id_rsa.pub`，重定向追加到远程文件`authorized_keys`的末尾。

**例子2：查看远程主机是否运行进程httpd**

`$ ssh user@host 'ps ax | grep [h]ttpd'`

**退出当前ssh会话**

使用`exit`命令或`logout`命令即可。