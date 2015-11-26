# Extras Learned
---
1. `numbers`模块
    `numbers`模块主要是定义一些数字类型的抽象基类，即`Abstract Base Classes (ABCs) for numbers`。通过`numbers`定义的数字类型的继承关系如下如所示：
    
    ```
               Number
                  |
               Complex
                  | \
                  |  \
               Real complex
                  | \
                  |  \
            Rational float
                  |
                Integral
                  | \
                  |  \
                int  long
    ```
    因此：
    * `issubclass(int, numbers.Integral)`或`issubclass(long, numbers.Integral)`返回`True`
    * `issubclass(int, numbers.Real)`或`issubclass(float, Complex)`返回`True`
    * `issubclass(int, float)`或`issubclass(int, complex)`返回`False`
2. `textwrap`模块
  `textwrap`模块的主要功能是格式化输出的字符串的格式。
3. `calendar`模块
  `tornado.httputil`模块的`format_timestamp`函数，可以将`timestamp`/`time.struc_time`/`datetime.datetime`的对象格式化成`HTTP`格式的时间格式。  
  主要使用的是[`calendar.timegm`](https://docs.python.org/2/library/calendar.html#calendar.timegm)函数。该函数和`time.gmtime`函数是互为逆反的。

    ```
    >>> import time
    >>> cur = time.time()
    >>> cur
    1446812390.056556
    >>> cur_gm = time.gmtime(cur)
    >>> cur_gm
    time.struct_time(tm_year=2015, tm_mon=11, tm_mday=6,     tm_hour=12, tm_min=19, tm_sec=50, tm_wday=4, tm_yday=310, tm_isdst=0)
    >>> import calendar
    >>> calendar.timegm(cur_gm)
    1446812390
    ```
  并且`datetime.datetime`格式的数据也能通过`datetime.datetime.utctimetuple`转换成`time.time_struct`格式的时间数据。因此也可以通过`calendar.timegm`函数返回timestamp。
4. `email.utils.formatdate`函数  
  按照[`RFC2822`](http://tools.ietf.org/html/rfc2822.html)格式(个人感觉跟`iso8601`格式很像)格式化`timestamp`。通过`email.utils.formatdate(timestamp, usegmt=True)`可以返回满足[`HTTP`协议的时间格式](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html)
5. `collections.namedtuple`类
  该类实现了命名tuple。调用传入的第一个参数是一个tuple的子类，并且这个子类有具体使用如下所示:

    ```
    >>> Point = namedtuple('Point', ['x', 'y'])
    >>> Point.__doc__                   # docstring for the new class
    'Point(x, y)'
    >>> p = Point(11, y=22)             # instantiate with positional args or keywords
    >>> p[0] + p[1]                     # indexable like a plain tuple
    33
    >>> x, y = p                        # unpack like a regular tuple
    >>> x, y
    (11, 22)
    >>> p.x + p.y                       # fields also accessable by name
    33
    >>> d = p._asdict()                 # convert to a dictionary
    >>> d['x']
    11
    >>> Point(**d)                      # convert from a dictionary
    Point(x=11, y=22)
    >>> p._replace(x=100)               # _replace() is like str.replace() but targets named fields
    Point(x=100, y=22)
    ```
