+++
author = "zhaojames0707"
comments = true
date = "2017-02-05T21:11:00+08:00"
draft = false
image = ""
share = true
tags = ["python", "django"]
title = "探究 Django 数据库连接"
discription = "foo"

+++

公司的 Django 项目中遇到了数据库连接方面的问题，引发了我对 Django 数据库连接内部实现的关注。
<!--more-->



*本文使用的 django 版本为 1.8.2，gunicorn 版本为19.6.0*

Django 会根据`settings`中`DATABASES`的配置，对每个数据库创建一个`DatabaseWrapper`实例，并将与该数据库的连接存放到实例的`connection`属性中。

Django 对支持的每种数据库`backend`都有不同的`DatabaseWrapper`实现（例如 MySQL 的实现类在`django.db.backends.mysql.base`模块中），但均继承自`django.db.backends.base.base.BaseDatabaseWrapper`。

参考[文章](https://www.the5fire.com/reduce-db-conn-with-django-persistent-connection.html)，Django 会把当前线程建立的若干个`DatabaseWrapper`对象存放在`ThreadLocal`中，并在每次请求开始和结束时进行以下过程：

```python
def close_old_connections(**kwargs):
    for conn in connections.all():
        conn.close_if_unusable_or_obsolete()

```

方法`close_if_unusable_or_obsolete`定义在`BaseDatabaseWrapper`中：

```python
    def close_if_unusable_or_obsolete(self):
        """
        Closes the current connection if unrecoverable errors have occurred,
        or if it outlived its maximum age.
        """
        if self.connection is not None:
            # If the application didn't restore the original autocommit setting,
            # don't take chances, drop the connection.
            if self.get_autocommit() != self.settings_dict['AUTOCOMMIT']:
                self.close()
                return

            # If an exception other than DataError or IntegrityError occurred
            # since the last commit / rollback, check if the connection works.
            if self.errors_occurred:
                # MySQL 的判断逻辑时调用 ping 方法，如果出现异常则连接不可用，需要关闭。
                if self.is_usable():
                    self.errors_occurred = False
                else:
                    self.close()
                    return

            if self.close_at is not None and time.time() >= self.close_at:
                # 连接建立时，Django 读取数据库配置中的 CONN_MAX_AGE 参数，
                # 如果不为 None (连接永久有效)，则取当时时间 + CONN_MAX_AGE，作为连接过期的时间。
                self.close()
                return
```

可见，对于一个线程，如果其数据库连接没有出现异常（除了`DataError`和`IntegrityError`），则 Django 不会实际 ping 数据库，而只会根据配置中的`CONN_MAX_AGE`决定是否需要关闭连接。

然而根据上述逻辑进行实验时，却出现了奇怪的现象：

- settings 中配置 CONN_MAX_AGE 为 600
- view 的逻辑：

```python
class Test(APIView):

    def get(self, request):
        print(threading.get_ident())  # 打印当前线程ID
        list(models.Foo.objects.all())  # 实际访问数据库
        return Response()
```

- 使用 gunicorn 启动，worker 数量为2：

```
gunicorn db_test.wsgi:application -w 2 -b 0.0.0.0:8000 -k gevent
```

- 在建立数据库连接的位置 print（修改`BaseDatabaseWrapper`的`get_new_connection`方法）：

```python
    def get_new_connection(self, conn_params):
        conn = Database.connect(**conn_params)
        conn.encoders[SafeText] = conn.encoders[six.text_type]
        conn.encoders[SafeBytes] = conn.encoders[bytes]
        print("get_new_connection!", conn)  # 此处 print
        return conn
```

随后重复请求 view，输出结果如下：

```
4556769432  # worker1
get_new_connection! <_mysql.connection open to '127.0.0.1' at 7fe4051ca418>
4556769432
...  # 略去重复部分
4556769432
4556769432
4556769432
4556769432
4544925168  # worker2
get_new_connection! <_mysql.connection open to '127.0.0.1' at 7fe40685cc18>
4544925168
4544925168
4544925168
4544925168
4544925168
4556769432  # worker1
get_new_connection! <_mysql.connection open to '127.0.0.1' at 7fe405ab6618>
4556769432

```

可以看出：

1. 一开始请求被调度到 worker1，worker1 与数据库建立了连接**（符合预期）**
2. 接下来的请求仍被调度到 worker1，worker1 没有重新建立数据库连接**（符合预期）**
3. 一段时间后 gunicorn 将请求调度到 worker2，worker2 与数据库建立了连接**（符合预期）**
4. 之后新的请求再次被调度到 worker1，然而 worker1 又重新与数据库建立了连接**（不符合预期）**
