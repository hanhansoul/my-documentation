# eventlet使用教程

## eventlet并发编程

### 创建绿色线程

eventlet提供了三个基本方法用于生成一个绿色线程，该线程将调用指定的函数`func`：

1. `eventlet.spawn(func, *args, **kw)`
2. `eventlet.spawn_n(func, *args, **kw)`
3. `eventlet.spawn_after(seconds, func, *args, **kw)`

示例：



### 绿色线程池

eventlet提供了：

1. eventlet.GreenPool
2. eventlet.GreenPile

```python
import eventlet
from eventlet.green import urllib2

urls = ["http://www.google.com/intl/en_ALL/images/logo.gif",
       "https://www.python.org/static/img/python-logo.png",
       "http://us.i1.yimg.com/us.yimg.com/i/ww/beta/y3.gif"]

def fetch(url):
    return urllib2.urlopen(url).read()

pool = eventlet.GreenPool()
for body in pool.imap(fetch, urls):
    print("got body", len(body))
```



### 队列Queue



## eventlet网络编程

