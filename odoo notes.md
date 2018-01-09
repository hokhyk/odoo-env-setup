# creating the module basic skeleton
$ ~/odoo-dev/odoo/odoo-bin scaffold --help
python odoo-bin 模块名 目录名
即可使用odoo自带的模板创建一个模块。
模板位于odoo/openerp/cli/template/default下，可以修改模板来定制自己需要的模板.
usage: odoo-bin scaffold [-h] [-t TEMPLATE] name [dest]

Generates an Odoo module skeleton.

positional arguments:
  name                  Name of the module to create
  dest                  Directory to create the module in (default: .)

optional arguments:
  -h, --help            show this help message and exit
  -t TEMPLATE, --template TEMPLATE
                        Use a custom module template, can be a template name
                        or the path to a module template (default: default)

Built-in templates available are: theme, default

# ORM字段类型
http://www.cnblogs.com/ygj0930/p/7044480.html
一：Odoo中常用字段类型

基本字段类型9种：

Binary：二进制类型（文件类型），odoo中的Binary字段在视图层显示为一个文件上传按钮。可以把图片、音频、视频、文档等等文件以二进制形式保存。

Char：字符型，size属性定义字符串长度。

Boolean：布尔型

Float：浮点型，如 rate = fields.float('Relative Change rate',digits=(12,6)), digits定义数字总长和小数部分的位数。

Integer：整型

Date：短日期，年月日，在view层以日历选择框显示。

Datetime：时间戳

Text：文本型，多用于多行文本框。可以用widget属性为它添加样式。

Html：与text类似，用于多行文本编辑，不过自带编辑器样式。

 

复杂类型2种：

Selection：下拉列表。

function：函数型，字段值由函数计算而得，不存储在数据表中。

如：fields.function(fnct, arg=None, fnct_inv=None, fnct_inv_arg=None, type='float', fnct_search=None, obj=None, method=False, store=True)·

参数解释：

type 是函数返回值的类型。·

method 为True表示本字段的函数是对象的一个方法，为False表示是全局函数，不是对象的方法。如果method=True，obj指定method所属对象。·

fcnt 是函数或方法，用于计算字段值。如果method = true, 表示fcnt是对象的方法。

fcnt_inv 是用于写本字段的函数或方法。

fcnt_search 定义该字段的搜索行为。

store 表示是否希望在数据库中存储本字段值，缺省值为False。不过store还有一个增强形式，格式为 store={'object_name':(function_name,['field_name1','field_name2'],priority)} ，其含义是，如果对象'object_name'的字段['field_name1','field_name2']发生任何改变，系统将调用函数function_name，函数的返回结果将作为参数(arg)传送给本字段的主函数，即fnct。

 

对象字段类型4种：

one2one: 一对一关系。

格式为：fields.one2one(关联对象Name, 字段显示名, ... )。在V5.0以后的版本中不建议使用，而是用many2one替代。

many2one: 多对一关系，格式为：fields.many2one(关联对象Name, 字段显示名, ... )。可选参数有：ondelete，可选值为"cascade"和"null"，缺省值为"null"，表示one端的record被删除后，many端的record是否级联删除。

复制代码
参数列表：
comodel_name(string) -- 目标模型名称，除非是关联字段否则该参数必选
domain -- 可选，用于在客户端筛选数据的domain表达式
context -- 可选，用于在客户端处理时使用
ondelete -- 当所引用的数据被删除时采取的操作，取值：'set null', 'restrict', 'cascade'
auto_join -- 在搜索该字段时是否自动生成JOIN条件，默认False
delegate -- 设置为True时可以通过当前model访问目标model的字段，与_inherits功能相同
复制代码
 

one2many: 一对多关系，格式为：fields.one2many(关联对象Name, 关联字段, 字段显示名, ... ),例：'address'=fields.one2many('res.partner.address', 'partner_id', 'Contacts')。

复制代码
参数列表：
comodel_name -- 目标模型名称，
inverse_name -- 在comodel_name 中对应的Many2one字段
domain -- 可选，用于在客户端筛选数据的domain表达式
context -- 可选，用于在客户端处理时使用
auto_join -- 在搜索该字段时是否自动生成JOIN条件，默认False
limit(integer) -- 可选，在读取时限制数量
复制代码
 

many2many: 多对多关系。

复制代码
comodel_name -- 目标模型名称，除非是关联字段否则该参数必选
relation -- 关联的model在数据库存储的表名，默认采用comodel_name获取数据
column1 -- 与relation表记录相关联的列名
column2 --与relation表记录相关联的列名
domain -- 用于在客户端筛选数据的domain表达式
context -- 用于在客户端处理时使用
limit(integer) --在读取时限制数量
复制代码
例如：'category_id'=fields.many2many('res.partner.category','res_partner_category_rel','partner_id','category_id','Categories')

