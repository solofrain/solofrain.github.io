Regex
==

# 1. 基本正则表达式

匹配字符 | 描述
:-:|-
<font color=Blue>.</font> 	| <font color=Blue>一个点，用来匹配任何字符。（）</font>
<font color=Blue>*  |	<font color=Blue>星号，匹配前面模式中的零个或者任意个</font>
<font color=Blue>+  |	<font color=Blue>加号，匹配前面模式中的一个或者任意个</font>
<font color=Blue>? 	| <font color=Blue>问号，匹配前面模式中的零个或者一个</font>
<font color=Brown>() | <font color=Brown>括号，创建一个子模式</font>
<font color=Brown>[] | <font color=Brown>中括号，用来表示一个区间</font>
<font color=Brown>\| | <font color=Brown>竖号，交替匹配</font>
<font color=Brown>$  |	<font color=Brown>美元号，将一个模式挂靠在要匹配的字符串的最后面</font>
<font color=Brown>^ | <font color=Brown>尖号，将一个模式挂靠在要匹配的字符串的最前面</font>
& |	 一行的开头
{2,4}  |	大括号可以用来更精确地限定字符的数目 

- 字符关键字

    这部分关键字包括26个英文字符（上面的表格没有列出来）。这些关键字的特点就是它们匹配自身。

- 数量关键字

    这部分关键字包括<font color=Blue> . </font>（点）<font color=Blue> *</font> （星号） <font color=Blue>+ </font>（加号） <font color=Blue>? </font>（问号）这4个关键字，这中间 . （点）这个关键字稍微特殊一点，因为它有2个作用：既可以作为字符关键字表示任何字符，又可以作为数量关键字代表1个字符。

    > 空字符也算任何字符，也就是说一个点可以表示有一个字符，也可以表示没有字符。**

- 模式关键字

    <font color=Brown>()</font> （括号） <font color=Brown>|</font> （竖号）<font color=Brown> []</font> （中括号） <font color=Brown>^</font> （尖号） <font color=Brown>$ </font>（美元号）这5个符号都属于模式关键字，它们要么代表模式本身（括号、竖号、中括号），要么作用于模式为模式提供其他更高级的功能（尖号、美元号）。

**模式**

模式就是一组用来匹配字符的关键字集合，一个最小的模式只有一个关键字，而大的模式则可以有无数个关键字：

```tcl
A     # 这是一个模式，代表A这个字符本身

A+    # 这也是一个模式，代表一个或者任意多个A字符
```

- 常用的正则表达式

正则表达式 | 	描述
:-: |-
[0-9]+ | 	匹配正整数
[-+]?[0-9]+(\.[0-9]+)? 	| 匹配数值，包括小数
[a-zA-Z_0-9]+ |	由字母、数字、下划线组成的单词
&Error | 	以字符串"Error"开始的行
&[ \t]*$ 或者 &\s*$ |	匹配空行 

-     在Tcl命令中，最好把正则表达式写在大括号中，这样有利于避免字符转义

- 常用命令

    - regexp

        语法：

        ```tcl
        regexp  [选项]  <正则表达式>  <匹配的原始字符串>  <保存匹配后字符串的变量>  [其他保存子模式匹配字符串的变量]
        ```

        - regexp 用来对字符串进行匹配。不仅可以用来测试是复匹配，还可以提取匹配的字符。
        - regexp 的返回值表示匹配是否成功
        - 忽略大小写可以用 "-nocase" 选项

        例：
        ```tcl
        regexp  {A+}  "AABBCC"  match
        puts  $match
        ```

        Output:

        ```tcl
        AA
        ```

        ```tcl
        set str "This is a cat"
        set rv [regexp {is a ([a-z]+)} $str str_match str_1
        puts $rv          ;  # rv = 1
        # 匹配到的字符串
        puts $str_match   ; # str_match = is a cat
        # 正则表达式中括号部分对应的字符串
        puts $str_1       ; # str_1     = cat
        ```

        - 将匹配结果作为List返回。`"-inline"` 选项的作用。
                
            ```tcl
            set rv [regexp -inline {is a ([a-z]+)} $str ]
            puts $rv ; # rv = {{is a cat} {cat}}
            ```

    - regsub

        - regsub 利用正则表达式对字符串进行替换操作
        - 忽略大小写可以用 "-nocase" 选项
        - 全部替换可以用 "-all" 选项

        ```tcl
        set str "This is a cat, another cat"
        # 返回值是替换后的字符串
        set str_2 [regsub {cat} $str dog]
        # 也可以直接赋值给新的变量
        regsub {cat} $str dog str_3
        puts $str_2  ; # str_2 = This is a dog, another cat
        puts $str_3  ; # str_3 = This is a dog, another cat

        # 默认 regsub 只替换第一个遇到的匹配
        # 替换所有的匹配可以使用 -all
        regsub -all {cat} $str dog $str_4
        puts $str_4  ; # str_3 = This is a dog, another dog
        ```


