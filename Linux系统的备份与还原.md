#Linux系统的备份与还原 

备份Linux系统的策略有很多，比如使用dd命令直接克隆硬盘分区：

```
sudo dd if=/dev/sda1 of=/dev/sdb1
```

或者，使用tar将硬盘上的文件打包：

```
cd /
sudo tar cvpzf backup.tgz --exclude=/proc --exclude=/mnt --exclude=/sys --exclude=/backup.tgz /
```

还原系统的命令是：

```
sudo dd if=/dev/sdb1 of=/dev/sda1
或
tar xvpfz backup.tgz -C /
```

但是我最终选择的是rsync命令。

## 1. 使用tar命令

和 备份Windows系统不同，如果你要备份Ubuntu系统(或者其它任何Linux系统)，你不再需要像Ghost这类备份工具。事实上，Ghost 这类备份工具对于Linux文件系统的支持很糟糕，例如一些Ghost版本只能完善地支持Ext2文件系统，如果你用它来备份Ext3文件系统，你可能会 丢失一些宝贵的数据。

```shell
1. 备份系统

我该如何备份我的Ubuntu系统呢？很简单，就像你备份或压缩其它东西一样，使用TAR。和Windows不同，Linux不会限制root访问任何东西，你可以把分区上的所有东西都扔到一个TAR文件里去！

首先成为root用户：
$ sudo su

然后进入文件系统的根目录(当然，如果你不想备份整个文件系统，你也可以进入你想要备份的目录，包括远程目录或者移动硬盘上的目录)：
# cd /

下面是我用来备份系统的完整命令：
# tar -cvpzf backup.tgz --exclude=/proc --exclude=/lost+found --exclude=/backup.tgz --exclude=/mnt --exclude=/sys / /.*
或#tar cvpzf backup.tgz / /.* --exclude=/proc --exclude=/lost+found --exclude=/backup.tgz --exclude=/mnt --exclude=/sys

# 特别注意的是tar打包不会把隐藏文件给打包，所以要加上.*：
#tar -czvfp test.tar.gz    .*   * 打包压缩当前目录的所有文件包括隐藏文件
#tar -czvfp test.tar.gz    .[!.]*   * 打包压缩当前目录的所有文件包括隐藏文件但不包换. 和..
#-p是保留权限；

让我们来简单看一下这个命令：

“tar”当然就是我们备份系统所使用的程序了。

“cvpfz”是tar的选项，意思是“创建档案文件”、“p代表保持权限”(保留所有东西原来的权限)、“使用gzip来减小文件尺寸”。

“backup.gz”是我们将要得到的档案文件的文件名。

“/”是我们要备份的目录，在这里是整个文件系统。

在 档案文件名“backup.gz”和要备份的目录名“/”之间给出了备份时必须排除在外的目录。有些目录是无用的，例如“/proc”、“/lost+ found”、“/sys”。当然，“backup.gz”这个档案文件本身必须排除在外，否则你可能会得到一些超出常理的结果。如果不把“/mnt”排 除在外，那么挂载在“/mnt”上的其它分区也会被备份。另外需要确认一下“/media”上没有挂载任何东西(例如光盘、移动硬盘)，如果有挂载东西， 必须把“/media”也排除在外。

有人可能会建议你把“/dev”目录排除在外，但是我认为这样做很不妥，具体原因这里就不讨论了。

执行备份命令之前请再确认一下你所键入的命令是不是你想要的。执行备份命令可能需要一段不短的时间。

备份完成后，在文件系统的根目录将生成一个名为“backup.tgz”的文件，它的尺寸有可能非常大。现在你可以把它烧录到DVD上或者放到你认为安全的地方去。

在备份命令结束时你可能会看到这样一个提示：’tar: Error exit delayed from previous errors’，多数情况下你可以忽略它。

你还可以用Bzip2来压缩文件，Bzip2比gzip的压缩率高，但是速度慢一些。如果压缩率对你来说很重要，那么你应该使用Bzip2，用“j”代替命令中的“z”，并且给档案文件一个正确的扩展名“bz2”。完整的命令如下：
# tar -cvpjf backup.tar.bz2 –-exclude=/proc –-exclude=/lost+found –-exclude=/backup.tar.bz2 –-exclude=/mnt –-exclude=/sys /
```

