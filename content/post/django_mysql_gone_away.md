+++
author = "zhaojames0707"
comments = true
date = "2016-06-29T23:22:18+08:00"
draft = false
share = true
tags = ["python", "django", "mysql"]
title = "解决 django 中 mysql gone away 的问题"

+++

最近在项目中，我使用 Django Command 模块写了一个脚本，处理从 MQ 发来的消息，并入库。在测试过程中，程序运行良好，但是在程序上线并运行一段时间后，出现了以下错误：

<!--more-->

```python
OperationalError: (2006, 'MySQL server has gone away')
```

### 发现问题

经过一段时间的排查后，我发现了问题的原因：因为我要入库的消息并不频繁，所以我的程序的入库操作之间可能会间隔一段时间，而当这段时间大于 MySQL 配置的超时时间后，MySQL 便会主动断开与该程序的连接；此时，程序做数据库相关操作，则会发现数据库连接已经失效，因而报 ```MySQL server has gone away```的异常。

查看 MySQL 配置的超时时间方法为：

```
show variables like 'wait_timeout';
```

### 分析问题

在网上搜索相关问题后，我发现有很多人问过相关问题，而 Django 官网的这个[讨论](https://code.djangoproject.com/ticket/21597#comment:29)，给了我很大帮助。

处理方法有两个：

1) 每次调用完 Model 后，手动关闭 connection

```python
from django.db import connection

connection.close()
```

2) 调整数据库的超时时间(不推荐！)

但是，这两个都不适合我的程序：

* 方法1是针对 Model 操作间隔一定很长的情况，如果某个时间段内需要很频繁的操作数据库，那么频繁关闭-新建数据库连接无疑是低效的。
* 方法2直接修改数据库超时时间，很容易影响别的服务，会带来很多潜在的问题。

针对我的情况，我参考了 Django 源码涉及数据库连接维护的部分。

在 ```django.db.__init__.py``` 中，有以下代码片段:

```python
# Register an event to reset transaction state and close connections past
# their lifetime.
def close_old_connections(**kwargs):
    for conn in connections.all():
        conn.close_if_unusable_or_obsolete()
signals.request_started.connect(close_old_connections)
signals.request_finished.connect(close_old_connections)
```

可见，Django 将*请求开始*/*请求结束*信号绑定给了 ```close_old_connections```函数，每当有请求开始和结束以后，Django 都会检查目前又没有失效的连接，如果有的话就将其关闭。通过这种办法，Django 保证处理请求时，数据库连接都是可用的，不会出现我遇到的问题；而我的程序在涉及 Model 操作时，没有检查连接的有效性，因而出现了题目中的错误。

### 解决问题

在定位到问题且知道处理方法后，接下来的工作就非常简单了。
仿照上述代码，定义函数：

```python
from django import connection

def close_unusable_connection():
    if not connection.connection:
        return
    if connection.is_usable():
        return
    connection.close()
```

然后在每次 Model 操作前调用```close_unusable_connection()```就解决问题了。当然，直接从```django.db```中引入```close_old_connections```然后调用应该也能解决问题，且支持多个连接的情况，请各位自行尝试。