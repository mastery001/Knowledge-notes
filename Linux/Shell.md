1.shell语法

1.1.第一行的#!

​     shell脚本一般开始的第一行就会有 #!/bin/bash 或者 #!/bin/sh 又或者是其他的路径，那么这第一行的这些东西到底代表什么？

​     

​     首先，当shell执行一个程序时，会要求unix内核启动一个新的进程，但是shell并非是编译型程序，内核则会认为这是一个shell脚本，就会启动一个新的/bin/sh副本来执行该程序。但是现在linux会有好多shell，当然也会有自己自定义的shell，所以这一行的#!后面的内容就是告诉内核用哪个shell来执行这个脚本。此外，这里还可以传递一个选项给解释器，例如第一行为：#!/bin/awk -f ，那么这个shell就只能解析awk这个命令的程序。

1. 2 注释

一行的开头#即为注释

1. 3 变量

shell的变量是动态赋值的，不需要事先定义，不存在全局和局部的变量定义

如：variable_name="1" 

引用变量方式： （1）$variable_name   （2）${variable_name  }

假设当前目录下有一个1.txt文件，可动态输出，shell中内容为： 

\#!/bin/bash

filename="1"

cat $filename.txt

这样既可输出1.txt中的内容。

​     连接shell变量是需要使用双引号的，如下：

​      first=Hello   middle=Shell   last=World

​      fullname="$first $middle $last"

shell是可以动态删除变量的：例如

a=1     #定义a变量

unset a      #删除a变量

下面附上unset的指令的参数说明

| 参数                 | 功能                        |
| -------------------- | --------------------------- |
| -v                   | 仅仅删除变量                |
| -f                   | 仅仅删除函数                |
| 【变量或者函数名称】 | 将要删除的shell变量或者函数 |

Important

在shell中时可以执行指令的，但是如果要将运行的结果赋值到变量需要将指令放在$()中，如下：

files=$(ls /data)          #这条语句会列出/data目录下所有的文件然后赋值给files变量

Notes：当在shell脚本中执行内置命令时引用自定义的shell变量时的引用方式为"'$variable_name'"；例如

curl -H "Content-Type:application/json;charset=utf-8" -X POST --data '{"usr":"lsw-040201-mzjl-01","pwd":"69E99149C3E4E3BE","ext":"10022","to":"18910502026,18645089436,15210057607","msg":"'$msg'"}' "http://ms.go.le.com/service/message"

1. 4.数组

声明数组：array_name=(value0 value1 value2 value3 ...) 也可声明空数组 array_name=()

遍历数组：

【使用@ 或 * 可以获取数组中的所有元素】