```shell
2. 恢复系统

在进行恢复系统的操作时一定要小心！如果你不清楚自己在做什么，那么你有可能把重要的数据弄丢，请务必小心！

接着上面的例子。切换到root用户，并把文件“backup.tgz”拷贝到分区的根目录下。

在 Linux中有一件很美妙的事情，就是你可以在一个运行的系统中恢复系统，而不需要用boot-cd来专门引导。当然，如果你的系统已经挂掉不能启动了， 你可以用Live CD来启动，效果是一样的。你还可以用一个命令把Linux系统中的所有文件干掉，当然在这里我不打算给出这个命令！

使用下面的命令来恢复系统：
# tar xvpfz backup.tgz -C /
# 还原之前最好把原有目录内的文件删除，因为如果将被还原系统的目录有不同名的文件不会被覆盖，同名的会被覆盖重写；

如果你的档案文件是使用Bzip2压缩的，应该用：
# tar xvpfj backup.tar.bz2 -C /
#还原之前最好把原有目录内的文件删除，因为如果将被还原系统的目录有不同名的文件不会被覆盖，同名的会被覆盖重写；

注意：上面的命令会用档案文件中的文件覆盖分区上的所有文件。

执行恢复命令之前请再确认一下你所键入的命令是不是你想要的，执行恢复命令可能需要一段不短的时间。

恢复命令结束时，你的工作还没完成，别忘了重新创建那些在备份时被排除在外的目录：
# mkdir proc
 # mkdir lost+found
 # mkdir mnt
 # mkdir sys
等等

当你重启电脑，你会发现一切东西恢复到你创建备份时的样子了！

用户参照了上面的教程做的备份和恢复，普遍反映：重启电脑后还是会一直提示用户名和密码 输入以后一闪还是提示用户名和密码，反正就是登不进系统。

网络上有人提到：请教了高手，找到了解决方法，还原后，执行以下命令再重启，即可解决这个问题：restorecon -Rv /
```

## 2. 使用dd if of命令

```shell
用tar会有各种各样难避免的问题，建议直接用启动盘启动系统后利用DD命令来整盘复制。
比如，举个最简单的例子，你只有两分区(以及各自挂载点） /dev/hda1(/) 和/dev/hda2(swap),而且你linux是在/dev/hda1(/)中，于是，你可以另接一硬盘（假设/dev/hdb1)用启动盘重启后，执行命令：
mount /dev/hdb1 /mnt/hdb1 -t ext2 #挂载到目录，没目录自已建，不用多解释了
dd if=/dev/hda1 of=/mnt/hdb1/sysimage.bak 
dd if=/dev/hda of=/mnt/hdb1/mbr.bak bs=1 count=512 #备份MBR和分区表，若分区表不备份就把512改446

等吧。。分区越大时间越久，dd没有ghost这些软件聪明。哪怕你的linux只有1M但你的分区有1G，那备份时间就是复制1G的文件的时间

到另一台机子后，以同样的方法，恢复。假设光盘无坏道且是空的。
dd if=<mbr.bak的路径> of=/dev/hda count=512 bs=1 #跟之前的硬盘的分区一样
dd if=<sysimage.bak的路径> of=/dev/hda1 #恢复系统
```

## 3. 使用rsync命令

rsync是一个非常优秀的文件同步工具，从它的名字可以看出，它支持远程同步。当然，在备份我的桌面系统时，只需要用到它的本地同步功能就行了。之所以选择rsync，是因为它具有如下优点：

- 在备份还原过程中，可以保存文件原有的时间、权限、软硬链接等信息；

- 首次备份时，需要复制所有文件，但是再次备份或还原系统时，只需要复制修改过的文件；

  ​

