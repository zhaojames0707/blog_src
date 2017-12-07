---
title: "Django 项目性能分析工具"
author: "zhaojames0707"
date: 2017-12-07T12:18:45+08:00
draft: false
share: true
comments: true
tags: ["python", "django", "profiling"]
---

公司项目的IM应用，在人数较多的群聊中发信出现了性能问题，本文提供了分析服务性能的工具。

<!--more-->

**注意**: 本文提供的工具供性能分析/调试，并非用于生产环境。

### 1. 安装

需要安装 django-extensions

    pip install django-extensions

安装 Qcachegrind

    brew install graphviz
    brew install qcachegrind --with-graphviz

### 2. 添加 Django 配置

将 django-extensions 添加到 settings 的 INSTALLED_APPS 中

```python
INSTALLED_APPS = (
    ...
    'django_extensions',
    ...
)
```

### 3. 创建性能分析文件目录

启动服务后，每次请求均会生成性能分析文件(.prof)，根据实际情况生成目录

    mkdir /tmp/my-profile-data

### 4. 启动服务

使用 django-extensions 提供的 runprofileserver 命令启动服务，输出 kcachegrind 格式的性能分析文件，输出到 prof-path 指定的路径下

    python manage.py runprofileserver --kcachegrind --prof-path=/tmp/my-profile-data

### 5. 查看性能分析文件

假设生成的文件名为 foo.prof

    cd /tmp/my-profile-data
    qcachegrind foo.prof

效果如图

![screen_20171207_01](/images/screen_20171207_01.png)


#### 本文参考

- Django Wiki 文档: https://code.djangoproject.com/wiki/ProfilingDjango
