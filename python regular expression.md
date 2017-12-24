Python正则表达式

一：正则表达式的符号与方法

常用符号：

.:匹配任何一个字符，换行符除外（所以，多行字符串中的匹配要特殊处理，见下面实例）

*:匹配前一个字符0次或多次

+：匹配前一个字符1次或多次

?:匹配前一个字符0次或1次

{n,m}:匹配前一个字符n次~m次

()：小括号内容作为结果返回

[]:匹配[]内任一元素

.*:贪心匹配，匹配任何一个字符0次或多次

.*?:非贪心匹配

(.*?)：把括号内的匹配内容作为结果返回

\w：匹配包括下划线的任何单词字符（即不是特殊符号的字符，符号字符是!@#?之类），相当于[A-Za-z0-9_]

\d：匹配任意数字，等价于 [0-9]

\W:匹配非单词字符。

\D：匹配非数字。

^:模式取非。

在Python的正则表达式中，有一个参数为re.S。它表示“.”（不包含外侧双引号，下同）的作用扩展到整个字符串，包括“\n”。
 正则表达式中，“.”的作用是匹配除“\n”以外的任何字符，也就是说，它是在一行中进行匹配。这里的“行”是以“\n”进行区分的。使用re.S参数以后，正则表达式会将这个字符串作为一个整体，将“\n”当做一个普通的字符加入到这个字符串中，在整体中进行匹配。

在re.py库的介绍中有以下语句：

"." Matches any character except a newline.

S DOTALL "." matches any character at all, including the newline.


常用方法：

findall()：匹配符合规则的内容，结果以列表形式返回

search()：匹配第一个符合规则的内容，并返回一个正则表达式对象

sub()：替换符合规则的内容，返回替换后的值

 

实例：

复制代码
# . 的使用：充当一个占位符，一个.代表一个符号
a="xz123"
b=re.findall("x.",a)  #第一个参数为查找规则，第二个参数为被查找对象
print b
结果：. 被赋予了具体内容
['xz']


# * 的使用：代表前一个字符出现0次或多次
a="xyxy123"
b=re.findall("x*",a)
print b
结果：逐个匹配：0次则匹配结果为空
['x', '', 'x', '', '', '', '', '']

# ? 的使用：匹配前一个字符0次或1次
a="xxxy123"
b=re.findall("x?",a)
print b
结果：逐位匹配，符合则打印，不符合则为空（注意：最后一位是\n）
['x', 'x', 'x', '', '', '', '', '']

# .* 的使用：贪心匹配，前后界限之间的字符尽可能多地匹配到
code="dfhasjkhxxIxxjkfasl244xxlovexxdhfjkh3455xxyouxxklfjsgdiou5"
b=re.findall("xx.*xx",code)
print b
结果：前后界限为xx  xx ,那么被查找对象中xx 与 xx 之间范围会尽量长
['xxIxxjkfasl244xxlovexxdhfjkh3455xxyouxx']

# .*? 的使用：非贪心匹配：. 匹配字符，* 控制字符多少， ？ 控制前面的字符子串出现的次数（0次或1次）
code="dfhasjkhxxIxxjkfasl244xxlovexxdhfjkh3455xxyouxxklfjsgdiou5"
b=re.findall("xx.*?xx",code)
print b
结果：符合前后界限的匹配子串会尽量短地匹配到
['xxIxx', 'xxlovexx', 'xxyouxx']

# (.*?) 的使用：. 匹配字符，* 控制字符多少， ？ 控制前面的字符子串出现的次数，()控制了返回结果：我们需要的就放在括号里，不需要的就放在括号外
code="dfhasjkhxxIxxjkfasl244xxlovexxdhfjkh3455xxyouxxklfjsgdiou5"
b=re.findall("xx(.*?)xx",code)
print b
结果：把符合前后界限匹配规则的结果中，括号内的匹配内容返回，而省略前后界限
['I', 'love', 'you']
复制代码
特殊实例：

复制代码
#多行字符串匹配
#如果按照前面的匹配方式，结果会这样
s='''jhfsdxxhello
xxjhdfxxworldxxh234'''
b=re.findall("xx(.*?)xx",s)
print b
输出：由于 . 不能匹配换行符\n，所以这里只能匹配到同一行中符合规则的内容
['jhdf']

