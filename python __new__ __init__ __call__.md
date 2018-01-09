详解Python中的__new__、__init__、__call__三个特殊方法

__new__： 对象的创建，是一个静态方法，第一个参数是cls。(想想也是，不可能是self，对象还没创建，哪来的self)
__init__ ： 对象的初始化， 是一个实例方法，第一个参数是self。
__call__ ： 对象可call，注意不是类，是对象。

先有创建，才有初始化。即先__new__，而后__init__。
上面说的不好理解，看例子。

1.对于__new__
?
1
2
3
4
5
6
7
8
	
class Bar(object): 
  pass
  
class Foo(object): 
  def __new__(cls, *args, **kwargs): 
    return Bar() 
  
print Foo()

 可以看到，输出来是一个Bar对象。

__new__方法在类定义中不是必须写的，如果没定义，默认会调用object.__new__去创建一个对象。如果定义了，就是override,可以custom创建对象的行为。
聪明的读者可能想到，既然__new__可以custom对象的创建，那我在这里做一下手脚，每次创建对象都返回同一个，那不就是单例模式了吗？没错，就是这样。可以观摩《飘逸的python - 单例模式乱弹》
定义单例模式时，因为自定义的__new__重载了父类的__new__，所以要自己显式调用父类的__new__，即object.__new__(cls, *args, **kwargs)，或者用super()。，不然就不是extend原来的实例了，而是替换原来的实例。

2.对于__init__

使用Python写过面向对象的代码的同学，可能对 __init__ 方法已经非常熟悉了，__init__ 方法通常用在初始化一个类实例的时候。例如：
?
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

3.对于__call__
对象通过提供__call__(slef, [,*args [,**kwargs]])方法可以模拟函数的行为，如果一个对象x提供了该方法，就可以像函数一样使用它，也就是说x(arg1, arg2...) 等同于调用x.__call__(self, arg1, arg2) 。模拟函数的对象可以用于创建防函数(functor) 或代理(proxy).
?
1
2
3
4
5
6
	
class Foo(object): 
  def __call__(self): 
    pass
  
f = Foo()#类Foo可call 
f()#对象f可call 

总结，在Python中，类的行为就是这样，__new__、__init__、__call__等方法不是必须写的，会默认调用，如果自己定义了，就是override,可以custom。既然override了，通常也会显式调用进行补偿以达到extend的目的。
这也是为什么会出现"明明定义def _init__(self, *args, **kwargs)，对象怎么不进行初始化"这种看起来诡异的行为。（注，这里_init__少写了个下划线，因为__init__不是必须写的，所以这里不会报错，而是当做一个新的方法_init__）
