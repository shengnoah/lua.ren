---
layout: post
title: 延迟加载模式（lazy evaluation） 
tags: [lua文章]
categories: [lua文章]
---
延迟加载模式（lazy evaluation）  
代码参考引用自：<https://github.com/faif/python-
patterns/blob/master/lazy_evaluation.py>

#### 代码

    
    
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
    
    21
    
    22
    
    23
    
    24
    
    25
    
    26
    
    27
    
    28
    
    29
    
    30
    
    31
    
    32
    
    33
    
    34
    
    35
    
    36
    
    37
    
    38
    
    39
    
    40
    
    41
    
    42
    
    43
    
    44
    
    45
    
    46
    
    47
    
    48
    
    49
    
    50
    
    51
    
    52
    
    53
    
    54
    
    55
    
    56
    
    57
    
    58
    
    59
    
    60

|

    
    
    # -*- coding: utf-8 -*-
    
    """
    
    Lazily-evaluated property pattern in Python.
    
    https://en.wikipedia.org/wiki/Lazy_evaluation
    
    References:
    
    bottle
    
    https://github.com/bottlepy/bottle/blob/cafc15419cbb4a6cb748e6ecdccf92893bb25ce5/bottle.py#L270
    
    django
    
    https://github.com/django/django/blob/ffd18732f3ee9e6f0374aff9ccf350d85187fac2/django/utils/functional.py#L19
    
    pip
    
    https://github.com/pypa/pip/blob/cb75cca785629e15efb46c35903827b3eae13481/pip/utils/__init__.py#L821
    
    pyramimd
    
    https://github.com/Pylons/pyramid/blob/7909e9503cdfc6f6e84d2c7ace1d3c03ca1d8b73/pyramid/decorator.py#L4
    
    werkzeug
    
    https://github.com/pallets/werkzeug/blob/5a2bf35441006d832ab1ed5a31963cbc366c99ac/werkzeug/utils.py#L35
    
    """
    
    class (object):
    
        def __init__(self, function):
    
            self.function = function
    
            functools.update_wrapper(self, function)
    
        def __get__(self, obj, type_):
    
            if obj is None:
    
                return self
    
            val = self.function(obj)
    
            obj.__dict__[self.function.__name__] = val
    
            return val
    
    class Person(object):
    
        def __init__(self, name, occupation):
    
            self.name = name
    
            self.occupation = occupation
    
        def relatives(self):
    
            # Get all relatives, let's assume that it costs much time.
    
            relatives = "Many relatives."
    
            return relatives
    
    def main():
    
        Jhon = Person('Jhon', 'Coder')
    
        print("Name: {0}    Occupation: {1}".format(Jhon.name, Jhon.occupation))
    
        print("Before we access `relatives`:")
    
        print(Jhon.__dict__)
    
        print("Jhon's relatives: {0}".format(Jhon.relatives))
    
        print("After we've accessed `relatives`:")
    
        print(Jhon.__dict__)
    
    ### OUTPUT ###
    
    # Name: Jhon    Occupation: Coder
    
    # Before we access `relatives`:
    
    # {'name': 'Jhon', 'occupation': 'Coder'}
    
    # Jhon's relatives: Many relatives.
    
    # After we've accessed `relatives`:
    
    # {'relatives': 'Many relatives.', 'name': 'Jhon', 'occupation': 'Coder'}  
  
---|---  
  
#### 分析

装饰器类 lazy_property 中的 `__get__()`方法，是为该类实现为了描述符（descriptor）。`__get__()`
方法是在访问类或者类实例的属性时，类实例的 `__dict__` 中不存在被访问属性，类的 `__dict__` 中也不存在被访问属性的情况下，会调用
`__get__()` 方法获取被访问属性，即例子代码中的 relatives。在执行语句 `print("John's relatives:
{0}".format(John.relatives))` 时，实例 Jhon 的 `__dict__` 中还没有属性 relatives，所以装饰器类的
`__get__` 方法被调用，为实例 Jhon 设置了 revatives 属性，达到延迟加载的效果。

本例说明的情况为，假设类 Person 的实例的 relatives 属性加载特别缓慢，在实例化 Person
类的时候暂时不要初始化该属性，找其他合适的时机再延迟加载。