# Python 好用的标准库

## 介绍
Python 自带了很多库，称为：标准库。

下面介绍了我在学习、使用、工作、查阅资料过程中收集的，好用的 Python 标准库。

## itertools
## collections

## timeit
显示代码片段的运行时间。timeit它接受一个参数为每个测试中调用被计时语句的次数，默认为**一百万次**；返回所耗费的**秒数**。

1.通过命令运行

```sh
python -m timeit '"-".join([str(n) for n in range(100)])'

结果：
10000 loops, best of 3: 27.5 usec per loop
```

2.代码内调用

```python
import timeit
print timeit.timeit('"-".join(str(n) for n in range(100))', number=10000)

结果：
0.3018611848820001
```

3.输出执行某个函数所用时间

```python
def test():
    """Stupid test function"""
    L = [i for i in range(100)]

if __name__ == '__main__':
    import timeit
    print(timeit.timeit("test()", setup="from __main__ import test"))
```
[官方文档 timeit](https://docs.python.org/2.7/library/timeit.html)

## bisect
通过二分算法，维持一个有序列表，每增加一个元素，列表无需重新排序。因为，二分法找到增加的元素在列表中的位置，然后直接插入。而不是把增加的元素放到列表中，再重新进行一次排序。对于很长的列表，会节省很多时间和空间。
```python
import timeit
import bisect
import random


def test_bisect(l):
    for i in range(0, 10):
        r = random.randint(1, 100000)
        bisect.insort(l, r)


def normal(l):
    for i in range(0, 10):
        r = random.randint(1, 100000)
        l.append(r)
        l.sort()

if __name__ == '__main__':
    print(timeit.timeit('test_bisect([num for num in xrange(100000000)])', setup='from __main__ import test_bisect', number=1))
    print(timeit.timeit('normal([num for num in xrange(100000000)])', setup='from __main__ import normal', number=1))

结果：
8.9976670742
23.4078929424
```
[官方文档 bisect](https://docs.python.org/2.7/library/bisect.html)

## cProfile
这个库是2.5新加的，比profile性能好，推荐使用。cProfile可以用于分析python脚本的运行时间和脚本中耗时的代码。

```python
import cProfile
cProfile.run('a = range(1000000) print a')

# 输出：
"""cProfile.run('range(1000000)')
         3 function calls in 0.030 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.006    0.006    0.030    0.030 <string>:1(<module>)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        1    0.024    0.024    0.024    0.024 {range}
"""        
#ncalls 函数的被调用次数
#tottime 函数总计运行时间，除去函数中调用的函数运行时间
#percall 函数运行一次的平均时间，等于tottime/ncalls
#cumtime 函数总计运行时间，含调用的函数运行时间
#percall 函数运行一次的平均时间，等于cumtime/ncalls
#filename:lineno(function) 函数所在的文件名，函数的行号，函数名

```
[官方文档 cProfile](https://docs.python.org/2/library/profile.html)

## operator
相当于一个工具箱，你可以通过`operator.xx()`调用该对象的拥有的各种方法，还是用代码说明吧：

```python
import operator
# 先来个简单的
a = operator.add(3, 4)
print a # 7

# 如果有一个类，你要得到他的所有属性
class Test(object):
    def __init__(self):
        self.name = 'xueweihan'
        self.age = 23
        self.sex = 'boy'
t = Test()
f = operator.attrgetter('name', 'age', 'sex')
all_attr = f(t)
print all_attr  # 需要获取的属性的元祖 ('xueweihan', 23, 'boy')

# 实用场景，排序，提取值
inventory = [('apple', 3), ('banana', 2), ('pear', 5), ('orange', 1)]
getcount = itemgetter(1)
map(getcount, inventory)  ＃[3, 2, 5, 1]

sorted(inventory, key=getcount)  ＃ [('orange', 1), ('banana', 2), ('apple', 3), ('pear', 5)]

```
[官方文档 operator](https://docs.python.org/2/library/operator.html)

## pprint
顾名思义：“漂亮地打印”。适用场景，需要 `print` 特别长的字典。例如：

```python
# 例如直接print一个结果，`quote_result`是一个很长的字典
print quote_result
# 打印结果如下
{ u'is_done': True, u'detail': {..省略...}, u'require_owner_info': False}

# 调用pprint的pprint()函数，打印结果
import pprint
pprint.pprint(quote_result)
# 打印结果如下
{u'detail': {u'biz_info': {u'No': u'',
                           u'discount': u'0.5670',
                           u'discount1': u'0.5670',
                           u'insure_type': 1,
                           u'payList': [],
                           u'premium_detail': {u'combust': {u'amount': u'27385.60',
                                                            u'code': u'combust',
                                                            u'group_code': 2,
                                                            u'premium': u'77.64'},
                                               ...
                                               u'third': {u'amount': u'300000.00',
                                                          u'code': u'third',
                                                          u'group_code': 1,
                                                          u'premium': u'695.14'}},
                           u'standard': u'3278.25',
                           u'startDate': u'2016-04-14',
                           u'total': u'1858.77'},
             u'is_support_counter': True,
             u'pay_info': [{u'total': u'1858.77', u'total_cent': 185877}],
             u'require_owner_info': False,
             },
 u'is_done': True,
 u'is_success': True,
}

```

[官方文档 pprint](https://docs.python.org/2.7/library/pprint.html)
