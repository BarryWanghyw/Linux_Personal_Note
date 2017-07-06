# Linux之AWK进阶 2017.2.12

## 1. NF和$NF区别  

1. NF是awk的内建变量，代表每行的字段数量，$NF代表：最后一列(Field)，

   {print NF} 是输出了域个数，{print $NF}是输出最后一个字段的内容：

```shell
[root@rhel6 network-scripts]# pwd
/etc/sysconfig/network-scripts
[root@rhel6 network-scripts]# echo $PWD | awk -F '/' '{print NF}'
4
[root@rhel6 etc]# pwd
/usr/local/etc
[root@rhel6 etc]# echo $PWD | awk -F '/' '{print $NF}'
etc
[root@rhel6 etc]# echo $PWD | awk -F/ '{print $NF}'
etc
[root@rhel6 ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:38:DA:41  
          inet addr:192.168.80.128  Bcast:192.168.80.255  Mask:255.255.255.0
eth1      Link encap:Ethernet  HWaddr 00:0C:29:38:DA:4B  
          inet6 addr: fe80::20c:29ff:fe38:da4b/64 Scope:Link
[root@rhel6 ~]# ifconfig | grep HWaddr | awk '{print $NF}'
00:0C:29:38:DA:41
00:0C:29:38:DA:4B
```

![hello](QQ截图20170213122606.png)

## 2. print $

```shell
awk默认是以行为单位处理文本的,
记录（默认就是文本的每一行）
字段 （默认就是每个记录中由空格或TAB分隔的字符串）
$0就表示一个记录，$1表示记录中的第一个字段。
一般 print $0 就是打印整行内容（$0前面不需要反斜杠），print $1表示只打印每行第一个字段。
```

```shell
[root@rhel7 ~]# cat >> file1 << END
> hello
> world
> ! 666
> he  he
> END
[root@rhel7 ~]# cat file1 
hello
world
! 666
he  he
[root@rhel7 ~]# awk '($1 == "!"){print $2}' file1
666
[root@rhel7 ~]# awk '{print $0}' file1
hello
world
! 666
he  he

[root@rhel7 ~]# cat file1 
hello
world
! 666
he      dhe
[root@rhel7 ~]# awk '($1 == "he"){print $2}' file1
dhe
```

## 3. awk 命令 

### 3.1 **awk 自定义分隔符**

awk 默认的分割符为空格和制表符，我们可以使用 -F 参数来指定分隔符

```shell
 # awk '{print $0}' /etc/passwd			#$0表示当前行
 # awk -F ':' '{print $1}' /etc/passwd
 上面的命令将 /etc/passwd 文件中的每一行用冒号 : 分割成多个字段，然后用 print 将第 1 列字段的内容打印输出
```

**如何在 awk 中同时指定多个分隔符**



```shell
比如现在有这样一个文件 some.log 文件内容如下
Grape(100g)1980raisins(500g)1990plum(240g)1997apricot(180g)2005nectarine(200g)2008
现在我们想将上面的 some.log 文件中按照 "水果名称(重量)年份" 来进行分割

$ awk -F '[()]' '{print $1, $2, $3}' some.log
Grape 100g 1980raisins 500g 1990plum 240g 1997apricot 180g 2005nectarine 200g 2008
在 -F 参数中使用一对方括号来指定多个分隔符，awk 处理 some.log 文件时就会使用 "(" 和 ")" 来对文件的每一行进行分割。
```

### 3.2  **awk 内置变量的使用** 

```shell
$0 这个表示文本处理时的当前行

$1 表示文本行被分隔后的第 1 个字段列

$2 表示文本行被分割后的第 2 个字段列

$3 表示文本行被分割后的第 3 个字段列

$n 表示文本行被分割后的第 n 个字段列

NR 表示文件中的行号，表示当前是第几行

NF 表示文件中的当前行列的个数，类似于 mysql 数据表里面每一条记录有多少个字段

FS 表示 awk 的输入分隔符，默认分隔符为空格和制表符，你可以对其进行自定义设置

OFS 表示 awk 的输出分隔符，默认为空格，你也可以对其进行自定义设置

FILENAME 表示当前文件的文件名称，如果同时处理多个文件，它也表示当前文件名称
```

比如我们有这么一个文本文件 fruit.txt 内容如下，我将用它来向你演示如何使用 awk 命令工具，顺便活跃一下此时尴尬的气氛。。

```shell
peach 100 Mar 1997 China
Lemon 150 Jan 1986 America
Pear 240 Mar 1990 Janpan
avocado 120 Feb 2008 china
```

我们来瞧一瞧下面这些简单到爆炸的例子，这个表示打印输出文件的每一整行的内容

```shell
$ awk '{print $0}' fruit.txt
peach 100 Mar 1997 China
Lemon 150 Jan 1986 America
Pear 240 Mar 1990 Janpan
avocado 120 Feb 2008 china
```

下面这个表示打印输出文件的每一行的第 1 列内容

```shell
$ awk '{print $1}' fruit.txt
peach
Lemon
Pear
avocado
```

下面面这个表示打印输出文件的每一行的第 1 列、第 2 列和第 3 列内容

```shell
$ awk '{print $1, $2, $3}' fruit.txt
peach 100 Mar
Lemon 150 Jan
Pear 240 Mar
avocado 120 Feb
其中加入的逗号表示插入输出分隔符，也就是默认的空格
```

**文件的每一行的每一列的内容除了可以用 print 命令打印输出以外，还可以对其进行赋值**

```shell
$ awk '{$2 = "***"; print $0}' fruit.txt
peach *** Mar 1997 China
Lemon *** Jan 1986 America
Pear *** Mar 1990 Janpan
avocado *** Feb 2008 china
上面的例子就是表示通过对 $2 变量进行重新赋值，来隐藏每一行的第 2 列内容，并且用星号 * 来代替其输出
```

**在参数列表中加入一些字符串或者转义字符之类的东东**，\t 是水平制表(制表符) ,就是键盘上的tab键的功能

```shell
$ awk '{print $1 "\t" $2 "\t" $3}' fruit.txt
peach 100 Mar
Lemon 150 Jan
Pear 240 Mar
avocado 120 Feb
像上面这样，你可以在 print的参数列表中加入一些字符串或者转义字符之类的东东，让输出的内容格式更漂亮，但一定要记住要使用双引号。
```

**awk 内置 NR 变量表示每一行的行号**

```shell
$ awk '{print NR "\t" $0}' fruit.txt
1 peach  100 Mar 1997 China
2 Lemon  150 Jan 1986 America
3 Pear  240 Mar 1990 Janpan
4 avocado 120 Feb 2008 china
```

**awk 内置 NF 变量表示每一行的列数**

```shell
$ awk '{print NF "\t" $0}' fruit.txt
5 peach  100 Mar 1997 China
5 Lemon  150 Jan 1986 America
5 Pear  240 Mar 1990 Janpan
5 avocado 120 Feb 2008 china
```



```shell
$ awk '{print $(NF - 1)}' fruit.txt
1997
1986
1990
2008
上面 $(NF-1) 表示倒数第 2 列， $(NF-2) 表示倒数第 3 列，依次类推。
```

