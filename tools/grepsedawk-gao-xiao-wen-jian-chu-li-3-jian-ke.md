# grep、sed、awk 高效文件处理 3 剑客！

[https://mp.weixin.qq.com/s/\_9-se4q9l4TkiuJZvCim\_g](https://mp.weixin.qq.com/s/_9-se4q9l4TkiuJZvCim_g)

作者：Java小咖秀[https://juejin.im/post/5efe8dc9f265da22aa0ef08f](https://juejin.im/post/5efe8dc9f265da22aa0ef08f)

> grep、sed、awk我们叫他们三剑客，掌握它们可以更好的运维，提升工作效率，即使不是运维，对我们处理数据都是非常方便的～就很多数据处理来讲，写程序肯定是也能处理的，但是远没有已经存在特定功能的命令更高效，我们只需要操作命令即可。通过本文可以讲解三剑客的一些基础知识和实用，希望大家可以自己动手敲，毕竟自己体会过的印象更深刻，后面还会持续更新。。。

## grep

### 简介

grep是一款强大的文本搜索工具，支持正则表达式。

全称（ global search regular expression\(RE\) and print out the line）

语法:grep \[option\]... PATTERN \[FILE\]...

常用:

```text
usage: grep [-abcDEFGHhIiJLlmnOoqRSsUVvwxZ] [-A num] [-B num] [-C[num]]
 [-e pattern] [-f file] [--binary-files=value] [--color=when]
 [--context[=num]] [--directories=action] [--label] [--line-buffered]
 [--null] [pattern] [file ...]
```

常用参数:

```text
            -v        取反
            -i        忽略大小写
            -c        符合条件的行数
            -n        输出的同时打印行号
            ^*        以*开头         
            *$         以*结尾 
            ^$         空行
```

### 实际使用

准备好一个小故事txt:

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# cat monkey
One day,a little monkey is playing by the well.一天,有只小猴子在井边玩儿.
He looks in the well and shouts :它往井里一瞧,高喊道：
“Oh!My god!The moon has fallen into the well!” “噢!我的天!月亮掉到井里头啦!”
An older monkeys runs over,takes a look,and says,一只大猴子跑来一看,说,
“Goodness me!The moon is really in the water!” “糟啦!月亮掉在井里头啦!”
And olderly monkey comes over.老猴子也跑过来.
He is very surprised as well and cries out:他也非常惊奇,喊道：
“The moon is in the well.” “糟了,月亮掉在井里头了!”
A group of monkeys run over to the well .一群猴子跑到井边来,
They look at the moon in the well and shout:他们看到井里的月亮,喊道：
“The moon did fall into the well!Come on!Let’get it out!”
“月亮掉在井里头啦!快来!让我们把它捞起来!”
Then,the oldest monkey hangs on the tree up side down ,with his feet on the branch .
然后,老猴子倒挂在大树上,
And he pulls the next monkey’s feet with his hands.拉住大猴子的脚,
All the other monkeys follow his suit,其他的猴子一个个跟着,
And they join each other one by one down to the moon in the well.
它们一只连着一只直到井里.
Just before they reach the moon,the oldest monkey raises his head and happens to see the moon in the sky,正好他们摸到月亮的时候,老猴子抬头发现月亮挂在天上呢
He yells excitedly “Don’t be so foolish!The moon is still in the sky!”
它兴奋地大叫：“别蠢了!月亮还好好地挂在天上呢!
```

**直接查找符合条件的行**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# grep moon monkey
“Oh!My god!The moon has fallen into the well!” “噢!我的天!月亮掉到井里头啦!”
“Goodness me!The moon is really in the water!” “糟啦!月亮掉在井里头啦!”
“The moon is in the well.” “糟了,月亮掉在井里头了!”
They look at the moon in the well and shout:他们看到井里的月亮,喊道：
“The moon did fall into the well!Come on!Let’get it out!”
And they join each other one by one down to the moon in the well.
Just before they reach the moon,the oldest monkey raises his head and happens to see the moon in the sky,正好他们摸到月亮的时候,老猴子抬头发现月亮挂在天上呢
He yells excitedly “Don’t be so foolish!The moon is still in the sky!”
```

**查找反向符合条件的行**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# grep -v  moon monkey
One day,a little monkey is playing by the well.一天,有只小猴子在井边玩儿.
He looks in the well and shouts :它往井里一瞧,高喊道：
An older monkeys runs over,takes a look,and says,一只大猴子跑来一看,说,
And olderly monkey comes over.老猴子也跑过来.
He is very surprised as well and cries out:他也非常惊奇,喊道：
A group of monkeys run over to the well .一群猴子跑到井边来,
“月亮掉在井里头啦!快来!让我们把它捞起来!”
Then,the oldest monkey hangs on the tree up side down ,with his feet on the branch .
然后,老猴子倒挂在大树上,
And he pulls the next monkey’s feet with his hands.拉住大猴子的脚,
All the other monkeys follow his suit,其他的猴子一个个跟着,
它们一只连着一只直到井里.
它兴奋地大叫：“别蠢了!月亮还好好地挂在天上呢!”
```

**直接查找符合条件的行数**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# grep -c  moon monkey
8
```

**忽略大小写查找符合条件的行数**

先来看一下直接查找的结果

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# grep my monkey
```

忽略大小写查看

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# grep -i my monkey
“Oh!My god!The moon has fallen into the well!” “噢!我的天!月亮掉到井里头啦!”
```

**查找符合条件的行并输出行号**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# grep -n monkey monkey
1:One day,a little monkey is playing by the well.一天,有只小猴子在井边玩儿.
4:An older monkeys runs over,takes a look,and says,一只大猴子跑来一看,说,
6:And olderly monkey comes over.老猴子也跑过来.
9:A group of monkeys run over to the well .一群猴子跑到井边来,
13:Then,the oldest monkey hangs on the tree up side down ,with his feet on the branch .
15:And he pulls the next monkey’s feet with his hands.拉住大猴子的脚,
16:All the other monkeys follow his suit,其他的猴子一个个跟着,
19:Just before they reach the moon,the oldest monkey raises his head and happens to see the moon in the sky,正好他们摸到月亮的时候,老猴子抬头发现月亮挂在天上呢
```

**查找开头是J的行**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# grep '^J' monkey
Just before they reach the moon,the oldest monkey raises his head and happens to see the moon in the sky,正好他们摸到月亮的时候,老猴子抬头发现月亮挂在天上呢
```

**查找结尾是呢的行**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# grep "呢$" monkey
Just before they reach the moon,the oldest monkey raises his head and happens to see the moon in the sky,正好他们摸到月亮的时候,老猴子抬头发现月亮挂在天上呢
```

大家可以grep --help，查看更多相关的命令，这里就不一一演示了。

### 小结

有了强大的网络以后，很多东西都可以在网上找到，但是基础的一定要自己 熟练掌握，才回在遇到事情的时候不慌。

## sed

sed是一种流编辑器，是一款处理文本比较优秀的工具，可以结合正则表达式一起使用。

#### sed执行过程

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/15/640-20200815100616719-100616.jpg)

#### sed命令

命令: sed

语法 : sed \[选项\]... {命令集} \[输入文件\]...

常用命令:

```text
            d  删除选择的行    
            s   查找    
            y  替换
            i   当前行前面插入一行
            a  当前行后面插入一行
            p  打印行       
            q  退出     

 替换符:

            数字 ：替换第几处    
            g :  全局替换    
            \1:  子串匹配标记，前面搜索可以用元字符集\(..\)
            &:  保留搜索刀的字符用来替换其他字符
```

操作:

**替换**

查看文件:

```text
➜  happy cat word
Twinkle, twinkle, little star
How I wonder what you are
Up above the world so high
Like a diamond in the sky
When the blazing sun is gone
```

替换：

```text
➜  happy sed 's/little/big/' word
Twinkle, twinkle, big star
How I wonder what you are
Up above the world so high
Like a diamond in the sky
When the blazing sun is gone
```

查看文本:

```text
➜  happy cat word1
Oh if there's one thing to be taught
it's dreams are made to be caught
and friends can never be bought
Doesn't matter how long it's been
I know you'll always jump in
'Cause we don't know how to quit
```

全局替换:

```text
➜  happy sed 's/to/can/g' word1
Oh if there's one thing can be taught
it's dreams are made can be caught
and friends can never be bought
Doesn't matter how long it's been
I know you'll always jump in
'Cause we don't know how can quit
```

按行替换（替换2到最后一行\)

```text
➜  happy sed '2,$s/to/can/' word1
Oh if there's one thing to be taught
it's dreams are made can be caught
and friends can never be bought
Doesn't matter how long it's been
I know you'll always jump in
'Cause we don't know how can quit
```

**删除:**

查看文本:

```text
➜  happy cat word
Twinkle, twinkle, little star
How I wonder what you are
Up above the world so high
Like a diamond in the sky
When the blazing sun is gone
```

删除:

```text
➜  happy sed '2d' word
Twinkle, twinkle, little star
Up above the world so high
Like a diamond in the sky
When the blazing sun is gone
```

显示行号:

```text
➜  happy sed '=;2d' word
1
Twinkle, twinkle, little star
2
3
Up above the world so high
4
Like a diamond in the sky
5
When the blazing sun is gone
```

删除第2行到第四行:

```text
➜  happy sed '=;2,4d' word
1
Twinkle, twinkle, little star
2
3
4
5
When the blazing sun is gone
```

**添加行:**

向前插入:

```text
➜  happy echo "hello" | sed 'i\kitty'
kitty
hello
```

向后插入:

```text
➜  happy echo "kitty" | sed 'i\hello'
hello
kitty
```

**修改行:**

替换第二行为hello kitty

```text
➜  happy sed '2c\hello kitty' word
Twinkle, twinkle, little star
hello kitty
Up above the world so high
Like a diamond in the sky
When the blazing sun is gone
```

替换第二行到最后一行为hello kitty

```text
➜  happy sed '2,$c\hello kitty' word
Twinkle, twinkle, little star
hello kitty
```

**写入行**

把带star的行写入c文件中,c提前创建

```text
➜  happy sed -n '/star/w c' word
➜  happy cat c
Twinkle, twinkle, little star
```

**退出**

打印3行后，退出sed

```text
➜  happy sed '3q' word
Twinkle, twinkle, little star
How I wonder what you are
Up above the world so high
```

## awk

### 名字由来

创始人 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 姓氏的首个字母。

### 强大的文本处理工具

比起sed和grep，awk不仅仅是一个小工具，也可以算得上一种小型的编程语言了，支持if判断分支和while循环语句还有它的内置函数等，是一个要比grep和sed更强大的文本处理工具，但也就意味着要学习的东西更多了。

下面来说一下awk的一些基础概念以及实际操作。

### 语法

#### 常用

Usage: awk \[POSIX or GNU style options\] -f progfile \[--\] file ...

Usage: awk \[POSIX or GNU style options\] \[--\] 'program' file ...

### 域

类似数据库列的概念，但它是按照序号来指定的，比如我要第一个列就是，第二列就是2，依此类推。$0就是输出整个文本的内容。默认用空格作为分隔符，当然你可以自己通过-F设置适合自己情况的分隔符。

提前自己编了一段数据，学生以及学生成绩数据表。

| 列数 | 名称 | 描述 |
| :--- | :--- | :--- |
| 1 | Name | 姓名 |
| 2 | Math | 数学 |
| 3 | Chinese | 语文 |
| 4 | English | 英语 |
| 5 | History | 历史 |
| 6 | Sport | 体育 |
| 8 | Grade | 班级 |

"Name Math Chinese English History Sport grade 输出整个文本

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk '{print $0}' students_store
Xiaoka          60   80    40    90   77  class-1
Yizhihua        70    66   50    80   90  class-1
kerwin          80    90   60    70   60  class-2
Fengzheng       90    78    62   40   62  class-2
```

输出第一列（姓名列\)

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk '{print $1}' students_store
Xiaoka
Yizhihua
kerwin
Fengzheng
```

### 模式&动作

```text
awk '{[pattern] action}' {filenames}
```

#### 模式

pattern 可以是

* 条件语句
* 正则

模式的两个特殊字段 BEGIN 和 END \(不指定时匹配或打印行数\)

* BEGIN ：一般用来打印列名称。
* END : 一般用来打印总结性质的字符。

#### 动作

action 在{}内指定，一般用来打印，也可以是一个代码段。

#### 示例

给上面的文本加入标题头:

```text
[root@iz2ze76ybn73dvwmdij06zz ~]#  awk 'BEGIN {print "Name     Math  Chinese  English History  Sport grade\n----------------------------------------------"} {print $0}' students_store

Name         Math  Chinese  English History  Sport  grade
----------------------------------------------------------
Xiaoka       60    80       40      90     77    class-1
Yizhihua     70    66       50      80     90    class-1
kerwin       80    90       60      70     60    class-2
Fengzheng    90    78       62      40     62    class-2
```

仅打印姓名、数学成绩、班级信息，再加一个文尾\(再接再厉\):

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk 'BEGIN {print "Name   Math  grade\n---------------------"} {print $1 2 "\t" $7} END {print "continue to exert oneself"}' students_store

Name     Math  grade
---------------------
Xiaoka   60   class-1
Yizhihua 70   class-1
kerwin   80   class-2
Fengzheng 90  class-2
continue to exert oneself
```

### 结合正则

像grep和sed也是支持正则表达式的。这边就不介绍正则表达式了，如果有兴趣，我单出一个文章。

使用方法：

符号 ~ 后接正则表达式

此时我们再加入一条后来的新同学，并且没有分班。

先来看下现在的数据

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# cat students_store
Xiaoka       60   80    40    90   77  class-1
Yizhihua     70    66   50    80   90  class-1
kerwin       80    90   60    70   60  class-2
Fengzheng    90    78   62    40   62  class-2
xman         -     -     -     -   -    -
```

#### 模糊匹配\|查询已经分班的学生

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk '$0 ~/class/' students_store
Xiaoka       60   80    40    90   77  class-1
Yizhihua     70    66   50    80   90  class-1
kerwin       80    90   60     70  60  class-2
Fengzheng    90    78   62     40  62  class-2
```

#### 精准匹配\|查询1班的学生

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk '$7=="class-1" {print $0}'  students_store
Xiaoka       60   80    40    90   77  class-1
Yizhihua     70    66   50    80   90  class-1
```

#### 反向匹配\|查询不是1班的学生

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk '$7!="class-1" {print $0}'  students_store
kerwin       80    90   60     70   60  class-2
Fengzheng    90    78    62     40  62 class-2
xman         -     -     -     -   -    -
```

#### 比较操作

#### 查询数学大于80的

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk '$2>60 {print $0}'  students_store
Yizhihua     70    66   50    80   90  class-1
kerwin       80    90   60     70   60  class-2
Fengzheng    90    78    62     40  62 class-2
```

#### 查询数学大于英语成绩的

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk '$2 > $4  {print $0}'  students_store
Xiaoka       60   80    40    90   77  class-1
Yizhihua     70    66   50    80   90  class-1
kerwin       80    90   60    70   60  class-2
Fengzheng    90    78    62   40   62  class-2
```

#### 匹配指定字符中的任意字符

在加一列专业，让我们来看看憨憨们的专业，顺便给最后一个新来的同学分个班吧。

然后再来看下此时的数据。

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# cat students_store
Xiaoka       60   80    40    90   77  class-1  Java
Yizhihua     70    66   50    80   90  class-1  java
kerwin       80    90   60     70   60  class-2 Java
Fengzheng    90    78    62     40  62 class-2  java
xman         -     -     -     -   -    class-3 php
```

#### 或关系匹配\|查询1班和3班的学生

```text
root@iz2ze76ybn73dvwmdij06zz ~]# awk '$0 ~/(class-1|class-3)/' students_store
Xiaoka       60   80    40    90   77  class-1  Java
Yizhihua     70    66   50    80   90  class-1  java
xman         -     -     -     -   -   class-3 php
```

#### 任意字符匹配\|名字第二个字母是

字符解释：

^ : 字段或记录的开头。

. : 任意字符。

```text
root@iz2ze76ybn73dvwmdij06zz ~]# awk '$0 ~/(class-1|class-3)/' students_store
Xiaoka       60   80    40    90   77  class-1  Java
Yizhihua     70    66   50    80   90  class-1  java
xman         -     -     -     -   -    class-3 php
```

### 复合表达式

#### && AND

的关系，必同时满足才行哦~

查询数学成绩大于60并且语文成绩也大于60的童鞋。

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk '{ if ($2 > 60 && $3 > 60) print $0}' students_store
Yizhihua     70    66   50    80   90  class-1  java
kerwin       80    90   60    70   60  class-2  Java
Fengzheng    90    78    62   40   62  class-2  java
```