## 1.1 模式关键字

### 1.1.1 `()` 子模式匹配关键字

小括号用来将一个大模式分为几段更小的模式，这样就可以更加精细的控制匹配方式了，我们来看一个例子：

```tcl
regexp -- {(AA)(BB)(CC)} "AABBCC" match sub1 sub2 sub3

puts "The match is:$match"

puts "The sub1 is:$sub1"

puts "The sub2 is:$sub2"

puts "The sub3 is:$sub3"
```

输出：

```tcl
The match is:AABBCC

The sub1 is:AA

The sub2 is:BB

The sub3 is:CC
```

上面的例子中，处于{}之间的内容是一个完整的正则表达式，在正则表达式里面我们用()将表达式分为3个子模式，后面的match变量中保存所有已经匹配到的字符，而几个sub?变量则保存相应子模式中匹配到的字符。

### 1.1.2 `|`     交替匹配关键字

交替匹配用来匹配|符号二边的一个模式，比如下面的例子：

```tcl
TOPSEC|topsec
```

上面的表达式表示匹配要么是全部大写的 `TOPSEC`，要么是全部小写的 `topsec`，不能2个都同时匹配。

### 1.1.3 `[]`    区间匹配

区间匹配用来表示匹配一系列字符串中间的一个，比如下面的例子：

```tcl
regexp {[ADEFG]} "AAABBBCCC" match

puts $match
```

输出：

```tcl
A
```

上面的表达式表示匹配 `ABCDE` 这5个字符中的一个，注意：只是一个

如果想匹配多个呢？可以使用数量关键字辅助：

```tcl
regexp {[ADEFG]+} "AAABBBCCC" match

puts $match
```

输出：

```tcl
AAA
```
 
区间匹配还可以使用 `[a-z]` 这样的语法来表示匹配从小写 `a` 到小写 `z` 这26个小写字母中的一个

这个关键字使用必须非常小心，因为在TCL语言中 `[]` 还有另外一个含义：所有处于 `[]` 中的内容是一条TCL命令，因此在 `regexp` 中使用的时候，必须用 `{}` 将 `[]` 的其他含义取消掉，如果将 `{}` 换成 `""`，那么上面的命令会报错。

### 1.1.4 `^ `    挂靠匹配，将模式挂靠在字符串的开头

这是一个很特殊的关键字，它不像其他关键字是作用于左边的模式上，而是作用于右边的模式上，千万注意这一点！它表示从要匹配的字符串的最前面开始匹配，我们来看一个比较的例子：

```tcl
regexp  {(AAA)}  "BBBAAACCC"  match
```

可以匹配到，match中的值是 `AAA`，但是我们加上挂靠匹配字符之后呢：

```tcl
regexp  {^(AAA)}  "BBBAAACCC"  match
```

无法匹配，match中的值为空，因为 `^` 符号要求必须从要匹配的字符最前面开始匹配，可惜要匹配的字符最前面是 `BBB`，所以无法匹配到。