表示以多对多关系关联到对象res.partner.category，关联表为'res_partner_category_rel'，关联字段为 'partner_id'和'category_id'。当定义上述字段时，OpenERP会自动创建关联表为 'res_partner_category_rel'，它含有关联字段'partner_id'和'category_id'。

 

（reference: 引用型已经很少使用了，可以忽略）

复制代码
class Stage(models.Model):  
    _name = 'todo.task.stage'  （对应表名）
    _order = 'sequence,name'  
    _description='类的描述，对应表的描述'
    //以下是该类对应表中的字段们
    # String fields:  
    name = fields.Char('Name', 40)  
    desc = fields.Text('Description')  
    state = fields.Selection(  
        [('draft','New'), ('open','Started'),('done','Closed')],  
        'State')  
    docs = fields.Html('Documentation')  
    # Numeric fields:  
    sequence = fields.Integer('Sequence')  
    perc_complete = fields.Float('% Complete', (3, 2))  
    # Date fields:  
    date_effective = fields.Date('Effective Date')  
    date_changed = fields.Datetime('Last Changed')  
    # Other fields:  
    fold = fields.Boolean('Folded?')  
    image = fields.Binary('Image') 
复制代码
（切记：每句后面不能有逗号 ,  否则不会同步到postgresql中。）

 

二：字段属性参数

 字段定义中可用的参数有：change_default，readonly，required，states，string，translate，size，priority，domain，invisible，context等。

change_default：别的字段的缺省值是否可依赖于本字段，缺省值为：False。

例如： 'zip'= fields.char('Zip', change_default=True, size=24),这个例子中，可以根据zip的值设定其它字段的缺省值，例如，可以通过程序代码，如果zip为200000则city设为“上海”，如果zip为100000则city为“北京”。

readonly: 本字段是否只读，缺省值：False。

required: 本字段是否必须的，缺省值：False

states: 定义特定state值才生效的属性，格式为：{'name_of_the_state': list_of_attributes}，其中list_of_attributes是形如[('name_of_attribute', value), ...]的tuples列表。

如：'partner_id'=fields.many2one('res.partner', 'Partner', states={'posted':[('readonly',True)]})

string: 字段显示名，任意字符串。

translate: 本字段值（不是字段的显示名）是否可翻译，缺省值：False。

size: 字段长度。

invisible: 本字段是否可见，即是否在界面上显示本字段，缺省值True。

domain: 域条件，缺省值：[]。在many2many和many2one类型中，字段值是关联表的id，域条件用于过滤关联表的record。

例：'default_credit_account_id'=fields.many2one('account.account', 'Default Credit Account', domain="[('type','!=','view')]")，意思是：本字段关联到对象('account.account')中type不是'view'的record。

 

三：Odoo保留字段

name(Char) -- _rec_name的默认值，在需要用来展示的时候使用
active(Boolean) -- 设置记录的全局可见性，当值为False时通过search和list是获取不到的
sequence(Integer) -- 可修改的排序，可以在列表视图里通过拖拽进行排序
state(Selection) -- 对象的生命周期阶段，通过fileds的states属性使用
parent_id(Many2one) -- 用来对树形结构的记录排序，并激活domain表达式的child_of运算符
parent_left,parent_right -- 与 _parent_store结合使用，提供更好的树形结构数据读取
 

四：自动化属性

在模块安装后，模块中的类会自动添加一些属性，这些属性是odoo自动化添加与修改的，可以在odoo调试模式下，点击一个model进行查看。如：



 

自动化属性主要有：



对应的数据库表中也会自动生成这些字段：



如果不想为model自动添加这些属性，可以在类中通过：



来关闭自动化属性。

 

五：动态计算

一个字段的值，可以通过一个函数来动态计算出来。定义格式如下：

字段名=fields.类型(compute="函数名",store=True/false) #store定义了该动态改变的字段值是否保存到数据库表中

@api.depends(依赖的字段值)#在函数中，通过依赖的字段值计算出上面的字段值
def 函数(self):
    self.字段=计算字段值
示例：




六：关联字段

可以定义一个字段，其值关联其他类的一个字段值。

格式：

字段=fields.类型(related="关联类.字段",store=true/false)


# python  os compared with sys
总结就是，os模块负责程序与操作系统的交互，提供了访问操作系统底层的接口;sys模块负责程序与python解释器的交互，提供了一系列的函数和变量，用于操控python的运行时环境。

os 常用方法

os.remove(‘path/filename’) 删除文件

os.rename(oldname, newname) 重命名文件

os.walk() 生成目录树下的所有文件名

os.chdir('dirname') 改变目录

os.mkdir/makedirs('dirname')创建目录/多层目录

os.rmdir/removedirs('dirname') 删除目录/多层目录

os.listdir('dirname') 列出指定目录的文件

os.getcwd() 取得当前工作目录