#### \|\|  OR

查询数学大于80或者语文大于80的童鞋。

```text
[root@iz2ze76ybn73dvwmdij06zz ~]#  awk '{ if ($2 > 80 || $4 > 80) print $0}' students_store
Fengzheng    90    78    62     40  62 class-2  java
```

### printf 格式化输出

除了能达到功能以外，一个好看的格式也是必不可少的，因此格式化的输出看起来会更舒服哦～

#### 语法

printf \(\[格式\]，参数\)

printf %x\(格式\) 具体参数 x代表具体格式

| 符号 | 说明 |
| :--- | :--- |
| - | 左对齐 |
| Width | 域的步长 |
| .prec | 最大字符串长度或小数点右边位数 |

格式转化符

其实和其他语言大同小异的

常用格式

| 符号 | 描述 |
| :--- | :--- |
| %c | ASCII |
| %d | 整数 |
| %o | 八进制 |
| %x | 十六进制数 |
| %f | 浮点数 |
| %e | 浮点数（科学记数法\) |
| % s | 字符串 |
| %g | 决定使用浮点转化e/f |

#### 具体操作示例

ASCII码🐎

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# echo "66" | awk '{printf "%c\n",$0}'
B
```

浮点数

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk 'BEGIN {printf "%f\n",100}'
100.000000
```

