Python技巧：元类（Metaclasses）和利用Type构建的动态类（Dynamic Classes）

`metaclass`和`type`关键字在Python代码中较少被使用（也正因如此，它们的作用也没有很好的被理解）。在这篇文章中，我们将探究`type()`的类型（types）和跟`metaclasses`相关的`type`的用法。


这是我的类型么？
首先来看`type()`的第一个广为人知的用法，即查看一个对象的类型。Python初学者常常会对此有这样的疑问：“我认为Python中是没有类型的啊！”可事实正相反，Python中的一切都有类型（即使是类型关键字‘types’本身！），这是因为Python中的一切都是对象（object）。下面来看几个例子：

[python] view plain copy

    >>> type(1)  
    <class 'int'>  
    >>> type('foo')  
    <class 'str'>  
    >>> type(3.0)  
    <class 'float'>  
    >>> type(float)  
    <class 'type'>  


`type`的类型

一切看起来都是那么理所当然，直到我们查看float的类型，`<class 'type'>`?这是怎么回事？继续往下看：

[python] view plain copy

    >>> class Foo(object):  
    ...     pass  
    ...  
    >>> type(Foo)  
    <class 'type'>  


啊，我们又看到了`<class 'type'>`。很显然，所有类自身的类型都是`type`（无论是内建类型或是用户定义类型的类）。那么`type`的`type`又是什么呢？

[python] view plain copy

    >>> type(type)  
    <class 'type'>  


好吧，测试到此为止。`type`是所有类型的类型，包括它自身。事实上，`type`是一个元类`metaclass`，也就是“一个用来构建类的东西”。诸如`list()`这样的类，是用来构建类的实例(instances)，例如`my_list=list()`。而元类则以同样的方式来构建类型，例如以下代码：

[python] view plain copy

    class Foo(object):  
        pass  


创建自己的元类

与常规类类似，元类也可以由用户自定义。具体用法是将目标类的`__metaclass__`属性设置为自定义的`metaclass`。`metaclass`必须是可调用(callable)的，并且返回一个`type`。通常，用户可以使用一个函数来设置`__metaclass__`属性，这个函数是`type`的另一种用法：利用三个参数创建一个新类。


`type`的另一面

如上文所述，`type`有着另一种完全不同的用法。这种用法由传入三个参数：type(name, bases, dict)能够创建一个新的类型。以下是创建一个新类的通常做法：

[python] view plain copy

    class Foo(object):  
        pass  


而我们能够通过以下的代码达到同样的效果：

[python] view plain copy

    Foo = type('Foo', (), {})  


目前Foo是一个名为“Foo”的类的引用，这个名为“Foo”的类以object为基类（通过type创建的类，如果没有特别指定其基类，将会默认创建新类型的类）。


这么做看起来非常棒，但如果我们需要向Foo中添加成员函数该怎么办呢？这很容易通过设置Foo的类属性办到：

[python] view plain copy

    def always_false(self):  
        return False  
      
    Foo.always_false = always_false  


上面的代码也能够写成如下形式：

[python] view plain copy

    Foo = type('Foo', (), {'always_false': always_false})  


bases参数是Foo的基类列表，在上例中我们留空了，同时可以用type从Foo类创建一个新类：

[python] view plain copy

    FooBar = type('FooBar', (Foo), {})  


这么做有什么用？

介绍完`type`和`metaclasses`之后问题随之而来：“我该什么时候使用它们呢？”答案是显然的，在日常的工作中我们并不一定会经常使用到它们。但当我们想要动态地创建类时，利用type是一个很合适的解决方案。让我们来看个例子：

sandman是我写的一个库，这个库能够自动地为已有数据库生成REST API以及基于网页的管理员接口（不需要配置模板代码）。大部分工作由SQLAlchemy这样一个对象关系映射框架完成。

利用SQLAlchemy注册一个数据库表只有一种方法：创建一个描述这个数据表的模板类（跟Django中得models不同）。为了SQLAlchemy能够识别一个表，一个对应的类必须通过某种方式生成。因为sandman没有关于数据库结构的任何先验知识，所以不能通过提前准备模板类的方式来注册数据表。相反，需要通过探查数据库动态的生成模板类。是不是听起来很熟悉？无论何时当你想动态地创建一个新类，type都将是你唯一正确地选择。

以下是sandman中的相关实现代码：

[python] view plain copy

    if not current_app.endpoint_classes:  
        db.metadata.reflect(bind=db.engine)  
        for name in db.metadata.tables():  
            cls = type(str(name), (sandman_model, db.Model),  
                    {'__tablename__': name})  
            register(cls)  


正如你所见，如果用户没有手动的为一个表创建模板类，代码将动态的创建，并且利用表名设置`__tablename__`属性(SQLAlchemy将利用这个属性匹配表与相应的模板类)。


总结

在这篇文章中，我们讨论了`type`、`metaclasses`的两种用法和使用时机。尽管`metaclasses`是一个多多少少会让人感到困惑的概念，希望读者现在对此能有一个清晰的认识，并在未来的学习中能够使用。
