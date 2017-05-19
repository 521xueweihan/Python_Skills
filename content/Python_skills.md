# Python 的常用技巧

## 介绍
Python 包含了很多语法特性，理解了这些编码的技巧，可以让 Python 使用起来更加得心应手。同时还会介绍了一些 Python 如何解决并发问题。

## 目录
* [一、语法特性](#一语法特性)
    * [1. 魔法方法](#1-魔法方法)
        * [1.1 `__new__`](#11-`__new__`)
        * [1.2 `__getattribute__`](#12-`__getattribute__`)
        * [1.3 其它魔法方法](#13-其它魔法方法)
    * [2. 装饰器](#2-装饰器)
    * [3. 上下文管理](#3-上下文管理)
    * [4. 可迭代对象、迭代器、生成器](#4-可迭代对象迭代器生成器)
        * [4.1 可迭代对象（Iterable）](#41-可迭代对象iterable)
        * [4.2 迭代器（Iterator）](#42-迭代器iterator)
        * [4.3 生成器（Generator）](#43-生成器generator)
    * [5. super](#5super)
* [二、并发](#二并发)
    * [1. GIL](#1-gil)
    * [2. PyPy](#2-pypy)

## 一、语法特性
### 1. 魔法方法
Python 的类是个很好玩的东西，它约定了很多‘特殊的方法’，例如：最常见的 `__init__` 方法，当实例化类的对象时，就会调用 `__init__` 方法；使用 `print` 该类的时候，会调用类的 `__str__` 方法，等等...

这些 Python 约定好的方法，就叫做‘魔法方法’。目的是：为了让实现定制类更加简单、用起来更加方便。比如：我要定制一个类的 `print` 输出内容，只需要实现 `__str__` 方法就可以了，在调用时也不需要 `print TestClass.__str__()`，直接 `print A()` 就可以了。

#### 1.1 `__new__`
`__new__` 方法总是和 `__init__` 方法放在一起讨论。
- `__new__`：创建实例（先调用）；
- `__init__`：初始化实例（后调用）；
- `__new__` 是类方法，`__init__` 是实例方法；
- 重载 `__new__` 方法，需要返回类的实例；

`__new__` 最常见的使用场景是：实现单例模式 —— 不管实例化多少次，该类只有一个实例，代码如下：

```python
class Singleton(object):
    def __new__(cls, *args, **kw):
        if not hasattr(cls,'_instance')
            cls._instance = super(Singleton, cls).__new__(cls, *args, **kw)  
        return cls._instance  # 必须要返回类的实例
```

#### 1.2 `__getattribute__`
不管访问的属性是否存在都会调用这个方法，访问属性，访问优先级别：`类属性 ->  实例属性 -> __getattr__`

在重载 `__getattribute__` 方法时，需要注意防止死循环。

```python
class A(object):
    def __init__(self):
        self.name = 'A'

    def __getattribute__(self, item):
        return self.__dict__[item] # 调用 __dict__ 获取属性时是通过 __getattribute__，所以这么写是个死循环
a = A()
a.name  # 报错：RuntimeError: maximum recursion depth exceeded

#-----正确方式如下-----#

class B(object):
    def __init__(self):
        self.name = 'B'

    def __getattribute__(self, item):
        return super(B, self).__getattribute__(item)
b = B()
b.name # 输出：B
```

#### 1.3 其它魔法方法
- `__str__` 是用 print 和 str 显示的结果，`__repr__` 是直接显示的结果
- `__getitem__` 用类似 `obj[key]` 的方式对对象进行取值
- `__getattr__` 用于获取不存在的属性 `obj.attr`
- `__call__` 使得可以对实例进行调用

参考：
- [定制类和魔法方法](http://funhacks.net/explore-python/Class/magic_method.html)


### 2. 装饰器
在不改动原函数代码的前提下，通过装饰器实现给原函数扩展功能（进行‘装饰’）。

下面为装饰器的示例代码：
```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
#   
#   Author  :   XueWeiHan
#   Date    :   17/5/5 上午10:32
#   Desc    :   装饰器示例代码
from functools import wraps

def hello(fn):
    def wrap():
        print 'hello'
        fn()
    return wrap


@hello
def simple_example():
    print 'world'


simple_example()


def add1(fn):
    # 装饰器不带参数，被装饰的函数结果＋1
    @wraps(fn)
    def wrap(*args):
        return fn(*args) + 1
    return wrap


def addnum(add_num):
    # 装饰器带参数，被装饰的函数结果＋add_num
    def real_decorator(fn):
        @wraps(fn)
        def wrap(*base_num):
            return fn(*base_num) + add_num
        return wrap
    return real_decorator


@add1
def value1():
    return 1


@add1
def value2(base_num):
    return base_num


@addnum(4)
def value3():
    return 1


@addnum(10)
def value4(base_num):
    return base_num

print value1()
print value1.__name__

print value2(2)
print value2.__name__

print value3()
print value3.__name__

print value4(4)
print value4.__name__

```

参考：
- [Python 修饰器的函数式编程](http://coolshell.cn/articles/11265.html)

### 3. 上下文管理
上下文管理是用于便于精确地分配和释放资源。例如：文件IO、数据库连接等。这些操作在使用完，都需要释放资源。

上下文管理是通过 Python 关键字 with 触发，在自定义类中实现 `__enter__` 和 `__exit__` 两个魔法方法。
1. 上下文开始时调用 `__enter__` 方法，如果该方法又返回值，该返回值会赋值给 `as` 后面的变量
2. 结束时调用 `__exit__` 方法，该方法还可以优雅的处理异常

下面为上下文管理的示例代码：
```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
#   
#   Author  :   XueWeiHan
#   Date    :   17/5/7 下午4:50
#   Desc    :   上下文管理示例代码


class File(object):
    def __init__(self, path, method_type):
        self.fb = open(path, method_type)

    def __enter__(self):
		# 如果有 return 的对象，则会赋值给 as 后面的变量
        return self.fb

    def __exit__(self, exc_type, exc_val, exc_tb):
        """
        :param exc_type: 异常类型
        :param exc_val: 异常信息
        :param exc_tb: 异常错误 traceback
        :return: 如果 return True 则不会抛出异常
        """
        self.fb.close()
        if exc_val:
            return False
        else:
            return True


with File('./find_pi.py', 'r+') as fb:
    print fb.read()  # 正确调用
    # print fb.rrr() # 没有这个方法，引出异常
```

也可以使用 `contextlib` 模块提供的 `contextmanager` 装饰器实现上下文管理

### 4. 可迭代对象、迭代器、生成器
#### 4.1 可迭代对象（Iterable）
Python 中任意的对象，只要它定义了可以返回一个迭代器的 `__iter__` 方法，或者定义了可以支持下标索引的 `__getitem__` 方法，那么它就是一个可迭代对象。可以用在 `for` 语句中的都是可迭代的，比如：list、string、dict、set

这些可迭代的对象你可以随意的读取所以非常方便易用，但是**你必须把它们的值放到内存里，当它们有很多值时就会消耗太多的内存。**

#### 4.2 迭代器（Iterator）
任意对象，只要定义了 `next` 方法和 `__iter__` 方法返回自己本身，它就是一个迭代器。

像 list、set 等，是没有 `next` 方法，所以不是迭代器。但是，可以使用 `iter` 内置方法实现迭代器。

使用类实现迭代器，示例代码如下：
```python
class Squares(object):
    def __init__(self, start, stop):
        self.start = start
        self.stop = stop
    def __iter__(self):
        return self
    def next(self):
        if self.start >= self.stop:
            raise StopIteration
        current = self.start * self.start
        self.start += 1
        return current
```

#### 4.3 生成器（Generator）
通过 `yield` 关键字，实现：不需要在生成元素巨多的迭代器时（[1, 2, ..., 100000000]），就开辟所有空间。而是每次调用 `next` 方法或 `for` 循环到该元素时才计算该位置元素的数值（即：惰性计算）。

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
#   
#   Author  :   XueWeiHan
#   Date    :   17/5/9 上午12:28
#   Desc    :   生成器示例代码

# 报错，因为太大了，生成不了
# big_list1 = range(1111111111111111111)


def generate_big_list(n):
    """
    使用生成器，生成大数据集
    """
    start = 0
    while start < n:
        yield start
        start += 1

big_list2 = generate_big_list(1111111111111111111)

for fi_big_list2 in big_list2:
    print fi_big_list2

# 使用内置 xrange 生成大数据集，
big_list3 = xrange(1111111111111111111)

# for fi_big_list3 in big_list3:
#     print fi_big_list3

```

参考：
- [Python 进阶——生成器篇](https://eastlakeside.gitbooks.io/interpy-zh/content/Generators/)
- [Python 关键字 yield 的解释](http://pyzh.readthedocs.io/en/latest/the-python-yield-keyword-explained.html)

### 5. super
Python 继承中，重写（在不改变方法名的前提下修改方法）父类方法实例代码（不推荐）：
```python
class Base(object):
    def __init__(self):
        print "enter Base"
        print "leave Base"

class A(Base):
    def __init__(self):
        print 'enter A'
        Base.__init__(self)
        print 'leave A'

a = A()
# 输出结果：
# enter A
# enter Base
# leave Base
# leave A
```

Python 重写父类方法时，**推荐使用 `super` 关键字**。

- 语法：`super(子类名, self/cls).需要继承的父类方法(参数)`
- 好处：
    - 父类名称改变，不需要修改子类中继承的地方
    - 多继承时，继承顺序按照子类的 `mro` 方法返回的列表中的顺序访问

多继承实例代码如下：
```python
class Base(object):
    def __init__(self):
        print "enter Base"
        print "leave Base"

class A(Base):
    def __init__(self):
        print "enter A"
        super(A, self).__init__()
        print "leave A"

class B(Base):
    def __init__(self):
        print "enter B"
        super(B, self).__init__()
        print "leave B"

class C(A, B):
    def __init__(self):
        print "enter C"
        super(C, self).__init__()
        print "leave C"

print C.mro()   
# 输出：[__main__.C, __main__.A, __main__.B, __main__.Base, object]

c = C()
# 输出如下：
# enter C
# enter A
# enter B
# enter Base
# leave Base
# leave B
# leave A
# leave C

```

参考：
- [你不知道的 super](http://funhacks.net/explore-python/Class/super.html)

## 二、并发
### 1. GIL
GIL 的全程为 Global Interpreter Lock ，意即全局解释器锁。在 Python 语言的主流解释器 CPython 中，GIL 是一个货真价实的全局线程锁，在解释器解释执行任何 Python 代码时，都需要先获得这把锁才行，在遇到 I/O 操作时会释放这把锁。如果是纯计算的程序，没有 I/O 操作，解释器会每隔特定次操作就释放这把锁，让别的线程有机会执行（这个次数可以通过 sys.setcheckinterval 来调整）。所以虽然 CPython 的线程库直接封装操作系统的原生线程，但 CPython 进程做为一个整体，同一时间只会有一个获得了 GIL 的线程在跑，其它的线程都处于等待状态等着 GIL 的释放。

GIL 最大的问题就是 Python 的多线程程序并不能利用多核 CPU 的优势。

Python 只能跑在一个核上，这也就为什么，Python 只能跑满一个 CPU，而且多线程由于分时切换，还跑不满一个 CPU。

所以，“不要使用多线程，使用多进程”，推荐使用多进程（multiprocessing 模块）或用 C 扩展解决计算密集型任务。

参考：
- [python 线程，GIL 和 ctypes](http://zhuoqiang.me/python-thread-gil-and-ctypes.html)
- [Python 的全局锁问题](http://python3-cookbook.readthedocs.io/zh_CN/latest/c12/p09_dealing_with_gil_stop_worring_about_it.html)

### 2. PyPy
PyPy 是 Python 的一种解释器，用 Python 的子集 rPython 实现的。

PyPy 的执行效率高于 CPython 是因为 JIT，也就是解释完 Python 代码后，编译代码的过程中会有优化，实现效率的提升。但是由于 JIT 的优化过程是耗时的，所以的编译过程比 CPython 漫长。

参考：
- [PyPy 为什么会比 CPython 还要快？](https://www.zhihu.com/question/19588346)

### 3. 多进程

### 4. 多线程



多线程的使用存在**线程安全问题**，即：有可能出现多个线程同时更改数据造成所得到的数据是脏数据（错误的数据）。所以，在多线程执行写入的操作时，要考虑线程同步问题。也就是需要加锁，以保证同一时间只有一个线程写入数据。（注意：加锁操作完后，要释放）

### 5. 协程
1. 协程是比线程**更轻量**（资源使用更少）。

2. 不同于线程，线程是抢占式的调度，而协程是协同式的调度，协程需要**自己做调度**。

3. 协程是**线程安全的**，一个进程可以同时存在多个协程，但是只有一个协程是激活的。


### 6. 事件循环（Event Loop）
### 7. gentlet
### 8. Gevent