`^` 这个字符也有二义性，如果把它放在中括号 `[]` 里面的话，它表示【非】的意思，比如 `[^a-z]` 表示匹配不是 `a-z` 字母的其他字符，但是不在中括号里面，比如 `^ab`表示必须最前面是 `ab` 这2个字符，这是很容易搞混的地方，一定要注意了。

### 1.1.5 `$`     挂靠匹配，将模式挂靠在字符串的结尾

这个关键字与 `^` 关键字作用相反，但是它和其他关键字一样，是作用于左边的模式上，还是看看例子：

```tcl
regexp  {(AAA)$}  "BBBCCCAAA"  match
```

可以匹配到，因为要匹配的字符最后面是 `AAA`，如果要匹配的字符是 `BBBAAACCC` 这样的，就无法匹配到了。

## 1.2 数量关键字

. （点） * （星号） + （加号） ? （问号）用来表示数量。

- `.`   匹配任意一个字符

`.`（点）是一个比较特殊的字符，它虽然表示匹配任意一个字符，但实际上任意字符也包括空字符。

- `*`       匹配前面模式中的零个或任意多个

零个这个概念很重要，也就是说不管有没有都会匹配，所以一般我们都会用 `.*` 这样的方式来表示任意多个任意字符，不管有没有都可以。

- `+`    匹配前面模式中的1个或任意多个

- `?`    匹配前面模式中的0个或1个

`?` 号还有一个术语——非贪婪模式，这也是正则表达式中非常重要的内容，所谓非贪婪模式，就是表示只要匹配到第一个就会停下来，而贪婪模式正好相反，它会尽可能多的匹配，这2种模式的最终结果就是：非贪婪模式总是获得第一个匹配，贪婪模式总是获得最后一个匹配。默认情况下，正则表达式总是处于贪婪模式下的。

# 2. 高级正则表达式

## 2.1 反斜杠字符序列

反斜杠序列 | 简要说明
:-:|-
\d | 表示0-9之间的数字
\D | 除了0-9之间数字的其他字符，与\d作用相反
\s |  空白符，包括空格、换行、回车、制表、垂直制表、换页符等
\S |  非空白符，与\s作用相反
\w | 数字、字母和下划线
\W | 非数字、字母和下划线的其他字符
\uXXXX | 16位Unicode字符编码
\n | 换行符，Unicode码是\u000A
\r | 换页符，Unicode码是\u000D
\t | 制表符，Unicode码是\u0009

## 2.2 字符类

除了反斜杠字符序列，高级正则表达式还支持字符类匹配，字符类就是利用一个单词代表复杂意思，大部分的字符类与反斜杠序列含义相同，但也有一些字符类是特有的，比如匹配16进制字符的xdigit，几乎所有情况下只要使用字符类就必须将它们放在[[: :]]符号中，下面的表格列出了所有字符类：

字符类 | 简要说明
:-:|-
[[:alnum:]] | 大小写字母和数字，不包括下划线
[[:alpha:]] | 大小写字母
[[:blank:]] | 空格和制表符
[[:cntrl:]] | 控制字符，也就是ASCII码表中1-31号的字符
[[:digit:]] | 0-9之间的数字，与\d的含义相同
[[:graph:]] | 所有可以显示的字符
[[:lower:]] | 小写字母
[[:print:]] | alnum的另外一种表示方法
[[:punct:]] | 所有标点字符
[[:space:]] | 空白字符，与\s的含义相同
[[:upper:]] | 所有大写字母
[[:xdigit:]] | 所有16进制数字，包括0-9 a-f A-F

# 3. 扩展的正则表达式语法

扩展语法中，我认为最为重要和方便的就是{}语法，它可以精确指定前面模式匹配的次数，{}语法有3种基本使用方法：

- {m}       匹配前面模式的m次

- {m,}      匹配前面模式最少m次，最多无限次

- {m,n}     匹配前面模式最少m次，最多n次

在实际使用时还可以在 `{}` 语法后面加上 `?` 号表示非贪婪匹配。

# 4. 实例详细说明

## 4.1  从tcpdump中，提取IP和端口号。