#跨行匹配：使用第三个参数：re.S,使 . 可以匹配\n
s='''jhfsdxxhello
xxjhdfxxworldxxh234'''
b=re.findall("xx(.*?)xx",s,re.S)
print b
结果：
['hello\n', 'world']
复制代码
 

比较：search与findall

复制代码
#search方法
#1：单纯调用
s='jhfsdxxhelloxx123xxworldxxh234'
b=re.search("xx(.*?)xx123xx(.*?)xx",s)
print b
结果：返回第一个匹配对象
<_sre.SRE_Match object at 0x02A92728>

#2：要取用具体的匹配内容，需要用group()方法
s='jhfsdxxhelloxx123xxworldxxh234'
b=re.search("xx(.*?)xx123xx(.*?)xx",s).group()
print b
结果：
xxhelloxx123xxworldxx

#3：如果规则中有多处()，可以用序号取用匹配结果中的第n处括号内的匹配内容
s='jhfsdxxhelloxx123xxworldxxh234'
b=re.search("xx(.*?)xx123xx(.*?)xx",s).group(1)
print b
结果：括号序号是从1开始的
hello
复制代码
复制代码
#findall()
#1：单纯调用，输出结果
s='jhfsdxxhelloxx123xxworldxxh234'
b=re.findall("xx(.*?)xx123xx(.*?)xx",s)
print b
结果：()匹配结果以列表方式返回，一个列表元素是一个元组，元组内容为众括号匹配内容
[('hello', 'world')]

#2：对返回结果进行具体取用
s='jhfsdxxhelloxx123xxworldxxh234'
b=re.findall("xx(.*?)xx123xx(.*?)xx",s)
print b[0]
结果：b[0]是一个元组
('hello', 'world')

s='jhfsdxxhelloxx123xxworldxxh234'
b=re.findall("xx(.*?)xx123xx(.*?)xx",s)
print b[1]
结果：报错
    print b[1]
IndexError: list index out of range

#3：由1，2可知，要具体访问一个()的匹配结果，需要用二维数组的形式，第一个下标访问结果列表中的元组，第二个下标访问元组中的具体元素
s='jhfsdxxhelloxx123xxworldxxh234'
b=re.findall("xx(.*?)xx123xx(.*?)xx",s)
print b[0][1]
结果：
world
复制代码
可见search与findall方法对匹配结果的取用不同：search返回结果以group(i)取用具体括号的匹配内容，findall返回结果以二维数组形式[i][j]取用具体括号匹配内容。

 

sub()的使用：替换

复制代码
1：
s='jhfsdxxhelloxx123xxworldxxh234'
b=re.sub("xx(.*?)xx","IiiI",s)
print b
结果：把包括前后缀的匹配模式的匹配内容全部替换了
jhfsdIiiI123IiiIh234

2：
s='jhfsdxxhelloxx123xxworldxxh234'
b=re.sub("xx(.*?)xx123xx(.*?)xx","IiiI",s)
print b
结果：不是按括号分开替换，而是把符合匹配规则的一整块内容替换
jhfsdIiiIh234
复制代码
 

二：一些使用技巧

1：建议使用import re，然后在代码中使用 re.XX 调用方法、常量，来方便区分正则内容与代码中自定义的内容。

2：不建议使用compile方式进行正则表达式匹配。

复制代码
compile方式的正则表达式的使用：
#1：创建被查找的对象
s='jhfsdxxhelloxx123xxworldxxh234'
#2：定义正则字符串
pattern_str="xx(.*?)xx"
#3：使用compile根据正则字符串创建模式
pattern=re.compile(pattern_str,re.S)#第二个参数是模式控制常量：如这里S是令.可以匹配\n
#4：使用findall、search方法进行内容查找
res=re.findall(pattern,s)
print res
结果：
['hello', 'world']
复制代码
不建议这样使用的原因：

在findall方法中，调用了_compile()方法创建模式；

而compile()方法底层其实就是return _compile(str)，之后再调用findall(pattern，s)时又调用一次_compile(str)就是多此一举了。

因此，我们不需要先compile，直接把正则字符串传给findall(str,s)作为匹配模式即可。