os.chmod() 改变目录权限

os.path.basename(‘path/filename’) 去掉目录路径，返回文件名

os.path.dirname(‘path/filename’) 去掉文件名，返回目录路径

os.path.join(path1[,path2[,...]]) 将分离的各部分组合成一个路径名

os.path.split('path') 返回( dirname(), basename())元组

os.path.splitext() 返回 (filename, extension) 元组

os.path.getatime\ctime\mtime 分别返回最近访问、创建、修改时间

os.path.getsize() 返回文件大小

os.path.exists() 是否存在

os.path.isabs() 是否为绝对路径

os.path.isdir() 是否为目录

os.path.isfile() 是否为文件

sys 常用方法

sys.argv 命令行参数List，第一个元素是程序本身路径

sys.modules.keys() 返回所有已经导入的模块列表

sys.exc_info() 获取当前正在处理的异常类,exc_type、exc_value、exc_traceback当前处理的异常详细信息

sys.exit(n) 退出程序，正常退出时exit(0)

sys.hexversion 获取Python解释程序的版本值，16进制格式如：0x020403F0

sys.version 获取Python解释程序的版本信息

sys.maxint 最大的Int值

sys.maxunicode 最大的Unicode值

sys.modules 返回系统导入的模块字段，key是模块名，value是模块

sys.path 返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值

sys.platform 返回操作系统平台名称

sys.stdout 标准输出

sys.stdin 标准输入

sys.stderr 错误输出

sys.exc_clear() 用来清除当前线程所出现的当前的或最近的错误信息

sys.exec_prefix 返回平台独立的python文件安装的位置

sys.byteorder 本地字节规则的指示器，big-endian平台的值是'big',little-endian平台的值是'little'

sys.copyright 记录python版权相关的东西

sys.api_version 解释器的C的API版本



sys.stdin,sys.stdout,sys.stderr
stdin , stdout , 以及stderr 变量包含与标准I/O 流对应的流对象. 如果需要更好地控制输出,而print 不能满足你的要求, 它们就是你所需要的. 你也可以替换它们, 这时候你就可以重定向输出和输入到其它设备( device ), 或者以非标准的方式处理它们
我们常用print和raw_input来进行输入和打印，那么print 和 raw_input是如何与标准输入/输出流建立关系的呢？
其实Python程序的标准输入/输出/出错流定义在sys模块中，分别 为： sys.stdin,sys.stdout, sys.stderr
下列的程序也可以用来输入和输出是一样的：
import sys

sys.stdout.write('HelloWorld!')

print 'Please enter yourname:',
name=sys.stdin.readline()[:-1]
print 'Hi, %s!' % name

那么sys.stdin, sys.stdout, stderr到底是什么呢？我们在Python运行环境中输入以下代码：
import sys
for f in (sys.stdin,sys.stdout, sys.stderr): print f
输出为：
<open file'<stdin>', mode 'r' at 892210>
<open file'<stdout>', mode 'w' at 892270>
<open file'<stderr>', mode 'w at 8922d0>

由此可以看出stdin, stdout, stderr在Python中无非都是文件属性的对象，他们在Python启动时自动与Shell 环境中的标准输入，输出，出错关联。
而Python程序的在Shell中的I/O重定向与本文开始时举的DOS命令的重定向完全相同，其实这种重定向是由Shell来提供的，与Python 本身并无关系。那么我们是否可以在Python程序内部将stdin,stdout,stderr读写操作重定向到一个内部对象呢？答案是肯定的。
Python提供了一个StringIO模块来完成这个设想，比如：
from StringIO import StringIO
import sys
buff =StringIO()

temp =sys.stdout                              #保存标准I/O流
sys.stdout =buff                                #将标准I/O流重定向到buff对象
print 42, 'hello', 0.001

sys.stdout=temp                                #恢复标准I/O流
print buff.getvalue()




# python的构建工具setup.py https://www.cnblogs.com/maociping/p/6633948.html

一、构建工具setup.py的应用场景

      在安装python的相关模块和库时，我们一般使用“pip install  模块名”或者“python setup.py install”，前者是在线安装，会安装该包的相关依赖包；后者是下载源码包然后在本地安装，不会安装该包的相关依赖包。所以在安装普通的python包时，利用pip工具相当简单。但是在如下场景下，使用python setup.py install会更适合需求：

在编写相关系统时，python 如何实现连同依赖包一起打包发布？

      假如我在本机开发一个程序，需要用到python的redis、mysql模块以及自己编写的redis_run.py模块。我怎么实现在服务器上去发布该系统，如何实现依赖模块和自己编写的模块redis_run.py一起打包，实现一键安装呢？同时将自己编写的redis_run.py模块以exe文件格式安装到python的全局执行路径C:\Python27\Scripts下呢？

       在这种应用场景下，pip工具似乎派不上了用场，只能使用python的构建工具setup.py了，使用此构建工具可以实现上述应用场景需求，只需在 setup.py 文件中写明依赖的库和版本，然后到目标机器上使用python setup.py install安装。