获得数组长度：length=${#array_name[@]}

取得数组单个元素长度：lengthn=${#array_name[n]}

（1）使用for..in

for item in ${array_name[@]}

do

​     echo ${item}

done

(2)根据数组元素个数遍历

length=${#array_name[@]}

for i in $(seq 0 $length)

do

​     echo ${array_name[i]}

done

1. 5.控制流：if/then/elif/else/fi

Important:注意"["和"]"前后的空格是必须有的，否则会提示错误

格式：

| if [ condition1 ]; then    executefi | if [ condition ]; then    execute1else     execute2fi | if [ condition1 ]; then     execute1elif [ condition2 ]; then      execute2else     execute3fi |
| ------------------------------------ | ----------------------------------------------------- | ------------------------------------------------------------ |
|                                      |                                                       |                                                              |

1. 6.for/do/done/while

实例：

| 循环   | 格式                                                         | 实例                                                         |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| for    | for var in word1 word2 ... wordNdo   Statement(s) to be executed for every word.done | #!/bin/bashfor i in 1 2 3 4 5 6do     echo $idone            |
| while  | while commanddo   Statement(s) to be executed if command is truedone | #!/bin/basha=0while [ $a -lt 10 ]do   echo $a   a=`expr $a + 1`done |
| select | select var in word1 word2 ... wordNdo   Statement(s) to be executed for every word.done | #!/bin/bashselect DRINK in tea cofee water juice appe all nonedo   case $DRINK in      tea\|cofee\|water\|all)         echo "Go to canteen"         ;;      juice\|appe)         echo "Available at home"      ;;      none)         break      ;;      *) echo "ERROR: Invalid selection"      ;;   esacdone |

1.7.格式化输出日期

shell通过date来获取日期

如：  date +%Y%m%d%H%M%S     #会输出当前日期和时间连成的字符串，假设当前时间为2016-01-14 15:01:31 那么会输出20160114150131

1. 8.exist

退出当前shell脚本，一般来说，返回0表示执行成功，其他值表示没有执行成功。

exist 0    # 返回0

exist 1    # 返回1

1. 9.系统变量和特殊变量

系统变量：

| 变量  | 作用                                                         |
| ----- | ------------------------------------------------------------ |
| $PWD  | 代表当前目录                                                 |
| $USER | 当前用户                                                     |
| $0    | 这个程式的执行名字                                           |
| $n    | 这个程式的第n个参数值，n=1..9                                |
| $*    | 这个程式的所有参数,此选项参数可超过9个。表示所有这些参数都被双引号引住 |
| $@    | 表示所有这些参数都分别被双引号引住                           |
| $#    | 这个程式的参数个数举例说：脚本名称叫test.sh 入参三个: 1 2 3运行test.sh 1 2 3后$*为"1 2 3"（一起被引号包住）$@为"1" "2" "3"（分别被包住）$#为3（参数数量） |
| $$    | 这个程式的PID(脚本运行的当前进程ID号)                        |
| $!    | 执行上一个背景指令的PID(后台运行的最后一个进程的进程ID号)    |
| $?    | 执行上一个指令的返回值 (显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误) |

特殊变量  【友情链接：<http://blog.itpub.net/10522540/viewspace-212846/>】

| 变量                           | 作用                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| ~  帐户的 home 目录            | 算是个常见的符号，代表使用者的 home 目录：cd ~；也可以直接在符号后加上某帐户的名称：cd ~user或者当成是路径的一部份：~/bin；~+ 当前的工作目录，这个符号代表当前的工作目录，她和内建指令 pwd 的作用是相同的。# echo ~+/var/log~- 上次的工作目录，这个符号代表上次的工作目录。# echo ~-/etc/httpd/logs |
| ; 分号 (Command separator)     | 在 shell 中，担任"连续指令"功能的符号就是"分号"。譬如以下的例子：cd ~/backup ; mkdir startup ; cp ~/.* startup/. |
| ;; 连续分号 (Terminator)       | 专用在 case 的选项，担任 Terminator 的角色。case "$fop" inhelp) echo "Usage: Command -help -version filename" ;;version) echo "version 0.1" ;;esac |
| . 逗号 (dot)                   | 在 shell 中，使用者应该都清楚，一个 dot 代表当前目录，两个 dot 代表上层目录。CDPATH=.:~:/home:/home/web:/var:/usr/local在上行 CDPATH 的设定中，等号后的 dot 代表的就是当前目录的意思。如果档案名称以 dot 开头，该档案就属特殊档案，用 ls 指令必须加上 -a 选项才会显示。除此之外，在 regular expression 中，一个 dot 代表匹配一个字元。 |
| 'string' 单引号 (single quote) | 被单引号用括住的内容，将被视为单一字串。在引号内的代表变数的$符号，没有作用，也就是说，他被视为一般符号处理，防止任何变量替换。heyyou=homeecho '$heyyou' # We get $heyyou |
| "string" 双引号 (double quote) | 被双引号用括住的内容，将被视为单一字串。它防止通配符扩展，但允许变量扩展。这点与单引数的处理方式不同。heyyou=homeecho "$heyyou" # We get home`command` 倒引号 (backticks)在前面的单双引号，括住的是字串，但如果该字串是一列命令列，会怎样？答案是不会执行。要处理这种情况，我们得用倒单引号来做。fdv=`date +%F`echo "Today $fdv"在倒引号内的 date +%F 会被视为指令，执行的结果会带入 fdv 变数中。 |
| \| 管道 (pipeline)             | pipeline 是 UNIX 系统，基础且重要的观念。连结上个指令的标准输出，做为下个指令的标准输入。who \| wc -l善用这个观念，对精简 script. 有相当的帮助。 |
| & 后台工作                     | 单一个& 符号，且放在完整指令列的最后端，即表示将该指令列放入后台中工作。tar cvfz data.tar.gz data > /dev/null &\<...\> 单字边界这组符号在规则表达式中，被定义为"边界"的意思。譬如，当我们想找寻 the 这个单字时，如果我们用grep the FileA你将会发现，像 there 这类的单字，也会被当成是匹配的单字。因为 the 正巧是 there 的一部份。如果我们要必免这种情况，就得加上 "边界" 的符号grep '\' FileA |
|                                |                                                              |

1.10 declare

1) declare声明方式：

​    declare [-aAfFilrtux] [-p] [name[=value] ...]

​    参数说明

2.一些常用命令

（1）有关输入输出

