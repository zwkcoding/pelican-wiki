# Python3 模块与包的导入
## 模块导入 
### 定义
- Python 模块(Module)，是**一个 Python 文件**，以 .py 结尾，包含了 Python 对象定义和Python语句。
- 避免函数名和变量名冲突
  -- 每个模块都是一个独立的名称空间，定义在这个模块中的函数，**把这个模块的名称空间当做全局名称空间**，这样我们在编写自己的模块时，就不用担心我们定义在自己模块中全局变量会在被导入时，与使用者的全局变量冲突
### 两种用法
#### import
- 第一次导入后就将模块名加载到内存了，后续的import语句仅是对已经加载大内存中的模块对象增加了一次引用，不会重新执行模块内的语句
- **首次** import导入模块干的事：
  - 为源文件(`my_moudle模块`)创建新的名称空间，在my_moudle中定义的函数和方法若是使用到了global时访问的就是这个名称空间。
  - 在新创建的命名空间中执行模块中包含的代码
  - 创建名字`my_moudle`来引用该命名空间
##### 导入多个模块
>import sys,os,re
#### From … import
- 不会把整个 `fib` 模块导入到当前的命名空间中，它只会将 `fib` 里的 fibonacci 单个引入到执行这个声明的模块的全局符号表。
- import导入模块干的事：
  - 直接拿到就是spam.py产生的名称空间中名字
- **避免使用** `from xxx import *`
- 容易跟当前文件的名称空间冲突
  - 解决办法：__all__来控制
  > __all__=['money','read1'] #这样在另外一个文件中用from spam import *就这能导入列表中规定的两个名字
  ---
  >如果my_moudle.py中的名字前加_,即_money，则from my_moudle import *,则money不能被导入


##### 导入多个模块
```python
from spam import (read1,
                  read2,
                  money)
```
#### AS 别名
编写代码来选择性地挑选读取模块
```python
if file_format == 'xml':
     import xmlreader as reader
elif file_format == 'csv':
    import csvreader as reader
data=reader.read_date(filename)
```
#### 模块首次加载的问题
考虑到性能的原因，每个模块只被导入一次,放入字典sys.module中，如果你改变了模块的内容，**你必须重启程序**，python不支持重新加载或卸载之前导入的模块。可以试试如下：

```python
import time,importlib
import aa
 
time.sleep(20)
# importlib.reload(aa)
aa.func1()
```

####  把模块当做脚本执行 

我们可以通过模块的全局变量__name__来查看模块名：
当做脚本运行：
\___name\___ 等于'__main__'

当做模块导入：
\___name\___= 模块名

作用：用来控制.py文件在不同的应用场景下执行不同的逻辑
if \___name\___ == '__main__':
```

#!/usr/bin/python3
# Filename: using_name.py

if __name__ == '__main__':
   print('程序自身在运行')
else:
   print('我来自另一模块')
运行输出如下：
#1、 
$ python using_name.py
程序自身在运行
#2、
$ python
>>> import using_name
我来自另一模块
>>>
```
### 模块的搜索路径
模块的查找顺序是：
1. 内存中已经加载的模块
2. 内置模块
3. 路径中包含的模块

执行的文件的**当前路径**就是sys.path
``` python
import sys
print(sys.path)
```

----

## 包导入 
### 定义
**无论是import形式还是from...import形式，凡是在导入语句中（而不是在使用时）遇到带点的，都要第一时间提高警觉：这是关于包才有的导入语法。.的左边必须是包**
> 凡是在导入时带点的，点的左边都必须是一个包。

简单来说，**包就是文件夹**，但该文件夹下必须存在 `__init__.py` 文件, 该文件的内容可以为空。`__init__.py`用于标识当前文件夹是一个包。

包是由**一系列模块**组成的集合。模块是处理某一类问题的**函数和类**的集合。

### 导入
```python
# import ...
import glance.db.models
glance.db.models.register_models('mysql') 

# from ... import ...
from glance.db import models
models.register_models('mysql')

from glance.db.models import  register_models
register_models('mysql')
```

### \___init\___.py文件
```python
#在__init__.py中定义
x=10

def func():
    print('from api.__init.py')

__all__=['x','func','policy']
```
此时我们在于`glance`同级的文件中执行`from glance.api import *`就导入__all__中的内容（versions仍然不能导入）。

### 绝对导入和相对导入
最顶级包`glance`是写给别人用的，然后在glance包内部也会有彼此之间互相导入的需求，这时候就有绝对导入和相对导入两种方式：

- **绝对导入**：以glance作为起始
- **相对导入**：用.或者..的方式最为起始（只能在一个包中使用，**不能用于不同目录内**）

例：我们在glance/api/version.py中想要导入glance/cmd/manage.py
```python
pwd: glance/api/version.py

#绝对导入
from glance.cmd import manage
manage.main()

#相对导入
from ..cmd import manage
manage.main()
```
**__注意__**：可以用import导入内置或者第三方模块（已经在sys.path中），但是要**绝对避免使用import来导入自定义包的子模块(没有在sys.path中)，应该使用from... import ...的绝对或者相对导入**,且包的相对导入只能用from的形式。

### 单独导入包
**避免单独导入包**，因为单独导入包名称时不会导入包中所有包含的所有子模块。

```python
#在与glance同级的test.py中
import glance
glance.cmd.manage.main()

'''
执行结果：
AttributeError: module 'glance' has no attribute 'cmd'

'''
```
__P_S__:`__all__`是用于控制`from...import *`

##后记
参考链接：
1. [【Python3】Python模块与包的导入](https://segmentfault.com/a/1190000009842139)
2. [江子川-模块和包](https://www.cnblogs.com/zhangningyang/p/7326092.html)：更全，更详细