二、setup.py介绍
复制代码

 1 from setuptools import setup, find_packages  
 2   
 3 setup(  
 4     name = "test",  
 5     version = "1.0",  
 6     keywords = ("test", "xxx"),  
 7     description = "eds sdk",  
 8     long_description = "eds sdk for python",  
 9     license = "MIT Licence",  
10   
11     url = "http://test.com",  
12     author = "test",  
13     author_email = "test@gmail.com",  
14   
15     packages = find_packages(),  
16     include_package_data = True,  
17     platforms = "any",  
18     install_requires = [],  
19   
20     scripts = [],  
21     entry_points = {  
22         'console_scripts': [  
23             'test = test.help:main'  
24         ]  
25     }  
26 )  

复制代码

    setup.py各参数介绍：

--name 包名称
--version (-V) 包版本
--author 程序的作者
--author_email 程序的作者的邮箱地址
--maintainer 维护者
--maintainer_email 维护者的邮箱地址
--url 程序的官网地址
--license 程序的授权信息
--description 程序的简单描述
--long_description 程序的详细描述
--platforms 程序适用的软件平台列表
--classifiers 程序的所属分类列表
--keywords 程序的关键字列表
--packages 需要处理的包目录（包含__init__.py的文件夹）
--py_modules 需要打包的python文件列表
--download_url 程序的下载地址
--cmdclass
--data_files 打包时需要打包的数据文件，如图片，配置文件等
--scripts 安装时需要执行的脚步列表
--package_dir 告诉setuptools哪些目录下的文件被映射到哪个源码包。一个例子：package_dir = {'': 'lib'}，表示“root package”中的模块都在lib 目录中。
--requires 定义依赖哪些模块
--provides定义可以为哪些模块提供依赖
--find_packages() 对于简单工程来说，手动增加packages参数很容易，刚刚我们用到了这个函数，它默认在和setup.py同一目录下搜索各个含有 __init__.py的包。

                          其实我们可以将包统一放在一个src目录中，另外，这个包内可能还有aaa.txt文件和data数据文件夹。另外，也可以排除一些特定的包

                          find_packages(exclude=["*.tests", "*.tests.*", "tests.*", "tests"])

--install_requires = ["requests"] 需要安装的依赖包
--entry_points 动态发现服务和插件，下面详细讲

     下列entry_points中： console_scripts 指明了命令行工具的名称；在“redis_run = RedisRun.redis_run:main”中，等号前面指明了工具包的名称，等号后面的内容指明了程序的入口地址。

1 entry_points={'console_scripts': [
2         'redis_run = RedisRun.redis_run:main',
3 ]}

      这里可以有多条记录，这样一个项目就可以制作多个命令行工具了，比如：
复制代码

