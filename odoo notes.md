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



