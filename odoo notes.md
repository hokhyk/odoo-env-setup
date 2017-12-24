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

