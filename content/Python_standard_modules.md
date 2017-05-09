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
[官方文档timeit](https://docs.python.org/2.7/library/timeit.html)

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
[官方文档bisect](https://docs.python.org/2.7/library/bisect.html)

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
[官方文档cProfile](https://docs.python.org/2/library/profile.html)

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
[官方文档](https://docs.python.org/2/library/operator.html)

## inspect
inspect是检查的意思，所以这个模块提供了一系列的实用方法，用于得到：模块，类，方法，函数等对象的各种信息。例子：

```python
import inspect

print inspect.getmember(int)
# 输出如下
[('__abs__', <slot wrapper '__abs__' of 'int' objects>),
 ('__add__', <slot wrapper '__add__' of 'int' objects>),
 ('__and__', <slot wrapper '__and__' of 'int' objects>),
 ('__class__', type),
 ('__cmp__', <slot wrapper '__cmp__' of 'int' objects>),
 ('__coerce__', <slot wrapper '__coerce__' of 'int' objects>),
 ('__delattr__', <slot wrapper '__delattr__' of 'object' objects>),
 ('__div__', <slot wrapper '__div__' of 'int' objects>),
 ('__divmod__', <slot wrapper '__divmod__' of 'int' objects>),
 ('__doc__',
  "int(x=0) -> int or long\nint(x, base=10) -> int or long\n\nConvert a number or string to an integer, or return 0 if no arguments\nare given.  If x is floating point, the conversion truncates towards zero.\nIf x is outside the integer range, the function returns a long instead.\n\nIf x is not a number or if base is given, then x must be a string or\nUnicode object representing an integer literal in the given base.  The\nliteral can be preceded by '+' or '-' and be surrounded by whitespace.\nThe base defaults to 10.  Valid bases are 0 and 2-36.  Base 0 means to\ninterpret the base from the string as an integer literal.\n>>> int('0b100', base=0)\n4"),
 ('__float__', <slot wrapper '__float__' of 'int' objects>),
 ('__floordiv__', <slot wrapper '__floordiv__' of 'int' objects>),
 ('__format__', <method '__format__' of 'int' objects>),
 ('__getattribute__', <slot wrapper '__getattribute__' of 'int' objects>),
 ('__getnewargs__', <method '__getnewargs__' of 'int' objects>),
 ('__hash__', <slot wrapper '__hash__' of 'int' objects>),
 ('__hex__', <slot wrapper '__hex__' of 'int' objects>),
 ('__index__', <slot wrapper '__index__' of 'int' objects>),
 ('__init__', <slot wrapper '__init__' of 'object' objects>),
 ('__int__', <slot wrapper '__int__' of 'int' objects>),
 ('__invert__', <slot wrapper '__invert__' of 'int' objects>),
 ('__long__', <slot wrapper '__long__' of 'int' objects>),
 ('__lshift__', <slot wrapper '__lshift__' of 'int' objects>),
 ('__mod__', <slot wrapper '__mod__' of 'int' objects>),
 ('__mul__', <slot wrapper '__mul__' of 'int' objects>),
 ('__neg__', <slot wrapper '__neg__' of 'int' objects>),
 ('__new__', <function __new__>),
 ('__nonzero__', <slot wrapper '__nonzero__' of 'int' objects>),
 ('__oct__', <slot wrapper '__oct__' of 'int' objects>),
 ('__or__', <slot wrapper '__or__' of 'int' objects>),
 ('__pos__', <slot wrapper '__pos__' of 'int' objects>),
 ('__pow__', <slot wrapper '__pow__' of 'int' objects>),
 ('__radd__', <slot wrapper '__radd__' of 'int' objects>),
 ('__rand__', <slot wrapper '__rand__' of 'int' objects>),
 ('__rdiv__', <slot wrapper '__rdiv__' of 'int' objects>),
 ('__rdivmod__', <slot wrapper '__rdivmod__' of 'int' objects>),
 ('__reduce__', <method '__reduce__' of 'object' objects>),
 ('__reduce_ex__', <method '__reduce_ex__' of 'object' objects>),
 ('__repr__', <slot wrapper '__repr__' of 'int' objects>),
 ('__rfloordiv__', <slot wrapper '__rfloordiv__' of 'int' objects>),
 ('__rlshift__', <slot wrapper '__rlshift__' of 'int' objects>),
 ('__rmod__', <slot wrapper '__rmod__' of 'int' objects>),
 ('__rmul__', <slot wrapper '__rmul__' of 'int' objects>),
 ('__ror__', <slot wrapper '__ror__' of 'int' objects>),
 ('__rpow__', <slot wrapper '__rpow__' of 'int' objects>),
 ('__rrshift__', <slot wrapper '__rrshift__' of 'int' objects>),
 ('__rshift__', <slot wrapper '__rshift__' of 'int' objects>),
 ('__rsub__', <slot wrapper '__rsub__' of 'int' objects>),
 ('__rtruediv__', <slot wrapper '__rtruediv__' of 'int' objects>),
 ('__rxor__', <slot wrapper '__rxor__' of 'int' objects>),
 ('__setattr__', <slot wrapper '__setattr__' of 'object' objects>),
 ('__sizeof__', <method '__sizeof__' of 'object' objects>),
 ('__str__', <slot wrapper '__str__' of 'int' objects>),
 ('__sub__', <slot wrapper '__sub__' of 'int' objects>),
 ('__subclasshook__', <function __subclasshook__>),
 ('__truediv__', <slot wrapper '__truediv__' of 'int' objects>),
 ('__trunc__', <method '__trunc__' of 'int' objects>),
 ('__xor__', <slot wrapper '__xor__' of 'int' objects>),
 ('bit_length', <method 'bit_length' of 'int' objects>),
 ('conjugate', <method 'conjugate' of 'int' objects>),
 ('denominator', <attribute 'denominator' of 'int' objects>),
 ('imag', <attribute 'imag' of 'int' objects>),
 ('numerator', <attribute 'numerator' of 'int' objects>),
 ('real', <attribute 'real' of 'int' objects>)]

```
具体使用可以参考官方文档：[标准库：inspect](https://docs.python.org/2/library/inspect.html)

## pprint
顾名思义：“漂亮地打印”。适用场景，需要`print`特别长的字典。例如：

```python
# 例如直接print一个结果，`quote_result`是一个很长的字典
print quote_result
# 打印结果如下
{ u'is_done': True, u'detail': {u'biz_info': {u'startDate': u'2016-04-14', u'endDate': u'2017-04-13', u'No': u'', u'standard': u'3278.25', u'discount': u'0.5670', u'discount1': u'0.5670', u'payList': [], u'total': u'1858.77', u'insure_type': 1, u'premium_detail': {u'third': {u'group_code': 1, u'amount': u'300000.00', u'code': u'third', u'premium': u'695.14'}, u'damage': {u'group_code': 1, u'amount': u'77800.00', u'code': u'damage', u'premium': u'843.54'}, u'combust': {u'group_code': 2, u'amount': u'27385.60', u'code': u'combust', u'premium': u'77.64'}, u'exempt_damage': {u'group_code': 2, u'amount': u'0.00', u'code': u'exempt_damage', u'premium': u'126.53'}, u'exempt_third': {u'group_code': 2, u'amount': u'0.00', u'code': u'exempt_third', u'premium': u'104.27'}, u'exempt_extra': {u'group_code': 2, u'amount': u'0.00', u'code': u'exempt_extra', u'premium': u'11.65'}}}, u'is_support_counter': True, u'require_owner_info': False, u'force_info': {u'lastYearEndDate': u'2017-03-24', u'lastYearStartDate': u'2016-03-25'}, u'cache_hit': True, u'pay_info': [{u'total_cent': 185877, u'total': u'1858.77'}]}, u'is_success': True}

# 调用pprint的pprint()函数，打印结果
import pprint
pprint.pprint(quote_result)
# 打印结果如下
{u'detail': {u'biz_info': {u'No': u'',
                           u'discount': u'0.5670',
                           u'discount1': u'0.5670',
                           u'endDate': u'2017-04-13',
                           u'insure_type': 1,
                           u'payList': [],
                           u'premium_detail': {u'combust': {u'amount': u'27385.60',
                                                            u'code': u'combust',
                                                            u'group_code': 2,
                                                            u'premium': u'77.64'},
                                               u'damage': {u'amount': u'77800.00',
                                                           u'code': u'damage',
                                                           u'group_code': 1,
                                                           u'premium': u'843.54'},
                                               u'exempt_damage': {u'amount': u'0.00',
                                                                  u'code': u'exempt_damage',
                                                                  u'group_code': 2,
                                                                  u'premium': u'126.53'},
                                               u'exempt_extra': {u'amount': u'0.00',
                                                                 u'code': u'exempt_extra',
                                                                 u'group_code': 2,
                                                                 u'premium': u'11.65'},
                                               u'exempt_third': {u'amount': u'0.00',
                                                                 u'code': u'exempt_third',
                                                                 u'group_code': 2,
                                                                 u'premium': u'104.27'},
                                               u'third': {u'amount': u'300000.00',
                                                          u'code': u'third',
                                                          u'group_code': 1,
                                                          u'premium': u'695.14'}},
                           u'standard': u'3278.25',
                           u'startDate': u'2016-04-14',
                           u'total': u'1858.77'},
             u'cache_hit': True,
             u'force_info': {u'lastYearEndDate': u'2017-03-24',
                             u'lastYearStartDate': u'2016-03-25'},
             u'is_support_counter': True,
             u'pay_info': [{u'total': u'1858.77', u'total_cent': 185877}],
             u'require_owner_info': False,
             },
 u'is_done': True,
 u'is_success': True,
}

```
漂亮的打印出json数据：`cat file.json | python -m json.tool`

[官方文档pprint](https://docs.python.org/2.7/library/pprint.html)
