# Python 的实用技巧

## 介绍
Python 包含了很多语法特性，以提高编程的效率。这里还会介绍一些 Python 并发实现和

## 语法特性
### 装饰器
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

### 上下文管理
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

### 可迭代对象、迭代器、生成器
#### 可迭代对象（Iterable）
Python 中任意的对象，只要它定义了可以返回一个迭代器的 `__iter__` 方法，或者定义了可以支持下标索引的 `__getitem__` 方法，那么它就是一个可迭代对象。可以用在 `for` 语句中的都是可迭代的，比如：list、string、dict、set

这些可迭代的对象你可以随意的读取所以非常方便易用，但是**你必须把它们的值放到内存里，当它们有很多值时就会消耗太多的内存。**

#### 迭代器（Iterator）
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

#### 生成器（Generator）
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

### super
### 魔法方法

## 并发
### GIL
GIL 的全程为 Global Interpreter Lock ，意即全局解释器锁。在 Python 语言的主流解释器 CPython 中，GIL 是一个货真价实的全局线程锁，在解释器解释执行任何 Python 代码时，都需要先获得这把锁才行，在遇到 I/O 操作时会释放这把锁。如果是纯计算的程序，没有 I/O 操作，解释器会每隔特定次操作就释放这把锁，让别的线程有机会执行（这个次数可以通过 sys.setcheckinterval 来调整）。所以虽然 CPython 的线程库直接封装操作系统的原生线程，但 CPython 进程做为一个整体，同一时间只会有一个获得了 GIL 的线程在跑，其它的线程都处于等待状态等着 GIL 的释放。

GIL 最大的问题就是 Python 的多线程程序并不能利用多核 CPU 的优势。

Python 只能跑在一个核上，这也就为什么，Python 只能跑满一个 CPU，而且多线程由于分时切换，还跑不满一个 CPU。

所以，“不要使用多线程，使用多进程”，推荐使用多进程（multiprocessing 模块）或用 C 扩展解决计算密集型任务。

参考：
- [python 线程，GIL 和 ctypes](http://zhuoqiang.me/python-thread-gil-and-ctypes.html)
- [Python 的全局锁问题](http://python3-cookbook.readthedocs.io/zh_CN/latest/c12/p09_dealing_with_gil_stop_worring_about_it.html)

### 多进程

### 多线程



多线程的使用存在**线程安全问题**，即：有可能出现多个线程同时更改数据造成所得到的数据是脏数据（错误的数据）。所以，在多线程执行写入的操作时，要考虑线程同步问题。也就是需要加锁，以保证同一时间只有一个线程写入数据。（注意：加锁操作完后，要释放）

### 协程
1. 协程是比线程**更轻量**（资源使用更少）。

2. 不同于线程，线程是抢占式的调度，而协程是协同式的调度，协程需要**自己做调度**。

3. 协程是**线程安全的**，一个进程可以同时存在多个协程，但是只有一个协程是激活的。


### 事件循环（Event Loop）
### gentlet
### Gevent
### PyPy
PyPy 是 Python 的一种解释器，用 Python 的子集 rPython 实现的。

PyPy 的执行效率高于 CPython 是因为 JIT，也就是解释完 Python 代码后，编译代码的过程中会有优化，实现效率的提升。但是由于 JIT 的优化过程是耗时的，所以的编译过程比 CPython 漫长。

参考：
- [PyPy 为什么会比 CPython 还要快？](https://www.zhihu.com/question/19588346)
