Python3单行定义多个变量或赋值
http://blog.csdn.net/wuxintdrh/article/details/53405529
你甚至可以在一行内将多个值赋值给多个变量

>>> a , b = 45, 54
>>> a
45
>>> b
54

    1
    2
    3
    4
    5

这个技巧用来交换两个数的值非常方便

>>> a, b = b , a
>>> a
54
>>> b
45

    1
    2
    3
    4
    5

要明白这是怎么工作的，你需要学习元组（tuple）这个数据类型。我们是用逗号创建元组。在赋值语句的右边我们创建了一个元组，我们称这为元组封装（tuple packing），赋值语句的左边我们则做的是元组拆封 （tuple unpacking）。
下面是另一个元组拆封的例子：

>>> data = ("shiyanlou", "China", "Python")
>>> name, country, language = data
>>> name
'shiyanlou'
>>> country
'China'
>>> language
'Python'