```shell
##<特别注意: 不能带*号，带*号的话不会备份隐藏文件，比如/boot/*则不会备份boot目录内的隐藏文件>；
##<注意:rsync -aAXvP /boot* /tmp 结果会在tmp中自动创建boot目录即/tmp/boot并会把整个/boot目录及基内的隐藏文件都备份下来，但如果/boot2目录则也会同样创建/tmp/boot2并把/boot2整个含隐藏文件全部备份>；
##<注意:rsync -aAXvP /boot /tmp 结果是在tmp中自动创建boot目录即/tmp/boot，并会把整个/boot含隐藏文件都备份下来>；
##<注意:rsync -aAXvP /boot/ /tmp 结果是将/boot目录含隐藏文件全部保存在tmp目录中但不会创建boot目录即/tmp/boot目录>；
##note: `date +%Y%m%d-%H:%M`年月日时分,或`date +%Y-%m-%d`等等，带M时要注意比创建目录是15分但变量赋值是16分就会报错因为没有这个目录。

远程备份：
# rsync -aAXvP  -e 'ssh -p 端口' /data/ftp/ root@192.168.30.133:/data/ftp     #把本地根/目录的备份到远程；
# rsync -aAXvP /data/ftp/ root@192.168.30.133:/ftp/trade/report
#rsync -aAXvP / <user>@<host>:<backup_dir>/ --exclude={/proc,/sys,/tmp,/mnt,/media,/lost+found} 
#rsync -aAXvP /home/backup_20170219/ root@192.168.189.136:/home/backup/ --exclude={/dev,/proc,/sys,/tmp,/mnt,/media,/run,/lost+found}  把本地根/目录的备份到远程；
#rsync -aAXvP --exclude={"/dev","/proc","/sys","/tmp","/run","/mnt","/media","/lost+found"} root@xx.xx.xx:/ /  #把远程备份到本地根/目录；

备份脚本：
#!/bin/bash
#Full backup for Linux operating system.
cd /
mkdir -p /home/backup/"backup_`date +%Y%m%d_%H`"
fullbakpath=/home/backup/"backup_`date +%Y%m%d_%H`"
/usr/bin/rsync -aAXvP --exclude={"/proc","/sys","/dev","/run","/tmp","/mnt","/media","/lost+found","/home/backup"} / $fullbakpath &>"$fullbakpath/sys_full_bak.log"

增备脚本：
#!/bin/bash
#Incremental backup for Linux operating system.
cd /
increpath=/home/backup/backup_20170219
/usr/bin/rsync -aAXvP --exclude={"/proc","/sys","/dev","/run","/tmp","/mnt","/media","/lost+found","/home/backup"} / $increpath &>"$increpath/incre_`date +%Y%m%d_%H`.log"

还原脚本：
#a. 还原可以在系统登陆运行状态下直接还原，也可以用Live cd制作的U盘启动进去还原，
#b. 还原之前最好把原有目录内的文件删除，因为如果将被还原系统的目录有不同名的文件不会被覆盖，同名的会被覆盖重写；
#c. 还原用rsync或者用tar还原。

#!/bin/bash
#Full restoring for Linux operating system.
cd /
respath=$fullbakpath/   #注意这个要加个/，因为是要将$fullbakpath备份目录里面所有的，不包括$fullbakpath本身；
/usr/bin/rsync -aAXvP --exclude={"/proc","/sys","/dev","/run","/tmp","/mnt","/media","/lost+found","$respath"} "$respath" / &>"$respath.log"
restorecon -Rv /
mkdir proc
mkdir lost+found
mkdir mnt
mkdir sys

还原实例：

setenforce 0
vim /etc/selinux/config  --> SELINUX=disabled
systemctl stop firewalld
systemctl disable firewalld

mv /home/backup/etc/sysconfig/network-scripts/* /tmp/     #移除备份文件的网络配置文件
mv /home/backup/etc/fstab /home/backup/etc/fstab.bak   #移除备份文件的挂载配置文件
mv /home/backup/boot/grub2/grub.cfg /home/backup/boot/grub2/grub.cfg.bak	#移除备份文件的grub配置文件
mv /home/backup/etc/rc.d/init.d/jenkins /home/backup/var/
mv /home/backup/etc/rc.d/init.d/jira1 /home/backup/var/
mv /home/backup/etc/rc.d/init.d/jira /home/backup/var/
rsync -aAXvP /home/backup/ root@192.168.189.138:/ --exclude={/dev,/proc,/sys,/tmp,/mnt,/media,/run,/lost+found,,/home/backup}

mount -a

ln -sf /lib/systemd/system/runlevel3.target /etc/systemd/system/default.target
setenforce 0
vim /etc/selinux/config  --> SELINUX=disabled
systemctl stop firewalld
systemctl disable firewalld
reboot
restorecon -Rv /
mv /var/jekins /etc/rc.d/init.d/
mv /var/jira1 /etc/rc.d/init.d/
mv /var/jira /etc/rc.d/init.d/
```