16进制数

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk 'BEGIN {printf "%x",996}'
3e4
```

更多操作，小伙伴有兴趣可以挨个试试~

### 内置变量

#### 频率较高常用内置变量

NF ：记录浏览域的个数，在记录被读后设置。

NR ：已读的记录数。

FS : 设置输入域分隔符

A R G C ：命令行参数个数，支持命令行传入。

RS : 控制记录分隔符

FIlENAME : awk当前读文件的名称

#### 操作

输出学生成绩表和域个数以及已读记录数。

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk '{print $0, NF , NR}' students_store
Xiaoka       60   80    40    90   77  class-1  Java 8 1
Yizhihua     70    66   50    80   90  class-1  java 8 2
kerwin       80    90   60     70  60  class-2  Java 8 3
Fengzheng    90    78   62     40  62  class-2  java 8 4
xman         -     -     -     -   -   class-3  php  8 5
```

### 内置函数

#### 常用函数

length\(s\) 返回s长度

index\(s,t\) 返回s中字符串t第一次出现的位置

match \(s,r\) s中是否包含r字符串

split\(s,a,fs\) 在fs上将s分成序列a

gsub\(r,s\) 用s代替r，范围全文本

gsub\(r,s,t\) 范围t中，s代替r

substr\(s,p\) 返回字符串s从第p个位置开始后面的部分（下标是从1 开始算的，大家可以自己试试）

