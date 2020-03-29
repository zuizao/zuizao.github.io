python datetime模块介绍

**1.datetime模块包含的类**

| 类别          | 说明                                                     |
| ------------- | -------------------------------------------------------- |
| date          | 日期对象，常用的属性有year,month,day                     |
| time          | 时间对象                                                 |
| datetime      | 日期时间对象，常用的属性有hour,minute,second,microsecond |
| datetime_CAPI | 日期时间对象C语言接口                                    |
| timedelta     | 时间间隔，跨度                                           |
| tzinfo        | 时间区域对象                                             |

**2.datetime模块中包含的常量**

| 常量    | 功能说明           | 用法             | 返回值 |
| ------- | ------------------ | ---------------- | ------ |
| MAXYEAR | 返回表示的最大年份 | datetime.MAXYEAR | 9999   |
| MINYEAR | 返回表示的最小年份 | datetime.MINYEAR | 1      |

date类

1.date类对象有year,month及day三部分构成

```python
date(year,month,day)
```

2.通过year,month,mday三个数据描述符可以进行访问:

```python
>>> import datetime
>>> a=datetime.date.today()
>>> a
datetime.date(2020, 3, 28)
>>> a.year
2020
>>> a.month
3
>>> a.day
28
>>> 

```

3.也可以通过__getattribute__方法获得上述值

```python
>>> a.__getattribute__('year')
2020
>>> a.__getattribute__('month')
3
>>> a.__getattribute__('day')
28

```

date对象中包含的方法和属性

1.用于日期比较大小的方法

| 方法名   | 方法说明     | 用法            |
| -------- | ------------ | --------------- |
| `__eq__` | 等于x==y     | `x.__eq__(y)`   |
| `__ge__` | 大于等于x>=y | `x.__ge__(y)`   |
| `__gt__` | 大于x>y      | `x.__gt__(y)`   |
| `__le__` | 小于等于x<=y | `x.__le__(y)`   |
| `__lt__` | 小于x        | `__x.__lt__(y)` |
| `__ne__` | 不等于x!=y   | `__x.ne__(y)`   |

以上方法的返回值为True\False

例子如下：

```python
>>> a=datetime.date(2020,3,20)
>>> b=datetime.date(2020,3,28)
>>> a.__eq__(b)
False
>>> a.__ge__(b)
False
>>> a.__gt__(b)
False
>>> a.__le__(b)
True
>>> a.__lt__(b)
True
>>> a.__ne__(b)
True

```

2.获得两个日期相差多少天

使用 `__sub__` 和 `__rsub__` 方法

例子如下

```python
>>> a
datetime.date(2020, 3, 20)
>>> b
datetime.date(2020, 3, 28)
>>> a.__sub__(b)
datetime.timedelta(days=-8)
>>> a.__rsub__(b)
datetime.timedelta(days=8)
```

计算结果的返回值类型为datetime.timedelta，如果获得整数类型的结果则按下面的方法操作:

```python
>>> a.__sub__(b).days
-8
>>> a.__rsub__(b).days
8

```

3.ISO标准化日期

如果想要让所使用的日期符合ISO标准，那么使用如下三个方法:
1).* isocalendar(...)*:返回一个包含三个值的元组，三个值依次为：year年份，week number周数，weekday星期数（周一为1…周日为7)：
示例如下

```python
>>> a=datetime.date(2020,3,28)
>>> a.isocalendar()
(2020, 13, 6)
>>> a.isocalendar()[0]
2020
>>> a.isocalendar()[1]
13
>>> a.isocalendar()[2]
6
```

2).`isoformat()`：返回符合iso 8601标准 `(YYYY-MM-DD)`的日期字符串;

示例如下

```python
>>> a=datetime.date(2020,3,28)
>>> a.isoformat()
'2020-03-28'

```

3).isoweekday:返回符合iso标准的指定日期所在的星期数(周一为1，周日为7)

示例如下

```python
>>> a=datetime.date(2020,3,28)
>>> a.isoweekday()
6
```

4).与isoweekday相似的还有一个weekday()方法，只不过是weekday方法返回的周一为0,周日为6

示例如下

```python
>>> a=datetime.date(2020,3,28)
>>> a.weekday()
5
```

4.其他方法与属性

1).timetuple:该方法为了兼容time.localtime返回一个类型为time.struct_time的数组，但有关时间的部分元素为0:

```python
>>> a=datetime.date(2020,3,28)
>>> a.timetuple()
time.struct_time(tm_year=2020, tm_mon=3, tm_mday=28, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=5, tm_yday=88, tm_isdst=-1)
>>> a.timetuple().tm_year
2020
>>> a.timetuple().tm_mon
3
>>> a.timetuple().tm_mday
28
```

2).toordinal:返回公元公立开始到现在的天数。公元1年1月1日为1

```python
>>> a=datetime.date(2020,3,28)
>>> a.toordinal()
737512
```