### 桌面系统备份实战

对于我的桌面系统，我选择的备份策略是使用rsync命令将硬盘上的所有文件(当然要排除/proc和/sys目录下的内容）复制到U盘上。首先使用`df -lh`命令查看一下我电脑的系统占多大空间，

```shell
[root@rhel7 ~]# df -lh
```

可以看出，根目录所在的分区占用了14G，而/boot目录所在的分区才占用100多M。其实Linux系统本来不用这么臃肿，只是因为我安装了太多的软件包，比如一整套的texlive啊什么的，才占用了这么多的空间。不过这都不是事儿，反正现在U盘也便宜，所以找个32G的USB 3.0的U盘来备份我这个系统，肯定是很轻松愉快的。

新U盘插到电脑上会被自动识别，使用不带参数的`mount`命令可以查看U盘的设备文件和挂载路径，使用`fdisk`命令了解U盘的大小和分区情况，

```shell
[root@rhel7 ~]# mount
[root@rhel7 ~]# sudo fdisk /dev/sdb		#再输入p
```

整个U盘分成一个区我没什么意见，不过我不喜欢它的vfat文件系统，万一碰到超过4G的巨型文件怎么办？so，先`umount`它，然后使用`mkfs.ext3`为它重新建立一个文件系统，

```shell
[root@rhel7 ~]# sudo umount /dev/sdb1
[root@rhel7 ~]# sudo mkfs.ext3 /dev/sdb1
```

再把U盘`mount`起来，挂载的路径为`/media/youxia/usb`，

```shell
[root@rhel7 ~]# sudo mkdir /media/youxia/usb
[root@rhel7 ~]# sudo mount /dev/sdb1 /media/youxia/usb
```

然后，可以开始备份了，我备份的命令是：

```shell
[root@rhel7 ~]# sudo rsync -aAPXv / /media/youxia/usb/backup_20141216 --exclude=/media/* --exclude=/sys/* --exclude=/proc/* --exclude=/mnt/* --exclude=/tmp/*
```

如果哪天系统再崩溃了的话，只需要使用`sudo rsync -aAPXv /media/youxia/usb/backup_20141216 /`即可恢复系统。

```shell
[root@rhel7 ~]# sudo rsync -aAPXv /media/youxia/usb/backup_20141216 /
```

### 虚拟机实战

```shell
[root@rhel7 ~]# mkdir /media/back_2017
[root@rhel7 ~]# rsync -aAXPv / /media/back_2017 --exclude=/media/* --exclude=/sys/* --exclude=/proc/* --exclude=/mnt/* --exclude=/tmp/* --exclude=/dev/sr0 --exclude=/media/cdrom/* --exclude=/media/back_2017/*
[root@rhel7 ~]# cd /media/back_2017/
[root@rhel7 back_2017]# ls
bin   core.4705  etc   home  lib64  mnt  proc  run   srv  test   test.sh  text  usr
boot  dev        etst  lib   media  opt  root  sbin  sys  test1  tets     tmp   var
[root@rhel7 back_2017]# du -sh
3.9G	

[root@rhel7 ~]# ll -a /root/test
total 8
drwxr-xr-x.  3 root root   39 Mar 11 14:57 .
dr-xr-x---. 23 root root 4096 Mar 11 15:41 ..
-rw-r--r--.  1 root root    6 Mar 11 14:57 .hh
-rw-r--r--.  1 root root    0 Mar 11 14:57 test
drwxr-xr-x.  2 root root    6 Mar 11 14:56 .test
[root@rhel7 ~]# rsync -aAPXv /root/test /root/testbak1
sending incremental file list
test/
test/.hh
test/test
test/.test/
sent 273 bytes  received 63 bytes  672.00 bytes/sec
total size is 6  speedup is 0.02
[root@rhel7 ~]# ls -a /root/testbak1
.  ..  test
[root@rhel7 ~]# ll -a /root/testbak1/test/
total 4
drwxr-xr-x. 3 root root 39 Mar 11 14:57 .
drwxr-xr-x. 3 root root 17 Mar 11 15:42 ..
-rw-r--r--. 1 root root  6 Mar 11 14:57 .hh
-rw-r--r--. 1 root root  0 Mar 11 14:57 test
drwxr-xr-x. 2 root root  6 Mar 11 14:56 .test


[root@rhel6 ~]# rsync -aXAvP Downloads/ /testt/
sending incremental file list
./
.tests
           6 100%    0.00kB/s    0:00:00 (xfer#1, to-check=2/4)
NetSarangXmanagerEnterprise5.rar
    51773892 100%    2.69MB/s    0:00:18 (xfer#2, to-check=1/4)
运维文档.zip
    32966600 100%    2.46MB/s    0:00:12 (xfer#3, to-check=0/4)

sent 84751252 bytes  received 78 bytes  2690518.41 bytes/sec
total size is 84740498  speedup is 1.00
[root@rhel6 ~]# du -sh Downloads/
81M	Downloads/
[root@rhel6 ~]# du -sh /testt
81M	/testt
```

## 4. 再生龙备份整个Linux

```shell
在网上开始搜索相关的软件，开始找到了 Partimage 和 G4L，down下来试用了一下感觉都不是很满意，操作比较复杂而且不能完全满足我的要求。后来QQ群里的一个朋友让我试试 一款叫"再生龙"的工具。下载一用非常满意，这才是我想要的~~~
  Clonezilla - 再生龍還原系統 是台湾人开发的一款开源的备份与还原系统，功能十分强大。下面是引用其官网的部分介绍。
  #可还原再生多种作业系统，包含Linux (ext2, ext3, ext4, reiserfs, reiser4, xfs, jfs), Mac OS (HFS+), 微软Windows (fat, ntfs), FreeBSD, NetBSD, OpenBSD (UFS)，以及VMware ESX (VMFS)。这些档案系统只备份有存资料的硬碟空间，因此可以节省备份时间与硬碟空间。其他不支援的档案系统Clonezilla采用全部复制(dd) 的方式处理。
  #支援GNU/Linux下的LVM2 (尚未支援LVM1)
  #支援群播(Multicast)。配合PXE网路开机，搭配DRBL的Clonezilla可以使用播(multicast)的方式，适合用来大备份与还原。硬体设备功能足够时(用户端支援Wake on LAN与PXE)，可以远端操作，人不需到现场
而且他支持多国语言，界面友好，操作简单。这些我都很喜欢。
http://clonezilla.nchc.org.tw/这是他的官网，如果想多了解一些可以去看看，
下面我简单说下怎么用他来克隆linux系统
我下载了Clonezilla live 的ISO，地址是:http://cdnetworks-kr-2.dl.sourceforge.net/project/clonezilla/clonezilla_live_stable/clonezilla-live-1.2.4-28-686.iso
一共一百多兆。因为没有空白CD 我就直接找了个U盘 用UltraISO试用版 把ISO写到了U盘上，然后直接启动。
下面是他的界面。

我选的是第一个选项,选择后进入语言选择界面，很丰富，呵呵我选择的繁体中文

然后是键盘分布，我选择了不改变，下面就进入主题了，进入选择模式界面
选择使用再生龙，或者是进入shell，我选使用，进入备份还是还原

我要克隆整个硬盘，所以选择了下面那项，下面问你是专家还是初学者
我建议大家选择专家模式，因为我第一次用初学模式时候出现了mount 错误，说无法对挂载中的分区进行操作，估计是初学者模式进行了一些mount操作。
丰富的进阶选项
我选择的是第一项，本地到本地，下面是选择硬盘
选择母盘，千万别选错了。
再选择目标盘
各种选项，根据需要自己选择
继续下一步，
如果下次操作相同的话可以直接用上面图里的命令来执行。
 
各种警告，就是怕你哪部做错了，呵呵，下面就开始复制了
复制结束后问你还要做什么
 
好了，到这里为止一块儿硬盘就复制完成，把目标盘安装到那台机器上开机，系统自检，进入shell，打开gnome，运行程序OK，一切正常。
复制一块儿已经使用30G空间的500G硬盘用了大概10分钟左右，速度还可以。
因为实际操作中无法截图，上面面这些截图都在虚拟机上完成，不过步骤是一样的
 
上面这些只不过是“再生龙”的一小部分功能，如果你想了解更多，可以去他们官网或者自己研究。我这里就说这么多了，希望这篇文章能对大家有所帮助O(∩_∩)O~

本文出自 “story的天空” 博客，请务必保留此出处http://storysky.blog.51cto.com/628458/291587
```