1 setup(
2     entry_points = {
3         'console_scripts': [
4             'foo = demo:test',
5             'bar = demo:test',
6         ]}
7 ）

复制代码

三、setup.py的项目示例代码
复制代码

 1 #!/usr/bin/env python
 2 # coding=utf-8
 3 
 4 from setuptools import setup
 5 
 6 '''
 7 把redis服务打包成C:\Python27\Scripts下的exe文件
 8 '''
 9 
10 setup(
11     name="RedisRun",  #pypi中的名称，pip或者easy_install安装时使用的名称，或生成egg文件的名称
12     version="1.0",
13     author="Andreas Schroeder",
14     author_email="andreas@drqueue.org",
15     description=("This is a service of redis subscripe"),
16     license="GPLv3",
17     keywords="redis subscripe",
18     url="https://ssl.xxx.org/redmine/projects/RedisRun",
19     packages=['RedisRun'],  # 需要打包的目录列表
20 
21     # 需要安装的依赖
22     install_requires=[
23         'redis>=2.10.5',
24         'setuptools>=16.0',
25     ],
26 
27     # 添加这个选项，在windows下Python目录的scripts下生成exe文件
28     # 注意：模块与函数之间是冒号:
29     entry_points={'console_scripts': [
30         'redis_run = RedisRun.redis_run:main',
31     ]},
32 
33     # long_description=read('README.md'),
34     classifiers=[  # 程序的所属分类列表
35         "Development Status :: 3 - Alpha",
36         "Topic :: Utilities",
37         "License :: OSI Approved :: GNU General Public License (GPL)",
38     ],
39     # 此项需要，否则卸载时报windows error
40     zip_safe=False
41 )

复制代码

四、修改后的项目代码（此时RedisRun模块是DrQueue模块的子模块，这是因为要导入某些公用的模块）
复制代码

 1 #!/usr/bin/env python
 2 # coding=utf-8
 3 
 4 from setuptools import setup
 5 
 6 '''
 7 把redis服务打包成C:\Python27\Scripts下的exe文件
 8 '''
 9 
10 setup(
11     name="RedisRun",  #pypi中的名称，pip或者easy_install安装时使用的名称
12     version="1.0",
13     author="Andreas Schroeder",
14     author_email="andreas@drqueue.org",
15     description=("This is a service of redis subscripe"),
16     license="GPLv3",
17     keywords="redis subscripe",
18     url="https://ssl.xxx.org/redmine/projects/RedisRun",
19     packages=['DrQueue'],  # 需要打包的目录列表
20 
21     # 需要安装的依赖
22     install_requires=[
23         'redis>=2.10.5',
24     ],
25 
26     # 添加这个选项，在windows下Python目录的scripts下生成exe文件
27     # 注意：模块与函数之间是冒号:
28     entry_points={'console_scripts': [
29         'redis_run = DrQueue.RedisRun.redis_run:main',
30     ]},
31 
32     # long_description=read('README.md'),
33     classifiers=[  # 程序的所属分类列表
34         "Development Status :: 3 - Alpha",
35         "Topic :: Utilities",
36         "License :: OSI Approved :: GNU General Public License (GPL)",
37     ],
38     # 此项需要，否则卸载时报windows error
39     zip_safe=False
40 )

复制代码

       此时项目的目录结构为：




# Python 错误和异常小结   
1.l异常类

    Python是面向对象语言，所以程序抛出的异常也是类。常见的Python异常有以下几个，大家只要大致扫一眼，有个映像，等到编程的时候，相信大家肯定会不只一次跟他们照面（除非你不用Python了）。

   
异常 	描述
NameError 	尝试访问一个没有申明的变量
ZeroDivisionError 	除数为0
SyntaxError 	语法错误
IndexError 	索引超出序列范围
KeyError 	请求一个不存在的字典关键字
IOError 	输入输出错误（比如你要读的文件不存在）
AttributeError 	尝试访问未知的对象属性
ValueError 	传给函数的参数类型不正确，比如给int()函数传入字符串形
2.捕获异常

    Python完整的捕获异常的语句有点像：

[html] view plain copy

    try:  
        try_suite  
    except Exception1,Exception2,...,Argument:  
        exception_suite  
    ......   #other exception block  
    else:  
        no_exceptions_detected_suite  
    finally:  
        always_execute_suite  

    额...是不是很复杂？当然，当我们要捕获异常的时候，并不是必须要按照上面那种格式完全写下来，我们可以丢掉else语句，或者finally语句；甚至不要exception语句，而保留finally语句。额，晕了？好吧，下面，我们就来一一说明啦。

2.1.try...except...语句

    try_suite不消我说大家也知道，是我们需要进行捕获异常的代码。而except语句是关键，我们try捕获了代码段try_suite里的异常后，将交给except来处理。

    try...except语句最简单的形式如下：
[python] view plain copy

    try:  
        try_suite  
    except:  
        exception block  

    上面except子句不跟任何异常和异常参数，所以无论try捕获了任何异常，都将交给except子句的exception block来处理。如果我们要处理特定的异常，比如说，我们只想处理除零异常，如果其他异常出现，就让其抛出不做处理，该怎么办呢？这个时候，我们就要给except子句传入异常参数啦！那个ExceptionN就是我们要给except子句的异常类（请参考异常类那个表格），表示如果捕获到这类异常，就交给这个except子句来处理。比如：

[python] view plain copy

    try:  
        try_suite  
    except Exception:  
        exception block  

    举个例子：

[python] view plain copy

    >>> try:  
    ...     res = 2/0  
    ... except ZeroDivisionError:  
    ...     print "Error:Divisor must not be zero!"  
    ...   
    Error:Divisor must not be zero!  

    看，我们真的捕获到了ZeroDivisionError异常！那如果我想捕获并处理多个异常怎么办呢？有两种办法，一种是给一个except子句传入多个异常类参数，另外一种是写多个except子句，每个子句都传入你想要处理的异常类参数。甚至，这两种用法可以混搭呢！下面我就来举个例子。

[python] view plain copy

    try:  
        floatnum = float(raw_input("Please input a float:"))  
        intnum = int(floatnum)  
        print 100/intnum  
    except ZeroDivisionError:  
        print "Error:you must input a float num which is large or equal then 1!"  
    except ValueError:  
        print "Error:you must input a float num!"  
      
    [root@Cherish tmp]# python test.py   
    Please input a float:fjia  
    Error:you must input a float num!  
    [root@Cherish tmp]# python test.py   
    Please input a float:0.9999  
    Error:you must input a float num which is large or equal then 1!  
    [root@Cherish tmp]# python test.py   
    Please input a float:25.091  
    4  

    上面的例子大家一看都懂，就不再解释了。只要大家明白，我们的except可以处理一种异常，多种异常，甚至所有异常就可以了。

    大家可能注意到了，我们还没解释except子句后面那个Argument是什么东西？别着急，听我一一道来。这个Argument其实是一个异常类的实例（别告诉我你不知到什么是实例），包含了来自异常代码的诊断信息。也就是说，如果你捕获了一个异常，你就可以通过这个异常类的实例来获取更多的关于这个异常的信息。例如：

[python] view plain copy

    >>> try:  
    ...     1/0  
    ... except ZeroDivisionError,reason:  
    ...     pass  
    ...   
    >>> type(reason)  
    <type 'exceptions.ZeroDivisionError'>  
    >>> print reason  
    integer division or modulo by zero  
    >>> reason  
    ZeroDivisionError('integer division or modulo by zero',)  
    >>> reason.__class__  
    <type 'exceptions.ZeroDivisionError'>  
    >>> reason.__class__.__doc__  
    'Second argument to a division or modulo operation was zero.'  
    >>> reason.__class__.__name__  
    'ZeroDivisionError'  

    上面这个例子，我们捕获了除零异常，但是什么都没做。那个reason就是异常类ZeroDivisionError的实例，通过type就可以看出。

2.2try ... except...else语句

    现在我们来说说这个else语句。Python中有很多特殊的else用法，比如用于条件和循环。放到try语句中，其作用其实也差不多：就是当没有检测到异常的时候，则执行else语句。举个例子大家可能更明白些：

[python] view plain copy

    >>> import syslog  
    >>> try:  
    ...     f = open("/root/test.py")  
    ... except IOError,e:  
    ...     syslog.syslog(syslog.LOG_ERR,"%s"%e)  
    ... else:  
    ...     syslog.syslog(syslog.LOG_INFO,"no exception caught\n")  
    ...   
    >>> f.close()  

2.3 finally子句

    finally子句是无论是否检测到异常，都会执行的一段代码。我们可以丢掉except子句和else子句，单独使用try...finally，也可以配合except等使用。

例如2.2的例子，如果出现其他异常，无法捕获，程序异常退出，那么文件 f 就没有被正常关闭。这不是我们所希望看到的结果，但是如果我们把f.close语句放到finally语句中，无论是否有异常，都会正常关闭这个文件，岂不是很 妙

[python] view plain copy

    >>> import syslog  
    >>> try:  
    ...     f = open("/root/test.py")  
    ... except IOError,e:  
    ...     syslog.syslog(syslog.LOG_ERR,"%s"%e)  
    ... else:  
    ...     syslog.syslog(syslog.LOG_INFO,"no exception caught\n")  
    ... finally:   
    >>>     f.close()  

    大家看到了没，我们上面那个例子竟然用到了try,except,else,finally这四个子句！:-)，是不是很有趣？到现在，你就基本上已经学会了如何在Python中捕获常规异常并处理之。

