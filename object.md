# [what does 'super' do in python](http://stackoverflow.com/questions/222877/what-does-super-do-in-python)

下面两个的区别是？

```python
class Child(SomeBaseClass):
    def __init__(self):
		super(Child, self).__init__()
```
和
```python
class Child(SomeBaseClass):
    def __init__(self):
		SomeBaseClass.__init__(self)
```

我在单个继承中已经看到`super` 被用的很多了。我能知道为什么要在多重继承的时候用它，但是还是不清楚在这种情况用它的好处。

---

180票的回答：（John Millikin)

在单一继承用`super`的好处很小--只是你不再需要硬编码基类名字到方法里面去了

然后，在多重继承里，不用`super()`几乎是不可能的。这包括常见的习语，像是mixins，interface,abstract classes等，
这能让你的代码在之后延伸。如果以后有人想写一个拓展`Child` 和 mixin的类，他们的代码不会很好的工作。

---
75票的回答：

###### 区别是什么？

`SomeBaseClass.__init__(self)`意思是调用`SomeBaseClass`的`__init__`方法
然后，`super(Child,self).__init__()`意思是从`Child`类的MRO的父类中调用一个绑定方法`__init__`
如果实例是Child的子类，有可能在方法解释顺序中的下一个父类是不一样的？？？

###### 向前兼容间接 ？？ （Indirection with Forward Compatibility)

这能给你什么？对于单重继承，问题中给出的例子几乎等同于静态分析。然而使用`super` 提供了具有向前兼容性的间接层
向前兼容对于经验丰富的开发者来说是很重要的。你希望你的代码在做出一些细微的改动之后还能工作。当您查看修订历史记录时，您希望准确地查看何时更改了哪些内容。

你可能先从单重继承开始，但是当你增加另外的基类，你只需要改变基类的顺序（change the line with the bases）
（if the bases change in a class you inherit from）如果类继承关系变了（比如增加了一个mixin)，其实你就没做什么改变。
尤其在python2中，要想给super正确的方法参数是很难的。如果你知道你在单重继承下正确的使用`super`，这样是的调试就容易一点了

###### 依赖注入 Dependency Injection

其他人可以使用你的代码然后插入一些父类到方法解释中(method resolution):
```python
class SomeBaseClass(object):
    def __init__(self):
		print('SomeBaseClass.__init__(self) called')
			
class UnsuperChild(SomeBaseClass):
	def __init__(self):
		print('UnsuperChild.__init__(self) called')
		SomeBaseClass.__init__(self)
							
class SuperChild(SomeBaseClass):
	def __init__(self):
		print('SuperChild.__init__(self) called')
		super(SuperChild, self).__init__()
```

现在你增加其他类，然后在Foo和Bar之间插入一个类

```python
class InjectMe(SomeBaseClass):
    def __init__(self):
		print('InjectMe.__init__(self) called')
		super(InjectMe, self).__init__()
				
class UnsuperInjector(UnsuperChild, InjectMe): pass
					
class SuperInjector(SuperChild, InjectMe): pass
```

使用un-super子类未能注入依赖，因为你是用的子类在自己执行打印后调用的是硬编码方法

```python
>>> o = UnsuperInjector()
UnsuperChild.__init__(self) called
SomeBaseClass.__init__(self) called
```

然而使用`super`的子类能正确的依赖注入

```python
>>> o2 = SuperInjector()
SuperChild.__init__(self) called
InjectMe.__init__(self) called
SomeBaseClass.__init__(self) called
```
(我：因为super按照MRO来寻找next类的，不是就是去找父类SomeBaseClass,
因为SuperInjector的MRO是自身> UnsuperChild > InjectMe > SomeBaseClass > object)

###### 结论
一直使用`super`来引用父类就好了

你想要引用的父类是MRO下一个类，而不是你看到的继承的关系

不使用`super` 回让你代码的使用者多了很多不必要的限制