substr\(s,p,n\) 返回字符串s从第p个位置开始后面n个字符串的部分

#### 操作

**length**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk 'BEGIN {print length(" hello,im xiaoka")}'
16
```

**index**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk 'BEGIN {print index("xiaoka","ok")}'
4
```

**match**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk 'BEGIN {print match("Java小咖秀","va小")}'
3
```

**gsub**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk 'gsub("Xiaoka","xk") {print $0}' students_store
xk       60   80    40    90   77  class-1  Java
```

**substr\(s,p\)**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk 'BEGIN {print substr("xiaoka",3)}'
aoka
```

**substr\(s,p,n\)**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# awk 'BEGIN {print substr("xiaoka",3,2)}'
ao
```

**split**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# str="java,xiao,ka,xiu"
[root@iz2ze76ybn73dvwmdij06zz ~]# awk 'BEGIN{split('"\"$str\""',ary,","); for(i in ary) {if(ary[i]>1) print ary[i]}}'
xiu
java
xiao
ka
```

### awk脚本

前面说过awk是可以说是一个小型编程语言。如果命令比较短我们可以直接在命令行执行，当命令行比较长的时候，可以使用脚本来处理，比命令行的可读性更高，还可以加上注释。

#### 写一个完整的awk脚本并执行步骤

**1.先创建一个awk文件**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# vim printname.awk
```

**2.脚本第一行要指定解释器**

```text
#!/usr/bin/awk -f
```

**3.编写脚本内容，打印一下名称**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# cat printname.awk
#!/usr/bin/awk -f
#可以加注释了，哈哈
BEGIN { print "my name is Java小咖秀"}
```

**4.既然是脚本，必不可少的可执行权限安排上~**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# chmod +x printname.awk
[root@iz2ze76ybn73dvwmdij06zz ~]# ll printname.awk
-rwxr-xr-x 1 root root 60 7月   1 15:23 printname.awk
```

**5.有了可执行权限，我们来执行下看结果**

```text
[root@iz2ze76ybn73dvwmdij06zz ~]# ./printname.awk
my name is Java小咖秀
```

了解了写awk脚本的步骤以后大家就可以自己去写一波了～