3.两个特殊的处理异常的简便方法
3.1断言（assert）

    什么是断言，先看语法：

[python] view plain copy

    assert expression[,reason]  

    其中assert是断言的关键字。执行该语句的时候，先判断表达式expression，如果表达式为真，则什么都不做；如果表达式不为真，则抛出异常。reason跟我们之前谈到的异常类的实例一样。不懂？没关系，举例子！最实在！

[python] view plain copy

    >>> assert len('love') == len('like')  
    >>> assert 1==1  
    >>> assert 1==2,"1 is not equal 2!"  
    Traceback (most recent call last):  
      File "<stdin>", line 1, in <module>  
    AssertionError: 1 is not equal 2!  

我们可以看到，如果assert后面的表达式为真，则什么都不做，如果不为真，就会抛出AssertionErro异常，而且我们传进去的字符串会作为异常类的实例的具体信息存在。其实，assert异常也可以被try块捕获：

[python] view plain copy

    >>> try:  
    ...     assert 1 == 2 , "1 is not equal 2!"  
    ... except AssertionError,reason:  
    ...     print "%s:%s"%(reason.__class__.__name__,reason)  
    ...   
    AssertionError:1 is not equal 2!  
    >>> type(reason)  
    <type 'exceptions.AssertionError'>  


3.2.上下文管理（with语句）

   如果你使用try,except,finally代码仅仅是为了保证共享资源（如文件，数据）的唯一分配，并在任务结束后释放它，那么你就有福了！这个with语句可以让你从try,except,finally中解放出来！语法如下：

[python] view plain copy

    with context_expr [as var]:  
        with_suite  

    是不是不明白？很正常，举个例子来！

