# 华通工作总计

# 1. 办公信息

```shell
# 上海华通铂银交易市场有限公司，上海金孛信息科技有限公司 公司公积金账号886344301205
# 周报系统 http://www.ht8.com/
		username: 王跃晖	passwd: ht-83...9
		1.访问地址：http://www.ht8.com
		2.绑定host：192.168.30.136	www.ht8.com
# OA在线系统 pha 
http://pha.huatongsilver-inc.com/ 
barrywang@huatongsilver.com   code:ht-83...49
# 电脑code:	ht-83...49
	ip: 192.168.40.3，GATEWAY 192.168.40.1，netmask 255.255.255.0， DNS 202.96.209.5 210.22.84.3
# 电话	021-58793401-8076
# QQ	3002842038 HT!@#83794245
# 邮箱：
"王跃晖"<barrywang@huatongsilver.com>，code:Ht-83...49
# 指纹：
号码 0125
# Teamviewer 	
160 708 850		ht-83...49
# Anydesk
502 899 901     ht-83...49
# 1个交易日：18：00-02：30，8：30-15：30。比如今天是5月16日，那么18：00-02：30，8：30-15：30就是5月17日的交易日，交易日是跨自然日的。
# 夜班时间：17：30-02：30，白班时间：8：30-17：00，
#Oracle   官网 barrywang@huatongsilver.com   code: Olele055
```

# 2. 系统信息

- VMware vSphere

```shell
IP： 192.168.30.50 
账号： vsphere.local\administrator  密码： VS30.50huatong  	
```

- vmware Citrix

  http://192.168.30.192/Citrix/StoreWeb

- 交易平台环境列表

```shell
  http://pha.huatongsilver-inc.com/
  前提：
  本地电脑--》C:\Windows\System32\drivers\etc\hosts文件中追加1条记录：
  192.168.30.179 pha.huatongsilver-inc.com
  
  用户名：barrywang  code: ht-83..49
```

- 虚拟机

```shell
192.168.30.130 123456
192.168.30.210 huatong
```

- 交易架构

```shell
前置： Nginx proxy+tomcat，

核心模块：
newsserver 新闻
tradeserver交易
schedule 日程 开闭市
managerserver 后台
sms 短信
session 会话

后端DB： Oracle
```



进程启动报错

因为开发在编译的时候一个是dos编码，一个是unix编码，则在配置文件中非编辑模式中输入：

```shell
/:set ff=unix
```



仓单架构

```shell
 svn://192.168.30.87/htschedule
svn://192.168.30.87/htwhr0424
svn://192.168.30.87/htbank
ops = Htxh#12D08
sysadmin = Sys@20161208

http://192.168.30.87:8080/jenkins/
sysadmin = Jks#30D87
```

加入访问代码权限nginx，

```shell
ssh 192.168.30.179    username: host01  passwd: 123456
host01@ubuntu:~$ cd /usr/local/nginx/conf/vhost
host01@ubuntu:/usr/local/nginx/conf/vhost$ ls
pha.conf
host01@ubuntu:/usr/local/nginx/conf/vhost$ sudo vim pha.conf 
server {
       listen 80;
       server_name  pha.huatongsilver-inc.com;
       access_log  /usr/local/nginx/logs/pha_access.log;
       error_log   /usr/local/nginx/logs/pha_error.log;

       location / {
                proxy_pass http://127.0.0.1:61280;
                   }

      location = /diffusion/ {
             proxy_pass http://127.0.0.1:61280;
             }

       location ~* /diffusion/.*git.* {
               proxy_pass http://127.0.0.1:61280;
               allow  192.168.30.176;
               allow  192.168.30.178;
               allow  192.168.30.180;
               allow  192.168.30.181;
               allow  192.168.30.182;
               allow  192.168.30.183;
               allow  192.168.30.184;
               allow  192.168.30.185;
               allow  192.168.30.186;
               allow  192.168.30.136;
               deny all; 
                }
   }
host01@ubuntu:~$ cd /usr/local/nginx/sbin
host01@ubuntu:/usr/local/nginx/sbin$ ls
nginx
host01@ubuntu:/usr/local/nginx/sbin$ sudo ./nginx -s reload
```

- 华通交易客户端登录配置文件：service..xml内容梳理

