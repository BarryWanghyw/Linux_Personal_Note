# Linux之Sed进阶

# 一、写在前面

- 一定学着用 / / ，/ / 内是要匹配的字段，sed -i 's///g 替换，sed -n '//p', sed '//d'都会用到，

 如sed '/2/d' 指匹配含有数字2的行 然后删除此行； 

```shell
字符 / / / 在sed中作为定界符使用，也可以使用任意的定界符
 sed 's:test:TEXT:g' 
 sed 's|test|TEXT|g' 
```

- sed所有的命令操作不会修改原文件，只有当加上 -i 时 会直接对原文件进行修改；
- sed 的参数要加单引号''，如sed -n '1p' file1。

# 二、命令简介

```shell
sed是stream editor的缩写，称之为 流编辑器， 是一个很好的文件处理工具。

常用选项：

        -n∶ 使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN的资料一般都会被列出到萤幕上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。

        -e∶ 直接在指令列模式上进行 sed 的动作编辑；

        -f∶ 直接将 sed 的动作写在一个档案内， -f filename 则可以执行 filename 内的sed 动作；

        -r∶ sed 的动作支援的是延伸型正规表示法的语法。(预设是基础正规表示法语法)

        -i∶ 直接修改读取的档案内容，直接对文件进行增删减等操作，也不输出到屏幕。       

常用命令：

        a   ∶ 新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～

        c   ∶ 取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！

        d   ∶ 删除，因为是删除啊，所以 d 后面通常不接任何咚咚；

         i   ∶ 插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；

         p  ∶ 列印，亦即将某个选择的资料印出。通常 p 会与参数 sed -n 一起运作～

         s  ∶ 取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！
```

# 三、具体用法

## 1. sed -i, s, g 替换字段

- s 替换特定字段，不加 -i 不修改原文件，加上 -i 直接对原文件进行操作：

sed -i 表示直接操作原文件，直接对原文件进行修改，

sed  's/原字段/新字段/'  文件，表示替换文件的每一行的第1个原字段，s表示替换，

sed  's/原字段/新字段/g'  文件，表示替换文件的所有的原字段，g表示匹配文件中所有指定字段并进行操作，

sed -i "s%^DocumentRoot.*%DocumentRoot /var/www.abc%" /etc/httpd/conf/httpd.conf

​	#note:上面使用百分号%是因为新字段中有/则偶尔会有异常，则可使用sed  's%原字段%新字段'  文件，

sed -i "s/listen-on port 53 { 127.0.0.1; }/listen-on port 53 { any; }/" /etc/named.conf

​	#note:sed中单引号''和双引号""的没有区别，故可统一采用单引号''。

```shell
[root@rhel6 ~]# cat test1
	~hellohello test hello
	wang @hello@hello*
	hello	/hello(hello)
[root@rhel6 ~]# sed -i 's/hello/hi/' test1		#note:替换每一行的第一个匹配到的字段。
[root@rhel6 ~]# cat test1
	~hihello test hello
	wang @hi@hello*
	hi	/hello(hello)
[root@rhel6 ~]# sed -i 's/hello/hi/g' test1		#note:替换每一行的所有匹配到的字段。
[root@rhel6 ~]# cat test1
	~hihi test hi
	wang @hi@hi*
	hi	/hi(hi)
	
[trade@trade01 tmp]$ sed 's/coffee/apple/g' test	#所有含有coffee的字段替换为apple，不会修改原文件。
bye
drink tea
or apple
```

```shell
find . -name appConfig.properties | xargs sed -i 's/strade01:61616/strade02:61616/g'
find . -name appConfig.properties | xargs sed -i 's/^env.idcard/\#env.idcard/g'

[trade@web02 ~]$ cat test.txt
1
2
3
4
5
6
[trade@web02 ~]$ sed -i 's/$/,/' test.txt | sed -i 's/^/,/' test.txt 
[trade@web02 ~]$ cat test.txt 
,1,
,2,
,3,
,4,
,5,
,6,
```

当需要从第N处匹配开始替换时，可以使用 /Ng：

```shell
 echo sksksksksksk | sed 's/sk/SK/2g' 
 skSKSKSKSKSK
 echo sksksksksksk | sed 's/sk/SK/3g'
 skskSKSKSKSK  
 echo sksksksksksk | sed 's/sk/SK/4g'
 skskskSKSKSK 
```