[python] view plain copy

    >>> with open('/root/test.py') as f:  
    ...     for line in f:  
    ...         print line  

    上面这几行代码干了什么？

    （1）打开文件/root/test.py

    （2）将文件对象赋值给  f

    （3）将文件所有行输出

     （4）无论代码中是否出现异常，Python都会为我们关闭这个文件，我们不需要关心这些细节。

    这下，是不是明白了，使用with语句来使用这些共享资源，我们不用担心会因为某种原因而没有释放他。但并不是所有的对象都可以使用with语句，只有支持上下文管理协议（context management protocol）的对象才可以，那哪些对象支持该协议呢？如下表：

file

decimal.Context

thread.LockType

threading.Lock

threading.RLock

threading.Condition

threading.Semaphore

threading.BoundedSemaphore

    至于什么是上下文管理协议，如果你不只关心怎么用with,以及哪些对象可以使用with，那么我们就不比太关心这个问题：）
4.抛出异常(raise)

    如果我们想要在自己编写的程序中主动抛出异常，该怎么办呢？raise语句可以帮助我们达到目的。其基本语法如下：

[python] view plain copy

    raise [SomeException [, args [,traceback]]  

    第一个参数，SomeException必须是一个异常类，或异常类的实例

    第二个参数是传递给SomeException的参数，必须是一个元组。这个参数用来传递关于这个异常的有用信息。

    第三个参数traceback很少用，主要是用来提供一个跟中记录对象（traceback）

    下面我们就来举几个例子。

[python] view plain copy

    >>> raise NameError  
    Traceback (most recent call last):  
      File "<stdin>", line 1, in <module>  
    NameError  
    >>> raise NameError()  #异常类的实例  
    Traceback (most recent call last):  
      File "<stdin>", line 1, in <module>  
    NameError  
    >>> raise NameError,("There is a name error","in test.py")  
    Traceback (most recent call last):  
      File "<stdin>", line 1, in <module>  
    >>> raise NameError("There is a name error","in test.py")  #注意跟上面一个例子的区别  
    Traceback (most recent call last):  
      File "<stdin>", line 1, in <module>  
    NameError: ('There is a name error', 'in test.py')  
    >>> raise NameError,NameError("There is a name error","in test.py")  #注意跟上面一个例子的区别  
    Traceback (most recent call last):  
      File "<stdin>", line 1, in <module>  
    NameError: ('There is a name error', 'in test.py')  

    其实，我们最常用的还是，只传入第一个参数用来指出异常类型，最多再传入一个元组，用来给出说明信息。如上面第三个例子。

5.异常和sys模块

    另一种获取异常信息的途径是通过sys模块中的exc_info()函数。该函数回返回一个三元组:(异常类，异常类的实例，跟中记录对象)

[python] view plain copy

    >>> try:  
    ...     1/0  
    ... except:  
    ...     import sys  
    ...     tuple = sys.exc_info()  
    ...   
    >>> print tuple  
    (<type 'exceptions.ZeroDivisionError'>, ZeroDivisionError('integer division or modulo by zero',), <traceback object at 0x7f538a318b48>)  
    >>> for i in tuple:  
    ...     print i  
    ...   
    <type 'exceptions.ZeroDivisionError'> #异常类      
    integer division or modulo by zero #异常类的实例  
    <traceback object at 0x7f538a318b48> #跟踪记录对象 


# python class __new__(cls,...) and __init__(self,...)

一、__init__ 方法是什么？

使用Python写过面向对象的代码的同学，可能对 __init__ 方法已经非常熟悉了，__init__ 方法通常用在初始化一个类实例的时候。例如：
Python
# -*- coding: utf-8 -*-

class Person(object):
    """Silly Person"""

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return '<Person: %s(%s)>' % (self.name, self.age)

if __name__ == '__main__':
    piglei = Person('piglei', 24)
    print piglei
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
	
# -*- coding: utf-8 -*-
 
class Person(object):
    """Silly Person"""
 
    def __init__(self, name, age):
        self.name = name
        self.age = age
 
    def __str__(self):
        return '<Person: %s(%s)>' % (self.name, self.age)
 
if __name__ == '__main__':
    piglei = Person('piglei', 24)
    print piglei

这样便是__init__最普通的用法了。但__init__其实不是实例化一个类的时候第一个被调用 的方法。当使用 Persion(name, age) 这样的表达式来实例化一个类时，最先被调用的方法 其实是 __new__ 方法。
二、__new__ 方法是什么？

__new__方法接受的参数虽然也是和__init__一样，但__init__是在类实例创建之后调用，而 __new__方法正是创建这个类实例的方法。
Python
# -*- coding: utf-8 -*-

class Person(object):
    """Silly Person"""

    def __new__(cls, name, age):
        print '__new__ called.'
        return super(Person, cls).__new__(cls, name, age)

    def __init__(self, name, age):
        print '__init__ called.'
        self.name = name
        self.age = age

    def __str__(self):
        return '<Person: %s(%s)>' % (self.name, self.age)

if __name__ == '__main__':
    piglei = Person('piglei', 24)
    print piglei
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
	
# -*- coding: utf-8 -*-
 
class Person(object):
    """Silly Person"""
 
    def __new__(cls, name, age):
        print '__new__ called.'
        return super(Person, cls).__new__(cls, name, age)
 
    def __init__(self, name, age):
        print '__init__ called.'
        self.name = name
        self.age = age
 
    def __str__(self):
        return '<Person: %s(%s)>' % (self.name, self.age)
 
if __name__ == '__main__':
    piglei = Person('piglei', 24)
    print piglei

执行结果：
Python
piglei@macbook-pro:blog$ python new_and_init.py
__new__ called.
__init__ called.
<Person: piglei(24)>
1
2
3
4
	
piglei@macbook-pro:blog$ python new_and_init.py
__new__ called.
__init__ called.
<Person: piglei(24)>

通过运行这段代码，我们可以看到，__new__方法的调用是发生在__init__之前的。其实当 你实例化一个类的时候，具体的执行逻辑是这样的：

1.p = Person(name, age)
2.首先执行使用name和age参数来执行Person类的__new__方法，这个__new__方法会 返回Person类的一个实例（通常情况下是使用 super(Persion, cls).__new__(cls, … …) 这样的方式），
3.然后利用这个实例来调用类的__init__方法，上一步里面__new__产生的实例也就是 __init__里面的的 self
所以，__init__ 和 __new__ 最主要的区别在于：
1.__init__ 通常用于初始化一个新实例，控制这个初始化的过程，比如添加一些属性， 做一些额外的操作，发生在类实例被创建完以后。它是实例级别的方法。
2.__new__ 通常用于控制生成一个新实例的过程。它是类级别的方法。
但是说了这么多，__new__最通常的用法是什么呢，我们什么时候需要__new__？
三、__new__ 的作用

依照Python官方文档的说法，__new__方法主要是当你继承一些不可变的class时(比如int, str, tuple)， 提供给你一个自定义这些类的实例化过程的途径。还有就是实现自定义的metaclass。
首先我们来看一下第一个功能，具体我们可以用int来作为一个例子：
假如我们需要一个永远都是正数的整数类型，通过集成int，我们可能会写出这样的代码。
class PositiveInteger(int):
    def __init__(self, value):
        super(PositiveInteger, self).__init__(self, abs(value))

i = PositiveInteger(-3)
print i

	
class PositiveInteger(int):
    def __init__(self, value):
        super(PositiveInteger, self).__init__(self, abs(value))
 
i = PositiveInteger(-3)
print i

但运行后会发现，结果根本不是我们想的那样，我们任然得到了-3。这是因为对于int这种 不可变的对象，我们只有重载它的__new__方法才能起到自定义的作用。
这是修改后的代码：
class PositiveInteger(int):
    def __new__(cls, value):
        return super(PositiveInteger, cls).__new__(cls, abs(value))

i = PositiveInteger(-3)
print i
1
2
3
4
5
6
	
class PositiveInteger(int):
    def __new__(cls, value):
        return super(PositiveInteger, cls).__new__(cls, abs(value))
 
i = PositiveInteger(-3)
print i

通过重载__new__方法，我们实现了需要的功能。
另外一个作用，关于自定义metaclass。其实我最早接触__new__的时候，就是因为需要自定义 metaclass，但鉴于篇幅原因，我们下次再来讲python中的metaclass和__new__的关系。
四、用__new__来实现单例

事实上，当我们理解了__new__方法后，我们还可以利用它来做一些其他有趣的事情，比如实现 设计模式中的 单例模式(singleton) 。
因为类每一次实例化后产生的过程都是通过__new__来控制的，所以通过重载__new__方法，我们 可以很简单的实现单例模式。
class Singleton(object):
    def __new__(cls):
        # 关键在于这，每一次实例化的时候，我们都只会返回这同一个instance对象
        if not hasattr(cls, 'instance'):
            cls.instance = super(Singleton, cls).__new__(cls)
        return cls.instance

obj1 = Singleton()
obj2 = Singleton()

obj1.attr1 = 'value1'
print obj1.attr1, obj2.attr1
print obj1 is obj2
1
2
3
4
5
6
7
8
9
10
11
12
13
	
class Singleton(object):
    def __new__(cls):
        # 关键在于这，每一次实例化的时候，我们都只会返回这同一个instance对象
        if not hasattr(cls, 'instance'):
            cls.instance = super(Singleton, cls).__new__(cls)
        return cls.instance
 
obj1 = Singleton()
obj2 = Singleton()
 
obj1.attr1 = 'value1'
print obj1.attr1, obj2.attr1
print obj1 is obj2

输出结果：
value1 value1
True
1
2
	
value1 value1
True

可以看到obj1和obj2是同一个实例。

# odoo web controller, routes
/home/vagrant/share/odoo-dev/addons/web/controllers/main.py