| 命令                          | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| cat filename                  | 输出文件的全部内容                                           |
| head -n \| -c number filename | 输出对应number的行数或字节数的文件,其中-n是行数，-c是字节数友情链接：<http://www.cnblogs.com/peida/archive/2012/11/06/2756278.html> |
| less filename                 | 一次显示屏幕所能显示的内容(页)，可通过j，k或者up，down来控制其显示向上或者向下 |
| more filename                 | 一次显示屏幕所能显示的内容(页)，可通过回车来控制向下输出     |
| tail [option] filename        | 用于显示指定文件末尾内容，不指定文件时，作为输入信息进行处理。常用查看日志文件。-f 为循环读取-n 为显示行数   tail -n 5 filename 显示该文件的最后5行数据  tail -n +5 filename 从第5行开始显示友情链接：<http://www.cnblogs.com/peida/archive/2012/11/07/2758084.html> |

head -n 5 1.txt  ==  sed '5q' 1.txt      ==> 显示出文件的前五行

（2）查找和替换

2.1 sed命令

形式： sed [options] 'command' file(s) |  sed [options] -f scriptfile file(s)

详细说明请参照：<http://www.iteye.com/topic/587673>   ， 这里只做部分介绍

替换：

(1) sed 's/test/mytest/g' 1.txt   # 会将1.txt中所有的test替换成mytest，如果没有/g则只会替换每行第一个匹配的test

(2) sed -n 's/\(love\)able/\1rs/p' 1.txt  # 会将所有的loveable都替换成lovers，并且替换的行会被打印出来; [-n一般与/p搭配使用] Notes: 这里的\(love\)是一种后向引用的机制，后面可以按出现的顺序来引用\1,\2等

(3) sed 's/^192.168.0.1/&localhost/' 1.txt  # &符号表示替换字符串中被找到的部分，所以192.168.0.1开头的行都会被替换为192.168.0.1localhost

删除：

(1) sed '2d' 1.txt  #删除1.txt的第二行     

(2) sed '2,$d' 1.txt #删除第二至最后一行     

(3) sed '/test/'d 1.txt  #删除出现了test的每一行

输出：

(1) sed -n '/test/p' 1.txt  #所有包含test的行都会被打印

(2) sed -n '/test/,/my/p' 1.txt  # 所有包含test和my的行都会被打印

(3) sed -n '5,/^test/p' 1.txt  #从第5行开始到第一个包含以test开头的行都会被打印

(4) sed '5q' 1.txt  # 打印出前五行内容

2.2 awk命令

1.介绍

awk是一个强大的文本分析工具；简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。

格式： awk [-F  field-separator]  'commands'  input-file(s)

例子：  who | awk '{print $1}'   ,其中输出如下

其实这条命令的全部命令是： who | awk -F " " '{print $1}'    ; awk命令是默认以" "空格来分割字符的

如果要输出两个值$1 $2 ，则需要中间以逗号隔开，否则awl将连接相邻的所有值，区别如下：

1.1.awk中有两个特殊的“模式”，它们提供awk程序的起始和清除的操作。这两种模式是可选用的，允许有多少BENGIN和END

BEGIN  {起始操作代码}

pattern1 {action1}

...

END {清除操作代码}

1.2.if..else语句

awk支持if..else语句，格式如下：

（1）

if(表达式)

​     语句1

else

​     语句2

（2）允许嵌套

if(表达式1） { 

​     if(表达式2）

​          语句1

​     else

​          语句2

}

语句3

else {

​     if(表达式3)

​          语句4

​     else

​          语句5

}

语句6

1.3 while语句

格式：  while(表达式) 语句

1.4 for语句

格式：for(初始表达式;终止条件;步长表达式)   {语句}

1.5 自定义函数

格式：function 函数名(参数表){

​     函数体

}

2.awk高级输入输出

2.1读取下一条记录

 awk的next语句导致awk读取下一个记录并完成模式匹配，然后立即执行相应的操作。通常它用匹配的模式执行操作中的代码。next导致这个记录的任何额外匹配模式被忽略。

 参考链接：<http://www.cnblogs.com/chengmo/archive/2010/10/13/1850145.html>

2.2 简单读取一条记录

  awk的 getline语句用于简单地读取一条记录。如果用户有一个数据记录类似两个物理记录，那么getline将尤其有用。它完成一般字段的分离(设置字段变 量$0 FNR NF NR)。如果成功则返回1，失败则返回0（到达文件尾） 。

参考链接：<http://www.itokit.com/2011/0606/66474.html>

3.awk内置

3.1awk内置变量

| 变量名                       作用                            |
| ------------------------------------------------------------ |
| ARGC               命令行参数个数ARGV               命令行参数排列ENVIRON            支持队列中系统环境变量的使用FILENAME           awk浏览的文件名FNR                浏览文件的记录数FS                 设置输入域分隔符，等价于命令行 -F选项NF                 浏览记录的域的个数NR                 已读的记录数OFS                输出域分隔符ORS                输出记录分隔符RS                 控制记录分隔符 |

3.2 awk内置函数

参考链接：<http://www.cnblogs.com/chengmo/archive/2010/10/08/1845913.html>