**选定行的范围：,（逗号）**

所有在模板test和check所确定的范围内的行都被打印：

```
 sed -n '/test/,/check/p' file
```

打印从第5行开始到第一个包含以test开始的行之间的所有行：

```
 sed -n '5,/^test/p' file
```

对于模板test和west之间的行，每行的末尾用字符串aaa bbb替换：

```
 sed '/test/,/west/s/$/aaa bbb/' file
```

如果要匹配的目标 字符串中包含元字符，需要使用转义符“ \ ”屏蔽其特殊意义，如下给出匹配句点“ . ”元字符和“ $”元字符及“  / ”元字符的命令，

```shell
[root@rhel6 ~]# cat test
wang @hi.com
$test$.cn
[root@rhel6 ~]# sed -i 's/\./5\$i/g' test	#note:把所有的.替换为5$i。
[root@rhel6 ~]# cat test
wang @hi5$icom
$test$5$icn
[root@rhel6 ~]# cat test2
/hello@/world
//hehe/end
[root@rhel6 ~]# sed -i 's/\//\/\\/g' test2 #note:把所有的/替换为/\。
[root@rhel6 ~]# cat test2
/\hello@/\world
/\/\hehe/\end
```

## 2. sed -n, p 打印整行

sed -n表示全部不打印出来，p表示打印，p与sed -n选项一般配合使用表示只打印进行操作的行，

sed -n '1,3p' file表示只打印第1~3行，sed -n '1,$p' file表示打印第1行到最后一行，

sed -n '13p' file表示打印第13行，

```shell
　显示某行
 [root@localhost ruby] # sed -n '1p' ab          #显示第一行 
 [root@localhost ruby] # sed -n '$p' ab          #显示最后一行
 [root@localhost ruby] # sed -n '1,2p' ab        #显示第一行到第二行
 [root@localhost ruby] # sed -n '2,$p' ab        #显示第二行到最后一行
[root@rhel6 ~]# sed -n '1,3p' test1		#note:只打印第1~3行.
[root@rhel6 ~]# sed -n '13p' test1		#note:只打印第13行而第13行没有故不打印.
[root@rhel6 ~]# sed -n '1,$p' test1		#note:打印第1行到最后一行。
```

### sed / /p 打印匹配行

sed要匹配字段和关键字等字符串都不但要用引号''还要用/ /符号，如上面讲的sed -i 's/ / /'替换操作，sed -i '/^$/d'删除操作，sed -n '/hello/p'打印包含hello的行，这个与grep只用引号''不同，

     [root@localhost ruby] # sed -n '/ruby/p' ab     #查询包括关键字ruby所在所有行
     [root@localhost ruby] # sed -n '/\$/p' ab       #查询包括关键字$所在所有行，使用反斜线\屏蔽特殊含义
     [trade@trade01 tmp]$ sed -n '/end/p' test   #匹配含有end的行，打印出来。
```shell
[root@rhel6 ~]# cat test2
/\hello@/\world.hi
^hello
/\/\hehe/\end
[root@rhel6 ~]# sed -n '/\./p' test2		#note:打印包含.的行。
/\hello@/\world.hi
[root@rhel6 ~]# sed -n '/hello/p' test2		#note:打印包含hello的行。
[root@rhel6 ~]# sed -n '/hello/,$p' test2	#note:打印第1个匹配hello的行到最后一行。
/\hello@/\world.hi
^hello
/\/\hehe/\end
[root@rhel6 ~]# sed -n '/hello/,2p' test2	#note:打印第1个匹配hello的行到第2行。
/\hello@/\world.hi
^hello
[root@rhel6 ~]# sed -n '2,/end/p' test2		#note:打印第2行到第1个匹配end的行。
^hello
/\/\hehe/\end
```



## 3. sed d 删除整行

d表示删除整行注意是整行，但同时也不是真正的对文件中的行进行删除：sed是面向行进行处理，每一次处理一行内容；处理时，sed会把要处理的行存储在缓冲区中，接着用sed命令处理缓冲区的内容，处理完成后，把缓冲区的内容送往屏幕；接着处理下一行，这样不断重复，直到文件末尾，所以此时sed命令并不会对原文件本身进行任何贸然更改，而只是对缓冲的文件副本行进行操作。除非用sed -i  -e命令等。

