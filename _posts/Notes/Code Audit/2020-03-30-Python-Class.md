---
layout: post
title: Python Class
data: 2020-03-30
categories: Notes
tags: 
 - Python
description: 
excerpt_separator: ""
---

# 类的基础

1. 使用class关键字定义一个继承自object的类

2. 创建类属性，使用双下划线开头的变量作为私有变量

3. 使用\_\_init\_\_()作为构造方法，类方法的第一个参数必为"self"

4. 创建一个类的实例，并设置实例属性

    ```python
    class Student(object):
        name = "student"
        __sex = "male"
        
        def __init__(self, name, sex):
            self.name = name
            self.__sex = sex
            
    student_1 = Student('Asinose', 'male')
    student_1.age = 18
    ```

# 一些会用到的函数

1. 返回对象的类

   ```python
   type(object)
   # 参数为类名
   ```

2. 返回一个新类

   ```python
   type(name, bases, dict)
   # 参数分别为类名、包含基类的元组、包含属性地字典
   ```

3. 判断对象是否是类的实例，考虑继承关系

   ```python
   isinstance(object, classinfo)
   # 参数分别为对象名、类名
   ```

3. 返回对象的所有方法和属性

    ```python
    dir([object])
    # 参数为对象名
    # 无参时返回当前模块的属性、方法
    ```

5. 查看、判断、设置对象的属性

    ```python
    # 返回对象地属性值
    getattr(object, name[, default])
    # 参数分别为对象名、属性名、属性不存在时返回地默认值
    
    # 判断对象的属性是否存在
    hasattr(object, name)
    # 参数分别为对象名、属性名
    
    # 设置对象的属性
    setattr(object, name, value)
    # 参数分别为对象名、属性名、属性值
    ```

# 特殊属性和方法

```python
object.__dict__
# 一个字典或其他类型的映射对象，用于存储对象的（可写）属性
```

```python
instance.__class__
# 类实例所属的类
```

```python
class.__bases__
# 由类对象的基类所组成的元组
```

```python
definition.__name__
# 类、函数、方法、描述器或生成器实例的名称
```

```python
definition.__qualname__
# 类、函数、方法、描述器或生成器实例的限定名称
```

```python
class.__mro__
# 此属性是由类组成的元组，在方法解析期间会基于它来查找基类
```

```python
class.mro()
# 此方法在类实例化时被调用，以为其实例定制方法解析顺序，其结果存储于 __mro__ 之中
```

```python
class.__subclasses__()
# 此方法将返回一个由仍然存在的所有此类引用组成的列表
```