```tcl
set dumpoutput {
16:49:52.278091 IP 10.11.105.15.2093 > 10.11.105.102.ssh: . ack 167128 win 14944
16:49:52.292780 IP 10.11.105.15.2093 > 10.11.105.102.ssh: . ack 167332 win 16232}

set pattern {.*(10.11.105.15)\.+?(\d+)\s+?>+?}

set status [regexp $pattern $dumpoutput tp iptp port]

puts "ip is:$iptp"

puts "port is: $port"
```

输出：

```tcl
ip is:10.11.105.15

port is: 2093
```

上面的代码中，`dumpoutput` 变量是从 `tcpdump` 程序中截获的报文，最重要的正则表达式是 `pattern` 变量中的内容，观察一个正则表达式，应该首先观察它的子模式，从子模式中一般我们可以看到正则表达式中最重要最核心的部分，然后再观察外围的其他字符。

上面的代码中有2个子模式，第一个子模式用来匹配IP地址，第二个子模式则使用高级正则表达式中的反斜杠字符序列，`\d` 表示任意数值，后面的 `+?` 则用来匹配任意多个数值。

外围的代码中，大量使用了 `?` 的非贪婪特性，其中 `\s` 这个反斜杠序列表示任意空白符号。

## 4.2  从tcpdump中，提取arp应答信息

```tcl
set dumpout {17:14:24.927839 arp who-has 10.11.105.254 tell 10.11.105.102
17:14:24.927936 arp reply 10.11.105.254 is-at 00:13:72:35:a6:fd}

set pattern {arp reply 10.11.105.254}
set st [regexp -- $pattern $dumpout match]
puts $match
```

这个正则表达式很简单，就是让关键字一个一个的对应匹配，其实刚刚开始写正则表达式有一个小技巧——首先将关键字全部复制出来，然后一点一点的替换，比如将空格替换成 `\s+`，数值替换成 `\d+` 等等。

## 4.3  检查arp表中是否清空了指定IP的arp记录

```tcl
set pcarp {
Address                  HWtype  HWaddress           Flags Mask            Iface
10.11.105.29                     (incomplete)                              eth0
10.11.105.19             ether   00:11:D8:35:13:84   C                     eth0}

set pattern {(10.11.105.29)+?.*?incomplete+?}
set patt "\u000A*\u000D*"
regsub -all -- $patt $pcarp {} pcarp
set st [regexp -- $pattern $pcarp match]
puts $match
```
输出：

```tcl
10.11.105.29                     (incomplete
```

上面的表达式使用了 `?` 这个非贪婪匹配关键字

## 4.4   从FW上获取系统当前时间

```tcl
set fwout {+00 2007-07-24 08:25:38}

set pat {.*(\+[0-9]{2})\s+([0-9]{4}-[0-9]{2}-[0-9]{2})\s+([0-9]{2}:[0-9]{2}:[0-9]{2}).*}
set st [regexp $pat $fwout - t1 t2 t3]
puts "time area:$t1\ndate:$t2\ntime:$t3"

set pat {([0-9]{2}):([0-9]{2}):([0-9]{2})}
regexp $pat $t3 - hour minute second
puts "hour:$hour\nminute:$minute\nsecond:$second"

set pat {([0-9]{4})-([0-9]{2})-([0-9]{2})}
regexp $pat $t2 - year month date
puts "year:$year\nmonth:$month\ndate:$date"
```

这个表达式使用了高级正则表达式中的概念，在模式后面用` {}` 括起来的数字表示匹配前面的模式多少次，利用子模式可以单独提取内容。

下面的实例除非必要就不再解释，请仔细观察。

## 4.5   从ifconfig 端口号中，获得IP地址。

```tcl
set result [exec ifconfig eth1]

set pat {(inet addr:)([^\s]+)\s+(Bcast:.*)}

regexp $pat $result - - ip

puts "ip is :$ip"
```

`regexp` 命令中的 `-` 表示不获取那个子模式中的值，因为这里使用了2个 `-`，因此ip变量获取的就是第2个子模式的值了（第一个 `-` 获取整个表达式匹配的所有字符，第二个 `-` 获取第一个子模式中的值。