- d 删除整行，不加 -i 不修改原文件，加上 -i 直接对原文件进行操作：


- 下面的操作是对文件的缓冲副本执行d删除操作以后，把结束输出到屏幕，**不修改原文件，**

```shell
删除某行 后 把删除后的内容打印到屏幕，不修改原文件：
 
     [root@localhost ruby] # sed '1d' ab              #删除第一行 
     [root@localhost ruby] # sed '$d' ab              #删除最后一行
     [root@localhost ruby] # sed '1,2d' ab           #删除第一行到第二行
     [root@localhost ruby] # sed '2,$d' ab           #删除第二行到最后一行
     [root@rhel6 ~]# sed '/^$/d' test2               #sed '/^$/d' 删除所有空白行，把删除空行后的内容打印到屏幕，但test2文件本身没有变化即不会对test2本身进行操作，^$表示以$开头，而$代表以什么结尾或者行尾符，行为空时，自然一行就以$开始，那么删除以行尾符$开头的行就是把空行给删了。 #一句话，在正则表达式中，^$表示空行
     
     [root@rhel6 ~]# sed '/he/d' test    #删除所有含有he字段的行。
```

- 下面的操作**直接对原文件进行修改**，不打印结果到屏幕，

```shell
删除某行，修改原文件，不打印出来：
[root@rhel6 ~]# sed -i '/^$/d' test2   #直接对原文件进行操作，删除所有的空行，而不仅仅是删除最后的空行。

[root@rhel6 ~]# sed -i '$d' test2	   #直接对原文件进行操作，删除 最后 一行。

[root@rhel6 ~]# sed -i '/he/d' test    #删除所有含有he字段的行。
```

删除文件的第2行：

```
 sed '2d' file
```

删除文件的第2行到末尾所有行：

```
 sed '2,$d' file
```

删除文件最后一行：

```
 sed '$d' file
```

删除文件中所有开头是test的行：

```
 sed '/^test/'d file
```

## 4. sed a 原有行的下一行增加一行

- a 增加一行或多行字符串, 不加 -i 不修改原文件，加上 -i 直接对原文件进行操作：

```shell
[root@localhost ruby]# cat test
     Hello!
     ruby is me,welcome to my blog.
     end
[root@localhost ruby] # sed '1a drink tea' test  #第一行后增加字符串"drink tea",a和drink间的空格不加也行。
[root@localhost ruby] # sed '1,3a drink tea' test #第一行到第三行后增加字符串"drink tea"

[root@localhost ruby] # sed '1a drink tea\nor coffee' test   #第一行后增加多行，使用换行符\n,这里在第1行的下一行增加了drink tea和下下一行增加了or coffee，
     Hello!
     drink tea
     or coffee
     ruby is me,welcome to my blog.
     end
     
[root@localhost ruby] # sed '1,3a drink tea\nor coffee' test   #在第1行的下一行增加了drink tea和下下一行增加了or coffee，在第2行的下一行增加了drink tea和下下一行增加了or coffee，在第3行的下一行增加了drink tea和下下一行增加了or coffee。

[root@localhost ruby] # sed '/ee/a drink tea' test	   #在所有含有ee字段的行的下一行增加一行drink tea.

[root@localhost ruby] # sed -i '/ee/a drink tea' test  #不打印到屏幕，直接对原文件进行操作。
```



## 5. sed i 原有行的上一行增加一行

- i 增加一行或多行字符串, 不加 -i 不修改原文件，加上 -i 直接对原文件进行操作：

```shell
[trade@trade01 tmp]$ cat test
bye
drink tea
or coffee
[trade@trade01 tmp]$ sed '$i hello' test #在最后一行的上一行增加一行hello。i和hello之间的空格有无都一样。
bye
drink tea
hello
or coffee
[trade@trade01 tmp]$ sed '$ihello\nhi' test #在最后一行的上一行 增加一行hello和增加一行hi。
bye
drink tea
hello
hi
or coffee
[trade@trade01 tmp]$ sed '/ee/ismile' test #在所有含有ee字段的行的上一行增加一行smile.
bye
drink tea
smile
or coffee
[trade@trade01 tmp]$ sed -i '/ee/ismile' test   #不打印到屏幕，直接对原文件进行操作。
[trade@trade01 tmp]$ cat test 
bye
drink tea
smile
or coffee
```