```shell

Updated Today所有用户
操作
<?xml version="1.0" encoding="utf-8"?>
<holysky>

<Services>
  <Service id="0" name="模拟盘" select="1" runmode="1"   tradehttpservlet="tradeweb/tradeHttpServlet" bankservlet="tradeweb/bank/index?hs_sid_t=" websocketservlet="tradeweb/tradeWebSocketServlet?hs_sid_t=" helpservlet="help/traderclient/index.html"  >
    <Links>
      <Link id="1" name="测试环境电信" url="180.168.69.245" port="80" safeport="443" select="1" />
      <Link id="2" name="测试环境内网" url="192.168.30.2" port="80" safeport="443" select="0" />
      <Link id="3" name="科泽模拟云盾电信" url="183.131.180.69" port="80" safeport="443" select="0" />
      <Link id="4" name="科泽模拟云盾联通" url="101.71.32.100" port="80" safeport="443" select="0" />
      <Link id="5" name="科泽模拟高防IP" url="122.144.208.25" port="80" safeport="443" select="0" />
    </Links>
  </Service>
<Service id="1" name="实盘" select="0" runmode="1" tradehttpservlet="tradeweb/tradeHttpServlet" websocketservlet="tradeweb/tradeWebSocketServlet?hs_sid_t=" helpservlet="help/traderclient/index.html">

  <Links>
    <Link id="1" name="科泽实盘域名" url="trade.huatongsilver.com" port="80" safeport="443" select="1" />
    <Link id="2" name="科泽实盘高防IP" url="122.144.208.20" port="8080" safeport="8443" select="0" />
    <Link id="3" name="有孚实盘电信" url="175.102.9.180" port="7777" safeport="7443" select="0" />
    <Link id="4" name="有孚实盘测试" url="175.102.9.177" port="80" safeport="443" select="0" />
  </Links>
</Service>
</Services>
</holysky>
```

# 3. 以前信息

```shell
入职事宜
2016年3月1日周三下午13：00入职，

电脑电话配置
分机：8035，
DELL，win7 64bits，
Ip 192.168.8.8，gw 192.168.8.1，dns 202.96.209.5, 210.22.84.3，

机房
192.168.70.70，user: superuser，code:?，存储管理 IBM Storwize V5000，
机房温度：15~28度

交易时间：8：30-15：30，18：00-02：30


工作
Xshell 5，Xftp 5；

Phabricator
Phabricator是一个集成了项目管理、代码库、代码评审（Pre-Commit Code Review）与审计（After-Commit Code Audit）、Bug管理以及持续集成的工具平台。该工具可以有效覆盖产品文档编写、设计稿、开发以及测试流程。

yum groupinstall "Desktop"
yum groupinstall "X Window System"
yum groupinstall "Chinese Support"  （这一步可能有问题，可以忽略）
yum groupinstall "Font"
startx
yum provides 

yum install tigervnc

window安装vnc-viewer
Windows界面打开VNC Viewer，输入192.168.30.112:1

登录到Pha平台：
控制面板\网络和 Internet\网络和共享中心以太网连接状态常规属性IPV4ip 192.168.70.17，netmask 255.255.255.0确定；
Win+Rcmd ping 192.168.70.70能ping通，打开浏览器输入192.168.70.70Storwize V5000输入用户名 superuser，code:? 
记事本打开C:\Windows\System32\drivers\etc\hosts文件，追加192.168.30.179  pha.huatongsilver-inc.com，浏览器打开http://pha.huatongsilver-inc.com登录

交易中心管理系统（模拟盘）
http://192.168.30.2/adminweb,  
username: wangyuehui, code:111…..

登录oracle
[oracle@db ~]$ sqlplus /nolog		
SQL*Plus: Release 11.2.0.1.0 Production on Thu Mar 2 16:42:16 2017
Copyright (c) 1982, 2009, Oracle.  All rights reserved.
SQL> conn /as sysdba		#note: 以管理员身份登录！
Connected.

[oracle@db ~]$ rlwrap sqlplus / as sysdba
SQL> drop user trade cascade;
User dropped.
[root@db ~]# cat >> /home/oracle/.bash_profile << end
> alias sqlplus='rlwrap sqlplus'
> alias rman='rlwrap rman'
> end
[root@db ~]# source /home/oracle/.bash_profile
[root@db ~]# alias sqlplus='rlwrap sqlplus'
[root@db ~]# alias rman='rlwrap rman'

开启远程桌面：
开始菜单搜索程序和文件框中输入mstsc输入IP；或先win+R 然后输入mstsc 输入用户名+密码。

工行测试环境：远程桌面连接，IP为192.168.20.232


Win桌面安装EditPlus方便看Linux下面的文件。

Zookeeper监听端口2181，基于java。

[root@db yum.repos.d]# cat inward.repo 
[base]
name=ustc centos 6.8 os
baseurl=ftp://192.168.30.64/pub/CentOS/ustc/6.8/os/x86_64
gpgcheck=1
gpgkey=ftp://192.168.30.64/pub/CentOS/ustc/RPM-GPG-KEY-CentOS-6
enabled=1

[update]
name=ustc centos 6.8 update
baseurl=ftp://192.168.30.64/pub/CentOS/ustc/6.8/updates/x86_64/
gpgcheck=1
gpgkey=ftp://192.168.30.64/pub/CentOS/ustc/RPM-GPG-KEY-CentOS-6
enabled=1

[extra]
name=ustc centos 6.8 extra
baseurl=ftp://192.168.30.64/pub/CentOS/ustc/6.8/extras/x86_64/
gpgcheck=1
gpgkey=ftp://192.168.30.64/pub/CentOS/ustc/RPM-GPG-KEY-CentOS-6
enabled=1

查看程序是否安装：
[root@web01 ~]# which java
/usr/local/java/bin/java
查看程序的位置：
[root@web01 ~]# whereis java
java: /etc/java /usr/lib/java /usr/local/java /usr/share/java
```

