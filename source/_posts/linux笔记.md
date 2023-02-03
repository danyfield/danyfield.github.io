---
title: linux笔记
date: 2023-02-03 14:43:05
tags: "linux"
categories: "linux"
---

### 常用快捷键

`[Ctrl]-d`：可以用于取代exit的输入

### 虚拟机目录

#### 目录含义

`/bin`：存放经常使用的命令

`/home`：存放普通用户的主目录，一般该目录名是以用户的账号命名

`/root`：该目录为系统管理员，也称作超级权限者的用户主目录

`/boot`：Linux启动相关文件

`/lib`：系统开机所需最基本的动态连接共享库，作用类似Windows里的DLL文件

`/lost+found`：一般情况下是空的，当系统非法关机后存放了一些文件

`/etc`：系统管理所需配置和子文件目录

`/user`：用户应用程序和文件

`/proc`：虚拟目录，系统内存映射，访问这个目录获取系统信息

`/srv`：存放服务启动后所需数据

`/sys`：该目录安装了2.6内核新出现的文件系统

`/tmp`：存放临时文件

`/mnt`：存放挂载文件

`/opt`：给主机额外安装软件的目录，即软件存放目录

`/user/local`：软件安装后的目标目录，一般是编译源码的方式安装的程序

#### 颜色含义

蓝色表示目录，白色表示文件，红色表示压缩文件，绿色表示可执行权限

### Vim编辑器

#### 四种模式

##### 命令模式

使用Vim编辑器时默认处于命令模式，该模式下可以移动光标位置，可以通过快捷键对文件内容进行复制、粘贴、删除等操作

##### 编辑模式或输入模式

在命令模式下输入小写字母a或i即可进入编辑模式

##### 末行模式

在命令模式下输入：即可进入末行模式，可以在末行输入命令对文件进行查找、替换、保存、退出等操作

##### 可视化模式

可以做一些列选操作（通过方向键选择某些列的内容，类似Windows鼠标刷黑）

#### 四种模式的关系

