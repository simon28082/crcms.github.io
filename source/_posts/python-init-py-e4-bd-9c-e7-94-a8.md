---
title: 'Python: __init__.py 作用'
tags:
  - __init__.py
  - import
  - python
url: 115.html
id: 115
categories:
  - Python
date: 2018-06-08 14:46:21
---

\_\_init\_\_.py的作用
------------------

`__init__.py`主要是用来初始化 Python包（packages）的，它在模块加载最开始运行。 以下目录结构为示例：

    ├─test                                    
    │  ├─package_1
    │  │  ├─__init__.py 
    │  │  ├─m1.py    
    │  ├─package_2  
    │  │  ├─__init__.py 
    │  │  └─m2.py
    │  ├─test.py
    

其中包含两个包`package_1`,`package_2`

\_\_init\_\_.py中有什么
-------------------

如上目录结构，如果两个包中的`__init__.py`都为空的话，默认在`test.py`中我们只能执行如下：

    from package_2 import m2
    
    print(m2.m2s())
    
    

当想要执行

    import package_2
    print(package_2.m2.m2s())
    

或

    from package_2 import *
    print(m2.m2s())
    

则会抛出异常

    AttributeError: module 'package_2' has no attribute 'm2'
    

这是因为在`__init__.py`中并没有进行包提升

如何执行
----

此时以package_2包中的`__init__.py`为例 当我们在包中提升导入权限即可：

    from package_2.m2 import m2s
    

在test.py中，执行

    from package_2 import m2s
    
    print(m2s())
    

通常我们经常还会遇到 `from xxx import *` 我们只需要在**init**.py中加载**all**允许全部导出的模块即可，如在package_2的**init**.py中加入

    __all__ = ['m2']
    

test.py中执行

    from package_2 import *
    print(m2.m2s())
    

`__all__` 是一个列表变量，放的是你希望导入的模块的名字。 当 `__init__.py` 中定义了 `__all__` 变量时，import * 只能导入 `__all__`允许的模块