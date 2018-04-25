title: 用 Python 实现一个 ORM
date: 2018-04-25 21:33:31
categories:
- 语言
tags: 
- Python
---

## 前言

本文实现一个非常简单的 ORM 初稿：

1. 完成 Python 类与数据库表的映射
2. 完成类实例与表每行记录的映射
3. 完成实例操作与增删改查的 SQL 语句的映射

这个初稿不涉及数据库的真正操作，只是在 `user.save()` 的时候打印类似 `insert into user ...` 的 SQL 语句。

## 类与表的映射

假设有如下的类：

```python
class User():
    __table__ = 'User_table'
    student_id = IntegerField('studentid', primaryKey=True)
```

回想 Django 的 ORM，每个模型都继承了一个 `Model` 类，我们也如法炮制。而所谓类与表的映射，就是在 Python 虚拟机启动后，自动寻找类属性，并将 `__table__` 转化为表名， `student_id` 转化为列名。这种需求类似于运行时自省，而 普通类的 `__new__` `__init__` 都是实例化类时被调用，在这两个方法上做文章没有用处。

这时候就该用元类 `metaclass` 了。

在 [Python2.7 源码 - 整数对象](http://www.lyyyuna.com/2017/12/24/python-internal2-integer-object/) 中已经有过介绍，元类 `metaclass` 是类的类。不仅是内置类型，用户自定义类型也有元类的概念。

* 内置类定义在 C 源码中，故虚拟机运行后，内存中就有了相应的数据与操作集合来对应这个内置类。
* 而用 `class` 语法定义的类，则需要根据元类 `metaclass` 来创建。
* 内置类也有元类，最终两者在虚拟机中拥有相同的结构。

元类 `metaclass` 实例化的结果就是我们的普通类，由虚拟机启动时自动执行。在元类实例化的过程中，便可以扫描类定义属性，实现类与表的映射。类默认继承自 `object`，获得的元类为 `type`。

Python2.x 中，用以下语法

```python
class C():
    __metaclass__ = Meta
```

可以将类 `C` 对应的元类替换为 `Meta`。这么一看，只要设计自己的元类，并在模型中添加进去就可以了：

```python
class User():
    __metaclass__ = Meta
    __table__ = 'User_table'
    student_id = IntegerField('studentid', primaryKey=True)
```

但这么做，在产品业务代码中暴露了元类 `metaclass`。我们可以设计一个公共的父类，并修改此父类的元类，所有继承的子类都能获得新的元类：

```python
class ModelType(type):
    def __new__(cls, name, bases, attrs):
        return type.__new__(cls, name, bases, attrs):

class Model(dict):
    pass

# Application
class User(Model):
    __table__ = 'User_table'

class Teacher(Model):
    __table__ = 'Teacher_table'
```

当 `User` 类在虚拟机中创建时，其行为就由 `ModelType` 控制。