---
title: Linux学习笔记
date: 2020-06-28
categories: Linux
tags: [Linux]
---

## 文件属性

```
[root@VM_0_14_centos ~]# ls -al
total 144
dr-xr-x---.  9 root root  4096 Jun 16 13:35 .
dr-xr-xr-x. 21 root root  4096 Jun 28 10:23 ..
-rw-------   1 root root 45993 Jun 28 10:23 .bash_history
-rw-r--r--.  1 root root    18 Dec 29  2013 .bash_logout
-rw-r--r--.  1 root root   176 Dec 29  2013 .bash_profile
-rw-r--r--.  1 root root   176 Dec 29  2013 .bashrc
drwxr-xr-x   3 root root  4096 Mar  7  2019 .cache
drwxr-xr-x   4 root root  4096 Sep  2  2019 .config
-rw-r--r--.  1 root root   100 Dec 29  2013 .cshrc
-r--------   1 root root    20 Sep  9  2019 .erlang.cookie
-rw-r--r--   1 root root    54 Sep  2  2019 .gitconfig
drwxr-xr-x   3 root root  4096 Apr  2 16:23 .m2
-rw-------   1 root root   618 Apr  7 18:42 .mysql_history
-rw-------   1 root root    10 Dec 13  2019 .node_repl_history
drwxr-xr-x   6 root root  4096 Dec 12  2019 .npm
-rw-------   1 root root    41 Oct 21  2019 .npmrc
drwxr-xr-x   2 root root  4096 Sep  2  2019 .pip
drwxr-----   3 root root  4096 Sep  9  2019 .pki
-rw-r--r--   1 root root    73 Sep  2  2019 .pydistutils.cfg
drwx------   2 root root  4096 Apr  2 13:56 .ssh
-rw-r--r--.  1 root root   129 Dec 29  2013 .tcshrc
-rw-------   1 root root  8627 Jun 16 13:35 .viminfo
[    1    ][2][ 3 ][ 4 ][  5 ][     6     ] [  7  ]
[  权限 ][连结][拥有者][群组][文件容量][修改日期][档名]
```

### 文件类型与权限

```
[文件][可读、可写、可执行][可读、可写、可执行][无权限]
    -         [rwx]         [rwx]         [---]
[档案类型][档案拥有者权限][档案所属群组权限][其他人权限]
```

第一个字符代表这个文件是[目录、文件或链接文件等等]:

`d`：代表是目录
`-`: 代表是文件
`l`: 代表是链接档
`b`: 代表是可供储存的接口设备
`c`: 代表是串行端口设备

接下来的字符中，以三个为一组，且均为[rwx]的三个参数的组合:

`r`: 代表是可读 read
`w`: 代表是可写 write
`x`: 代表是可执行 execute

### 如何改变文件属性与权限

* chgrp: 改变文件所属群组
* chown: 改变文件拥有者
* chmod: 改变文件的权限，SUID，SGID，SBIT等等的特性。

```
[root@VM_0_14_centos ~]# chgrp [-R] users test
选项和参数：
-R : 进行递归的持续变更，亦即连同次目录下的所有文件、目录都更新成为这个群组之意。常常用在变更某一目录内所有的文件。
```

拷贝文件

```
[root@VM_0_14_centos ~]# cp 来源文件 目标文件
```

**数字类型改变文件权限**

各权限的分数对照表:
r: 4
w: 2
x: 1

每种身份(owner/group/others)各自的三个权限(r/w/x)分数是需要累加的，例如当权限为：[-rwxrwx---]分数则是：
```
owner = rwx = 4+2+1 = 7
group = rwx = 4+2+1 = 7
others = --- = 0+0+0 = 0
```

```
[root@VM_0_14_centos ~]# chmod [-R] xyz 文件或目录
[root@VM_0_14_centos ~]# ls -l
total 4
drwxr-xr-x 2 root root 4096 Jun 28 10:51 test
[root@VM_0_14_centos ~]# chmod 777 test
[root@VM_0_14_centos ~]# ls -l
total 4
drwxrwxrwx 2 root root 4096 Jun 28 10:51 test
如果要将test权限全部设定启用，权限分数为[4+2+1][4+2+1][4+2+1] = 777

[root@VM_0_14_centos ~]# chmod 755 test
[root@VM_0_14_centos ~]# ls -l
total 4
drwxr-xr-x 2 root root 4096 Jun 28 10:51 test

如果要将权限设定为drwxr-xr-x，权限分数为[4+2+1][4+1][4+1] = 755
```

**符号类型改变文件权限**

从之前的介绍可以发现，基本上就九个权限分为(1)`user` (2)`group` (3)`others`三种身份，`a`代表 `all` 全部。

```
chmod [u/g/o/a] [+/-/=] [r/w/x] 文件或目录
```

假如我们要设定一个文件的权限为[-rwxr-xr-x]时，基本上就是:

* user(u): 具有可读、可写、可执行的权限。
* group(g)和others(o)：具有可读与可执行的权限。

```
[root@VM_0_14_centos ~]# chmod u=rwx,go=rx test
```

只想增加这个文件的每个人可写入的权限：

```
[root@VM_0_14_centos ~]# chmod a+w test
```

**权限对文件的重要性**

文件是实际含有数据的地方，包括一般文件、数据库内容文件、二进制可执行文件等等。因此，权限对文件来说，他的意义就是：

* r(read): 可读取此文件的实际内容。
* w(write): 可以编辑、新增或者是修改该文件的内容(但不含删除该文件)。
* x(execute): 该文件具有可以被系统执行的权限。

> window底下的一个文件是否具有执行的能力是有[拓展名]判断的，例如：.exe, .bat 等，但是在Linux底下，我们的文件是否能被执行，则是藉由是否具有`x`这个权限来决定的。

**权限对目录的重要性**

目录主要的内容是记录文件名列表，文件名与目录有强烈的关联。

* r: 表示具有读取目录结构列表的权限，利用 `ls` 将内容列表显示出来。
* w: 表示具有异动该目录结构的权限:
    建立新的文件与目录。
    删除已经存在的文件与目录。
    将已存在的文件或目录更名。
    搬移该目录内的文件、目录位置。
* x: 表示用户能否进入该目录内成为工作目录。

## Linux 目录配置的依据-FHS

Filesystem Hierarchy Standard(FHS) 标准

### 文件目录检视：ls

```
ls -a 全部的文件。
ls -l 长数据串行展示，包含文件和权限等数据。
ls -al 
ls -d 仅列出目录本身，而不是列出目录内的文件。
ls --full-time 以完整时间模式
```

### 复制、删除与移动：cp，rm，mv

```
cp -a 来源文件 目标文件 (完整复制，常用)
cp -i 来源文件 目标文件 (已存在文件或目录，询问是否覆盖)
cp -p 来源文件 目标文件 (连同文件的属性权限\用户\时间，备份常用)
cp -r 来源文件 目标文件 (递归复制，用于目录的复制行为，常用)
cp -l 来源文件 目标文件 (进行硬式结连的连结档建立，而非复制文件本身)
cp -s 来源文件 目标文件 (复制成为符号链接文件，亦即快捷方式)
```


## 参考

* [鸟哥的linux私房菜第四版](https://item.jd.com/12443890.html)