## 6. sed c 替换整行

- c 替换整行, 不加 -i 不修改原文件，加上 -i 直接对原文件进行操作：

```shell
[trade@trade01 tmp]$ cat test
bye
drink tea
or coffee
[trade@trade01 tmp]$ sed '1c hello' test #第一行替换为hello。
hello
drink tea
or coffee
[trade@trade01 tmp]$ sed -i '1c hello' test #不打印到屏幕，直接对原文件进行操作。
hello
drink tea
or coffee
[trade@trade01 tmp]$ cat test
hello
drink tea
or coffee
```



## 7. 元字符集

```shell
 ^ 匹配行开始，如：/^sed/匹配所有以sed开头的行;
 
 $ 匹配行结束，如：/sed$/匹配所有以sed结尾的行;
 
 . 匹配一个非换行符的任意字符，如：/s.d/匹配s后接一个任意字符，最后是d;
 
 * 匹配0个或多个字符，如：/*sed/匹配所有模板是一个或多个空格后紧跟sed的行;
  
 [] 匹配一个指定范围内的字符，如/[ss]ed/匹配sed和Sed;
   
 [^] 匹配一个不在指定范围内的字符，如：/[^A-RT-Z]ed/匹配不包含A-R和T-Z的一个字母开头，紧跟ed的行;
  
 \(..\) 匹配子串，保存匹配的字符，如s/\(love\)able/\1rs，loveable被替换成lovers;
  
 & 保存搜索字符用来替换其他字符，如s/love/**&**/，love这成**love**;
  
 \< 匹配单词的开始，如:/\<love 匹配包含以love开头的单词的行;\> 匹配单词的结束，如/love\>/匹配包含以love结尾的单词的行;
 
 x\{m\} 重复字符x，m次，如：/0\{5\}/匹配包含5个0的行;
 
 x\{m,\} 重复字符x，至少m次，如：/0\{5,\}/匹配至少有5个0的行;
 
 x\{m,n\} 重复字符x，至少m次，不多于n次，如：/0\{5,10\}/匹配5~10个0的行;
```



## 8 sed 替换标记

```shell
 g 表示行内全面替换;

 p 表示打印行;

 w 表示把行写入一个文件;

 x 表示互换模板块中的文本和缓冲区中的文本;

 y 表示把一个字符翻译为另外的字符（但是不用于正则表达式）;
 
 \1 子串匹配标记;

 & 已匹配字符串标记;
```



**已匹配字符串标记&**

正则表达式 \w\+ 匹配每一个单词，使用 [&] 替换它，& 对应于之前所匹配到的单词：

```
 echo this is a test line | sed 's/\w\+/[&]/g'
 [this] [is] [a] [test] [line] 
```

所有以192.168.0.1开头的行都会被替换成它自已加localhost：

```
 sed 's/^192.168.0.1/&localhost/' file 192.168.0.1localhost
```

**子串匹配标记\1**

匹配给定样式的其中一部分：

```
 echo this is digit 7 in a number | sed 's/digit \([0-9]\)/\1/' 
 this is 7 in a number
```

命令中 digit 7，被替换成了 7。样式匹配到的子串是 7，\(..\) 用于匹配子串，对于匹配到的第一个子串就标记为 \1，依此类推匹配到的第二个结果就是 \2，例如：

```
 echo aaa BBB | sed 's/\([a-z]\+\) \([A-Z]\+\)/\2 \1/' 
 BBB aaa
```

love被标记为1，所有loveable会被替换成lovers，并打印出来：

```
 sed -n 's/\(love\)able/\1rs/p' file
```

## 9. 变形y

**变形：y命令**

把1~10行内所有abcde转变为大写，注意，正则表达式元字符不能使用这个命令：

```
 sed '1,10y/abcde/ABCDE/' file
```

## 10. 打印匹配字符串的下一行

```shell
 grep -A 1 SCC URFILE 
 sed -n '/SCC/{n;p}' URFILE 
 awk '/SCC/{getline; print}' URFILE
```