![](https://s1.ax1x.com/2023/02/03/pSsnXVS.png)

#### 各模式下的相关操作

##### 命令模式

###### 如何进入命令模式

当使用vim命令直接打开某个文件时，默认进入的就是命令模式；若处于其它模式时连续按两次Esc键也可以返回命令模式

###### 命令模式下能做什么

移动光标、复制粘贴、剪切粘贴删除、撤销与恢复

###### 移动光标到首行或末行

移动光标到首行 => gg

移动光标到末行 => G

###### 翻屏

向上翻屏：`ctrl + b（before）或 PgUp`

向下翻屏：`ctrl + f（after）或 PgDn`

向上翻半屏：`ctrl + u（up）`

向下翻半屏：`ctrl + d（down）`

###### 快速定位光标到指定行

行号 + G，如150G代表快速移动光标到第150行

###### 复制/粘贴

复制当前行（光标所在那一行）	按键：yy	

从当前行开始复制指定行数，如复制5行，5yy

粘贴：在想要粘贴的地方按下p【将粘贴光标所在行的下一行】，如果想粘贴光标所在行之前可以使用P键

###### 剪切/删除

在Vim中，剪切和删除都是`dd`，若未用p进行粘贴就是删除；粘贴了就是剪切

剪切/删除光标所在当前行之后的内容，但是删除之后下一行不上移按键：D（删除之后当前行会变成空白行）

###### 撤销/恢复

撤销：u（undo）

恢复：ctrl + r恢复（取消）之前的撤销操作【重做，redo】

##### 末行模式

###### 如何进入末行模式

进入末行模式的方法只有一个，在命令模式下使用`:`或`/`的方式进入

###### 末行模式下能做什么

文件保存、退出、查找与替换、显示行号、paste模式等

###### 查找/搜索

在命令模式下输入`/`，进入末行模式后输入要查找或搜索的关键词

若在一个文件中存在多个满足条件的结果。在搜索结果中切换上/下一个结果：N/n（大写N代表上一个结果，小写n代表next）

若不需要高亮，则在末行模式输入`:noh`【no highlight】

###### 文件内容的替换

第一步：首先进入末行模式

第二步：根据需求替换内容

只替换光标所在行第一个满足的结果

```
:s/要替换的关键词/替换后的关键词	+	回车
```

替换光标所在行所有满足条件的结果（替换多次，只能替换一行）

```
:s/要替换的关键词/替换后的关键词/g	g=global全局替换
```

针对整个文档中的所有行进行替换，只替换每一行中满足条件的第一个结果

```
:%s/要替换的关键词/替换后的关键词
```

针对整个文档中的所有关键词进行替换（只要满足条件就进行替换操作）

```
:%s/要替换的关键词/替换后的关键词/g
```

###### 显示行号

```
:set nu
【nu = number】，行号
```

```
取消行号 => :set nonu
```

###### set paste模式

为什么要使用paste模式？

在终端Vim中粘贴代码时会发现插入的代码有多余的缩进，且逐行累加。原因是终端把粘贴的文本存入键盘缓存（Keyboard Buffer）中，Vim则把这些内容作为用户的键盘输入来处理。导致在遇到换行符的时候，如果Vim开启了自动缩进，就会默认的把上一行缩进插入到下一行的开头，最终使代码变乱。

在粘贴数据之前开启paste模式：`:set paste`

在粘贴完毕后关闭paste模式：`:set nopaste`

##### 可视化模式

###### 如何进入可视化模式

在命令模式中，按`ctrl+v（可视块）`或`ctrl+V（可视行）`或`v（可视）`，然后根据方向键选择需要复制的区块，按下`y`进行复制，最后按下`p`粘贴

退出可视模式按下`Esc`

#### Vim实用功能

##### 代码着色

通过`:syntax on`或`:syntax off` 开启或关闭代码着色功能

##### 异常退出解决方案

突然关闭终端或断电下退出的情况为异常退出，文件一般为`.文件名称.swp`

解决办法：将交换文件直接删除

### 用户管理

```
添加用户：useradd 用户名
给用户指定密码：passwd 用户名
删除用户（保留家目录）：userdel 用户名
删除用户（删除所有目录）：userdel -r 用户名
查询用户信息：id 用户名
查看当前登录用户：who am i
```

### CentOS7找回root密码

1. 启动系统，进入开机页面，按`e`键进入编辑页面
2. 光标向下移动，找到“Linux16”开头的行数，行末输入`init=/bin/sh`，接着按`ctrl+x`进入单用户模式
3. 在光标闪烁位置输入：mount -o remount,rw /，完成后回车
4. 接着输入passwd，完成后回车，输入密码后回车，再次输入密码，修改成功后会显示passwd...
5. 接着在光标位置输入：touch / .autorelabel，完成后回车，等待系统重启，新密码生效

### 开关机问题

`sync`：将内存中尚未更新的数据写入硬盘中；目前的shutdown/reboot/halt等指令均在关机前进行了sync

`shutdown`：只有root拥有该权限

| 选项与参数 | 说明                                   |
| ---------- | -------------------------------------- |
| -t sec     | -t后面加秒数，即几秒后关机             |
| -k         | 不是真的关机，只是发送警告讯息         |
| -r         | 在系统服务停掉后重新启动（常用）       |
| -h         | 在系统服务停掉后立即关机               |
| -n         | 不经过init程序，直接以shutdown功能关机 |
| -f         | 关机并开机后，强制略过fsck的磁盘检查   |
| -c         | 取消已经在进行的shutdown指令内容       |

`忘记root密码`：

1. 启动虚拟机并按e进入下列页面

   ![](https://img-blog.csdnimg.cn/4426447226b8445b8df7c8e3ee63a1fb.png)

2. 用键盘上下移动光标，到linuix…UTF-8后面加上rd.break,按下Ctrl+x

   ![](https://img-blog.csdnimg.cn/f6ca6a8701094d31a4f55380a88f9b63.png)

3. 接下来进入switch_root页面里，输入

   \# mount -o remount,rw /sysroot

   \# chroot /sysroot

4. 之后就会进入sh-4.2#的页面输入passwd，输入你的新密码（直接输入就行了，不显示字符）

   ![](https://img-blog.csdnimg.cn/181e7534a8804eb4b9acb5fdf0897941.png)

5. 输入touch / .autorelabel

   输入exit回到switch_root页面

   再次输入exit

### 文件属性

#### 使用者与群组

##### 所有者（User）

可以对所属档案设定权限

##### 群组（Group）

方便组内成员互相修改对方的数据，隔绝组外成员

##### 其他人（Others）

不属于文件所有者或文件所属群组的用户

##### 查看文件属性

`ls [-al]`显示文件名及相关属性

![](https://s1.ax1x.com/2023/02/03/pSsnqDf.png)

![](https://s1.ax1x.com/2023/02/03/pSsn7vt.png)

- 第一个字符代表文件类型[目录、档案或链接文件等]
  - 当为[d]则是目录
  - 当为[-]则是档案
  - 若为[|]则表示为连结档（link file）
  - 若是[b]则表示为装置文件内的可供储存的接口设备（可随机存取装置）
  - 若是[c]则表示为装置文件内的串行端口设备，例如键盘、鼠标（一次性读取装置）

### 文件目录指令

#### less指令

显示文件内容时，并不是一次将整个文件加载之后才显示，而是根据显示需要加载内容，对于显示大型文件具有较高的效率

分屏查看文件内容：less 要查看的文件

#### 指令>和指令>>

输出重定向（覆盖）>，追加>>

将列表内容覆盖写入文件：ls -l > 文件

将列表内容追加写入文件：ls -al >> 文件

将文件1内容覆盖到文件2：cat 文件1 > 文件2

将内容追加到文件：echo 内容 >> 文件

#### ln指令

给源文件创建一个链接：ln -s [源文件或目录] [链接名]

#### history指令

```
查看所有历史命令：history	查看最近5条命令：history 5
执行历史编号为5的命令：!5
```

### 日期指令

指定格式显示年月日时分秒：`date "+%Y-%m-%d %H:%M:%S"`

设置日期：date -s 字符串日期

查看日历：cal 【选项】，不指定选项，默认当前月日历

### 查找指令

#### find指令

find从指定目录向下递归遍历各个子目录，将满足条件的文件或目录显示在终端

**常用选项：**

-name查找指定文件`find . -name startup.sh`

使用正则表达式通配符查找匹配的文件`find . -name *.sh`

-type查找指定类型的文件`find ./webapps/ -type d`

查找最近修改过的文件（最近2天内有更新的文件）`find ./logs -mtime -2`

查找指定目录深度下的文件`find /var/log/ -mindepth 2 -name *.log`

根据inode删除文件

```
首先通过ls -li查找inode	ll -i
然后通过find -inum inode号 -delete指定文件
（此方法对于删除上传到服务器后文件名乱码的文件非常有用）
find -inum 2663645 -delete
```

![](https://img-blog.csdnimg.cn/2021040115050495.png)

清除查找到的超过指定时间的日志文件

```
[root@test1 apache-tomcat-9.0.44]# find ./logs/ -mtime +5 -ok rm {} ;
< rm … ./logs/catalina.2021-03-26.log > ? y
< rm … ./logs/localhost.2021-03-26.log > ? y
< rm … ./logs/manager.2021-03-26.log > ? y
< rm … ./logs/host-manager.2021-03-26.log > ? y
< rm … ./logs/localhost_access_log.2021-03-26.txt > ? y
```

查找当前目录下具有指定权限的文件并获取完整路径

```
使用perm参数查找指定权限的文件
find /home/wuhs/apache-tomcat-9.0.44/bin -type f -perm 750 
-exec ls -l {} \;
-rwxr-x— 1 wuhs wuhs 25294 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/catalina.sh
-rwxr-x— 1 wuhs wuhs 1997 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/ciphers.sh
-rwxr-x— 1 wuhs wuhs 1922 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/configtest.sh
-rwxr-x— 1 wuhs wuhs 9100 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/daemon.sh
-rwxr-x— 1 wuhs wuhs 1965 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/digest.sh
-rwxr-x— 1 wuhs wuhs 3382 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/makebase.sh
-rwxr-x— 1 wuhs wuhs 3708 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/setclasspath.sh
-rwxr-x— 1 wuhs wuhs 1902 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/shutdown.sh
-rwxr-x— 1 wuhs wuhs 1904 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/startup.sh
-rwxr-x— 1 wuhs wuhs 5540 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/tool-wrapper.sh
-rwxr-x— 1 wuhs wuhs 1908 Mar 5 05:52 /home/wuhs/apache-tomcat-9.0.44/bin/version.sh
```

查找指定用户的文件`find ./webapps/ -user wuhs -type f -print`

查找指定大小的文件`find . -size +1M -type f`

##### 使用语法及参数说明

###### 使用语法

```
find [路径] [表达式选项] [行动]
```

###### 常用表达式选项参数说明

| 选项                    | 参数说明                                                     |
| ----------------------- | ------------------------------------------------------------ |
| -mount，-xdev           | 只检查和指定目录在同一文件系统下的文件                       |
| -(a/c)min n             | 在过去n分钟内被(读取/修改)过                                 |
| -(a/c)newer file        | 比文件file更晚被(读取/修改)过的文件                          |
| -(a/c)time n            | 在过去n天内被(读取/修改)过的文件                             |
| -empty                  | 空的文件-gid n or -group name                                |
| -ipath p，-path p       | 路径名称符合p的文件，ipath会忽略大小写                       |
| -name name，-iname name | 文件名称符合name的文件，iname会忽略大小写                    |
| -size n                 | 文件大小是n单位，b代表512位元组的区块，c表示字元数，k 表示 kilo bytes，w 是二个位元组 |
| -type b/d/c/p/l/f       | 块设备、目录、字符设备、管道、符号链接、普通文件             |
| -perm                   | 按执行权限查找                                               |
| -user username          | 按文件属主查找                                               |
| -group groupname        | 按组来查找                                                   |
| -depth                  | 指定查找目录深度                                             |
| -follow                 | 若遇到符号链接文件，就跟踪链接所指的文件                     |
| -prune                  | 忽略某个目录                                                 |
| -maxdepth               | 指定查找目录最大深度                                         |
| -mindepth               | 指定查找目录最小深度                                         |
| -version                | 查看版本                                                     |
| -help                   | 获取帮助                                                     |

###### 常用行动参数说明

| 参数          | 参数说明                    |
| ------------- | --------------------------- |
| -delete       | 删除查找到的文件            |
| -exec command | 对查找的文件执行command命令 |
| -ok command   | 执行命令前需要进行确认      |
| -printf       | 格式化输出                  |

###### 多条件组合参数

| 参数 | 参数说明 |
| ---- | -------- |
| -o   | 或者     |
| -a   | 而且     |
| -not | 相反     |

#### locate指令

快速查找指定文件的路径：locate 文件

由于该指令是基于数据库查询，第一次使用必须用updatedb指令创建数据库

##### locate特点

1. locate是基于数据库的查询，速度很快，但不是实时的查询
2. locate是模糊查询
3. 需要对文件的目录有rx的权限

##### locate语法

```
-b --basename 			仅匹配路径名的基本名称 
-c --count				只输出找到的数量
-d --database DBPATH	使用DBPATH指定的数据库，而不是默认数据库 /var/lib/mlocate/mlocate.db
-e --existing			仅打印当前现有文件的条目
-1						安全模式，使用者不会看到权限无法看到的档案，会使速度减慢，因为locate必须至实际的档案系统中取得档案的权限资料
-0 --null				在输出上带有NULL的单独条目
-S --statistics			不搜索条目，打印有关每个数据库的统计信息
-q						安静模式，不显示任何错误讯息
-P --nofollow，-H		检查文件存在时不要遵循尾随的符号链接
-l --limit，-n LIMIT		将输出（或计数）限制为LIMIT个条目
-n						至多显示n个输出
-r --regexp REGEXP		使用基本正则表达式
-q，--quiet				安静模式，不会显示任何错误信息
-o						指定资料库存的名称
-h --help				显示帮助
-i --ignore-case		忽略大小写
-V --version			显示版本信息
```

##### locate的日常使用

###### 查询passwd

```
[root@192 ~]# locate passwd -n 5
/etc/passwd
/etc/passwd-
/etc/pam.d/passwd
/etc/security/opasswd
/usr/bin/gpasswd
```

###### 忽略大小写查询

```
[root@192 ~]# locate -i CONF -n 5
/ansible/roles/webserver/files/httpd.conf
/boot/config-3.10.0-957.el7.x86_64
/boot/grub2/i386-pc/configfile.mod
/etc/GeoIP.conf
/etc/GeoIP.conf.default
```

###### 更新本地数据库

```
[root@192 ~]# updatedb -V
updatedb (mlocate) 0.26
Copyright (C) 2007 Red Hat, Inc. All rights reserved.
This software is distributed under the GPL v.2.

This program is provided with NO WARRANTY, to the extent permitted by law.
```

###### 打印系统数据库的信息

```
root@192 ~]# locate  -S
Database /var/lib/mlocate/mlocate.db:
	13,857 directories
	189,679 files
	11,268,161 bytes in file names
	4,331,363 bytes used to store database
```

#### which指令

用于查看给定命令的绝对路径

##### 常用示例

###### 查找命令所在路径

```
$ which passwd
/usr/bin/passwd
```

#### grep指令

**文本搜索工具，根据用户指定的“模式（过滤条件）”对目标文本逐行进行匹配检查，打印匹配到的行**

从文本文件或管道数据流中筛选匹配的行和数据

| 参数选项    | 解释说明               |
| ----------- | ---------------------- |
| -v          | 排除匹配结果           |
| -n          | 显示匹配行及行号       |
| -i          | 不区分大小写           |
| -c          | 只统计匹配的行数       |
| -E          | 使用egrep命令          |
| -color=auto | 为grep过滤结果添加颜色 |
| -w          | 只匹配过滤的单词       |
| -o          | 只输出匹配的内容       |

![](https://img-blog.csdnimg.cn/e9f2c61829ce4dad8f0b19bdf6338819.png)

### 压缩解压指令

#### tar指令

| 常用选项 | 功能               |
| -------- | ------------------ |
| -c       | 产生.tar打包文件   |
| -z       | 打包同时压缩       |
| -x       | 解包.tar文件       |
| -v       | 显示详细信息       |
| -f       | 制定压缩后的文件名 |

压缩多文件：`tar -zcvf dc.tar.gz /hmoe/bbb/cat.txt /home/bbb/dog.txt `

将bbb文件夹压缩成myb.tar.gz：`tar -zcvf myb.tar.gz bbb`

将文件解压到当前目录：`tar -zxvf myb.tar.gz`

将myb.tar.gz解压到tom目录下：`tar -zxvf myb.tar.gz -C tom`

### Linux组

#### 修改权限chmod

- 方式一：+、-、=变更权限

  u：所有者，g：所有组，o：其他用户，a：所有人

  **给文件的所有者读写执行权限，给所在组读权限，给其他用户执行权限：**chmod u=rwx,g=rx,o=x

  **给其他人增加写权限：**chmod o+w 文件/目录名/所有者/所有组

  **该文件不让所有人执行：**chmod a-x 文件/目录名/所有者/所有组

- 方式二：数字变更权限

  r=4，w=2，x=1

  chmod u=rwx,g=rx,o=x 文件/目录名 相当于 chmod 751 文件/目录名

#### 修改文件所有者

改变所有者：chown	新所有者	文件/目录

改变所有者和所在组：chown	新所有者:新所有组	文件/目录

### 定时任务调度

任务调度：系统在某个时间执行特定的命令或程序

#### 任务调度分类

1. 系统工作：某些重要工作周而复始的进行，如病毒查杀

2. 个别用户工作：个别用户执行某些程序，如打开qq

   **定时任务的设置：crontab [选项]**

| 选项 | 功能                       |
| ---- | -------------------------- |
| -e   | 编辑crontab定时任务        |
| -l   | 查询当前任务调度           |
| -r   | 删除当前用户所有的定时任务 |

#### 重启任务调度

service crond restart

crontab -e 回车然后输入 */1 * * * * ls

| *号位置 | 含义                                         |
| ------- | -------------------------------------------- |
| 第一个  | 一小时当中的第几分钟（分钟）                 |
| 第二个  | 一天当中的第几小时（小时）                   |
| 第三个  | 一月当中的第几天（天）                       |
| 第四个  | 一年中第几个月（月）                         |
| 第五个  | 一周当中的星期几（范围0-7,0和7都表示星期天） |

| 特殊符号 | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| *        | 表示任何时间，比如第一个*表示一小时每分钟都执行一次          |
| ,        | 表示不连续时间，比如"0 8,10 * * *"表示每天8点和10点执行      |
| -        | 表示连续的时间范围，比如"0 2 * * 1-6"表示周一到周六凌晨2点执行一次命令 |
| */n      | 表示每隔多久执行一次，如"*/10 * * * *"表示每10分钟执行一次   |

**特定时间执行案例**

![](https://s1.ax1x.com/2023/02/03/pSsnLb8.png)

#### at定时任务

at命令是一次性定时计划任务 ，at的守护进程atd会以后台模式检查作业队列运行。默认情况下, atd守护进程每60秒检查作业队列,有作业时,会检查作业运行时间,如果时间与当前时间四配，则运行此作业。at命令只执行一次。

在使用at命令的时候，一定要保证atd进程的启动，可以使用相关指令来查看

检测当前进程有哪些：ps -ef

检测acd进程是否在运行：ps -ef | grep atd

命令格式：at 【选项】 【时间】，ctrl+d结束at命令输入

| 选项        | 功能                                                   |
| ----------- | ------------------------------------------------------ |
| -m          | 当指定的任务被完成后,将给用户发送邮件,即使没有标准输出 |
| -I          | atq（显示系统中待执行的任务列表）的别名                |
| -d          | atrm（删除待执行任务队列中的任务）的别名               |
| -v          | 显示任务将被执行的时间                                 |
| -V          | 显示版本信息                                           |
| -c          | 打印任务的内容到标准输出                               |
| -q 队列     | 使用指定的队列                                         |
| -f 文件     | 从指定文件读入任务而不是从标准输入读入                 |
| -t 时间参数 | 以时间参数的形式提交要运行的任务                       |

##### at指定时间方式

1. hh:mm（小时：分钟）24小时制指定时间，若该时间已过，会到第二天执行
2. 使用midnight (深夜)， noon (中午)， teatime (饮茶时间，一般是下午4点)等模糊词来指定时间
3. 采用12小时计时制，在后面加上am (上午)或pm (下午)说明是上午还是下午
4. 指定命令执行的具体日期，指定格式为month day(月日)或mm/dd/yy (月/日/年)或dd.mm.yy
5. 使用相对计时法，如：now + 5 minutes
6. 直接使用today、tomorrow指定完成命令的时间

###### 使用示例

一天后凌晨12点执行 /bin/ls home

![](https://img-blog.csdnimg.cn/20210717194539556.png)

- 统计/opt下文件个数：`ls -l /opt | grep "^-" | wc -l`（^-是以-开头的文件，wc统计个数）
- 统计/opt下目录个数：`ls -l /opt | grep "^d" | wc -l`
- 统计/opt文件夹下文件的个数，包括子文件夹里的：`ls -lR /opt | grep "^-" | wc -l `
- 统计/opt文件夹下目录的个数，包括子文件夹里的：`ls -lR /opt | grep "^d" | wc -l`
- 以树状显示目录结构：`tree 目录`，注意默认是没有安装tree的，安装要root权限，安装tree命令：`yum install tree`

### 进程管理

使用`ps 【选项】`查看当前系统哪些进程在执行

| 选项 | 功能                       |
| ---- | -------------------------- |
| -a   | 显示当前终端的所有进程信息 |
| -u   | 以用户的格式显示进程信息   |
| -x   | 显示后天进程运行的参数     |
| -e   | 显示所有进程               |
| -f   | 全格式                     |

#### 终止进程

kill 【选项】 进程号

killall 进程名称

常用选项：-9：强迫进程立即停止

#### 查看进程树

pstree 【选项】

常用选项

-p：显示进程的PID

-u：显示进程的所属用户

#### 服务管理

service 服务名 [start|stop|restart|reload|status]

在CentOS7后很多服务不再使用service，而是systemctl

查看service指令管理的服务：ls -l /etc/init.d

#### chkconfig指令

用于检查、设置系统的各种服务

查看服务：chkconfig --list [| grep xxx]

给服务在指定运行级别下设置开关：chkconfig --level 5 服务名 on/off

#### systemctl指令

管理操作系统和服务的命令，是systemd（system daemon，操作系统服务管理器）交互的主要工具，实现的功能包含了service和chkconfig这两个命令的功能

```
#例：查看当前防火墙的状况，关闭防火墙和启动防火墙
1、systemctl status firewalld.service
2、systemctl stop firewalld.service
3、systemctl start firewalld.service
```

#### 防火墙

![](https://s1.ax1x.com/2023/02/03/pSsnbKP.png)

防火墙开启时xshell访问linux需要打开22端口号，关闭后可以直接访问

#### firewall指令

打开端口：firewall-cmd --permanent --add-port=端口号/协议

关闭端口：firewall-cmd --permanent --remove-port=端口号/协议

重新载入才能生效：firewall-cmd --reload

查看所有开放端口：firewall-cmd --zone=public --list-port

查询端口是否开放：firewall-cmd --query-port=端口/协议

#### 动态监控进程

top与ps命令相似，用于显示正在执行的进程，top和ps最大的不同之处在于top在执行一段时间可以更新正在运行的进程

指令：top	【选项】

| 选项    | 功能                                       |
| ------- | ------------------------------------------ |
| -d 秒速 | 指定top命令每隔几秒更新，默认3秒           |
| -i      | 使top不显示任何闲置或僵死进程              |
| -P      | 通过指定监控进程ID来仅仅监控某个进程的状态 |

交互操作说明

| 操作 | 功能                                    |
| ---- | --------------------------------------- |
| P    | 以CPU使用率排序，从大到小，默认就是此项 |
| M    | 以内存的使用率排序，从大到小            |
| N    | 以PID排序，从大到小                     |
| Q或q | 退出top                                 |

### rpm和yum

#### rpm

用于下载包的打包及安装工具，它生成具有.rpm扩展名的文件，类似windows的setup.exe；

```
查询所有安装rpm软件包：rpm -qa
查询软件包是否安装：rpm -q 软件包名
查询软件包信息：rpm -qi 软件包名
查询软件包中的文件：rpm -ql 软件包名
查询文件所属的软件包：rpm -qf 文件全路径名
卸载软件包：rpm -e 软件包
安装软件包：rpm -ivh 安装的全路径
```

#### yum

Shell前端软件包管理器，基于RPM包管理，能够从指定的服务器自动下载RPM包并安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包

查询yum服务器是否有需要安装的软件：yum list | grep xx软件列表

安装指定的yum包：yum install xx下载安装

### Java环境安装

#### jdk安装

##### 安装步骤

1. 创建jdk文件夹：mkdir /opt/jdk

2. 通过xftp传输Linux版本的jdk安装包到/opt/jdk目录下

3. 进入jdk目录：cd /opt/jdk

4. 解压jdk安装包：tar -zxvf jdk-8u261-linux-x64.tar.gz

5. 创建java文件夹：mkdir /usr/local/java

6. 移动jdk安装文件：mv /opt/jdk/jdk 1.8.0_261/ /usr/local/java

7. 配置环境变量：vim /etc/profile

8. 在profile文末添加

   ```
   export JAVA_HOME=/usr/local/java/jdk1.8.0_261
   export PATH=$JAVA_HOME/bin:$PATH
   ```

9. 让编辑过的环境变量生效：source /etc/profile

#### Tomcat安装

##### 安装步骤

1. 新建tomcat目录：mkdir /opt/tomcat
2. 通过xftp传输Linux版本的tomcat安装包到/opt/tomcat目录下
3. 进入tomcat目录：cd /opt/tomcat
4. 解压tomcat：tar -zxvf apache-tomcat-8.5.69.tar.gz ，下载core核心包地址https://tomcat.apache.org/download-80.cgi
5. 进入tomcat的bin目录：cd apache-tomcat-8.5.69/bin，启动tomcat：./startup.sh
6. 开放端口8080：firewall-cmd --permanent --add-port=8080/tcp
7. 重新载入生效：firewall-cmd --reload
8. 测试是否打开端口号：firewall-cmd --query-port=8080/tcp

#### MySQL安装

##### 安装步骤

1. 新建mysql文件夹，进入：mkdir /opt/mysql

2. Xftp将安装包传输到/opt/tomcat目录下

3. 进入mysql目录：cd /opt/mysql

4. 解压mysql安装包：tar -xvf mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar

5. 查询mariadb： rpm -qa | grep mari。注意centos7.6自带的类mysql数据库是mariadb，会跟mysql冲突，要先删除

6. 卸载mariadb：rpm -e --nodeps mariadb-libs ，rpm -e --nodeps marisa

7. 开始安装mysql

   ```
   rpm -ivh mysql-community-common-5.7.26-1.el7.x86_64.rpm 
   rpm -ivh mysql-community-libs-5.7.26-1.el7.x86_64.rpm 
   rpm -ivh mysql-community-client-5.7.26-1.el7.x86_64.rpm
   rpm -ivh mysql-community-server-5.7.26-1.el7.x86_64.rpm 
   ```

8. 启动服务：systemctl start mysqld.service

9. 开始设置root密码；Mysql自动给root用户设置随机密码，运行grep “password” /var/log/mysqld.log可看到当前密码

   运行mysql -u root -p，复制粘贴输入上述密码

   修改密码

   运行如下命令使密码生效

   ```
   flush privileges;
   ```

### Shell编程

#### Shell脚本执行方式

##### 脚本格式要求

脚本以#!/bin/bash开头，需要可执行权限

##### 常用执行方式

1. 输入脚本的绝对路径或相对路径（./xxx.sh）
2. sh+脚本

#### shell变量

1. 定义变量：变量名=值
2. 撤销变量：unset 变量
3. 声明静态变量：readonly 变量，静态变量不能unset
4. 显示当前shell中所有变量：set
5. 一般情况下var和{var}没有区别，但是用${}会较精确的界定变量名称的范围

```
A=`date`	#运行反引号内的命令，将结果返回给变量A
A=$(date)	#等同于上面的语句
```

##### 设置环境变量（全局变量）

export 变量名=变量值（将shell变量输出为环境变量）

source 配置文件（让修改后的配置信息立即生效）

echo $变量名（查询环境变量的值）

###### shell注释

- 单行注释：通过一个'#'实现，开始部分的#不是用于注释的

- 多行注释

  ```
  <<xxxx
  	注释内容
  xxxx
  ```

  xxxx可以为任意字符串，中间部分为注释

##### 位置参数变量

用于在执行脚本时获取命令行的参数信息

```
$n	#n为数字，表示第几个参数，10以上需要用大括号如${10}
$*	#命令行所有参数，将所有参数当做一个整体
$@	#命令行所有参数，将每个参数分别输出
$#	#命令行中所有参数的个数
```

##### 预定义变量

事先定义好的变量，直接在脚本中使用

```
$$	#当前进程的进程号PID
$!	#后台运行的最后一个进程的进程号PID
$?	#最后一次执行命令的返回状态；若为0则表示正确执行
```

#### 运算符

```
$((表达式))或$[表达式]或expr m+n
```

#### 条件判断

```
if [ 条件判断 ]
then
代码
fi

#若输入参数大于等于60则输出及格了，若小于60则输出不及格
if [ $1 -ge 60 ]
then
	echo	"及格了"
elif [ $1 -lt 60 ]
then
	echo	"不及格"
fi


case $1 in
"1")
	echo	"周一"
;;
"2")
	echo	"周二"
;;
*)
	echo	"other"
;;
esac


for i in "$*"
do
	echo	"num is $i"
done

SUM=0
i=0
while [ $i -le $1]
do
	SUM=$[$SUM+$i]
	i=$[$i+1]
done
echo	"结果=$SUM"
```

#### read读取控制台输入

##### 基本语法

read [选项] [参数]

##### 选项

-p：指定读取值时的提示符；

-t：指定读取值时等待的时间；

##### 参数

变量：指定读取值的变量名

```
#!/bin/bash
#案例1：读取控制台输入一个num1值
read -p "请输入指定的num1=" NUM1
echo "输入的num1=$NUM1"
#案例2：读取控制台输入一个num2值，在5秒内输入
read -t 5 -p "请输入num2=" NUM2
echo "输入的num2=$NUM2"
```

#### 函数

##### basename

返回一个字符串参数的基本文件名称

###### 语法

basename String [ Suffix ]

###### 描述

读取String参数，删除以/结尾的前缀及任何指定的Suffix参数，并将剩余基本文件名称写至标准输出

1. 若String包含的都是斜杠字符，则将字符串更改为单个/

2. 若指定Suffix参数和剩余字符相同，则不修改此字符

   ```
   basename /u/dee/desktop/cns.boo cns.boo		=>	cns.boo
   ```

3. 若只与字符串的后缀相同，则除去指定后缀

   ```
   basename /u/dee/desktop/cns.boo .boo		=>	cns
   ```

###### 常见应用

1. 显示一个shell变量的基本名称

   ```
   basename $WORKFILE
   #显示WORKFILE值的基本名称，若WORKFILE变量值是/home/jim/program.c文件，则此命令显示program.c 
   ```

2. 构造一个和另一个文件名称相同（除了后缀）的文件名称

   ```
   OFILE=`basename $1 .c`.o
   ```

##### dirname

获取某个文件或目录的上一级目录

###### 应用实例

```
dirname /home/users/aoquan/
#输出结果：/home/users
```

##### 自定义函数

###### 应用实例

```
#!/bin/bash
#案例：计算输入两个参数的和，getSum
function getSum(){
        sum=$[$n1+$n2]
        echo "和是=$sum"
}
#输入两个值
read -p "第一个值=" n1
read -p "第二个值=" n2
#调用函数
getSum $n1 $n2
```

#### 数据备份案例

###### 需求分析

每天凌晨2:30备份数据库smile到/data/backup/db；备份开始和结束给出相应的提示信息；备份文件以备份时间为文件名，并打包成.tar.gz形式；在备份的同时检查是否有7天前备份的数据库文件，若有就将其删除

`vim /usr/sbin/mysql_db_backup.sh 内容如下`

```shell
#!/bin/bash
#备份目录
BACKUP=/data/backup/db
#当前时间
DATATIME=$(date + %Y-%m-%d_%H%M%S)
#数据库地址
HOST=localhost
#数据库用户名
DB_USER=root
#数据库密码
DB_PW=root
#备份的数据库名
DATABASE=smile
#若备份目录不存在则创建
[ ! -d "${BACKUP}/${DATATIME}" ] && mkdir -p "${BACKUP}/${DATATIME}"
#备份数据库
echo "开始备份数据库${DATABASE}"
mysqldump -u${DB_USER} -p${DB_PW} --host=${HOST} -q -R --databases ${DATABASE} | gzip > ${BACKUP}/${DATATIME}/$DATATIME.sql.gz
#将备份文件夹处理成.tar.gz的格式
cd ${BACKUP}
tar -zcvf $DATATIME.tar.gz ${DATATIME}
#删除对应的备份目录
rm -rf ${BACKUP}/${DATATIME}
#删除7天前的备份
find ${BACKUP} -atime +7 -name "*.tar.gz" -exec rm -rf {} \;
echo "备份数据库${DATABASE}成功"
```

`定时脚本`

`命令行敲crontab -e ，接着输入如下内容`

```
30 2 * * * /usr/sbin/mysql_db_backup.sh
```

### 日志管理

#### 系统常用日志

| 日志文件                                 | 说明                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| <font color=red>/var/log/boot.log</font> | 系统启动日志                                                 |
| <font color=red>/var/log/cron</font>     | 记录与系统定时任务相关的日志                                 |
| /var/log/cups/                           | 记录打印信息的日志                                           |
| /var/log/dmesg                           | 记录系统开机时自检信息，可以使用dmesg命令查看内核自检信息    |
| /var/log/btmp                            | 记录错误登录的日志，该日志为二进制文件，不能用vi查看，要使用lastb命令查看 |
| <font color=red>/var/log/lasllog</font>  | 记录系统所有用户最后一次登录时间的日志，也只能使用lastlog命令查看 |
| <font color=red>/var/log/mailog</font>   | 记录邮件信息的日志                                           |
| <font color=red>/var/log/message</font>  | 记录系统重要消息的日志，若系统出现问题，首先应该检查的就是该日志文件 |
| <font color=red>/var/log/secure</font>   | 记录验证和授权方面的信息，只要涉及账户和密码的程序都会记录，如系统的登录、ssh的登录、sudo授权 |
| /var/log/wtmp                            | 永久记录所有用户的登录、注销信息、同时记录系统的启动、重启、关机事件，只能使用lastb命令查看 |
| <font color=red>/var/log/ulmp</font>     | 记录当前已登录用户的信息，随着用户的登录和注销不断变化，使用w、who、users等命令查看 |

### 挂载

除了根文件系统，其他所有文件系统都要先挂载到根文件系统（系统启动时安装的第一个文件系统，内核映像所在的文件系统）中的某个目录后才能访问，而`挂载到某个目录`中的`某个目录`就是挂载点

Linux挂载文件系统命令：`mount device dir`，device为要挂载的设备文件，dir为挂载点；一个磁盘上的不同分区都可以看作不同设备