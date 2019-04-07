# 第5部分 模块与包

## 第22章 模块

模块是最高级别的程序组织单元，它将程序代码和数据封装起来以便重用。

每个Python源文件都是一个模块。

模块可以由`import`语句和`from`语句，以及内置函数`imp.reload`进行处理：

- `import`：使客户端（导入者）以一个整体获得一个模块。
- `from`：允许客户端从一个模块文件中获取特定的变量名。
- `imp.reload`：在不终止Python程序的情况下，提供了一种重新载入模块文件代码的方法。

### 22.1 为什么使用模块

在一个模块文件顶层定义的所有的变量名都成了被导入的模块对象的属性。也就是说，导入操作给予了对模块的全局作用域中的变量名的读取权。

在模块导入时，模块文件的全局作用域变成了模块对象的命名空间。

从抽象的视角来看，模块至少有3个角色：

- 代码重用

- 系统命名空间的划分

- 实现共享服务和数据

    

### 22.2 Python程序架构

一个程序就是一个模块的系统。它有一个顶层脚本文件（启动后运行程序）以及多个模块文件（用来导入工具库）。

***脚本*** 和 ***模块*** 都是包含了Python语句的文本文件，尽管在模块中的语句通常都是创建之后使用的对象。

Python 标准库提供了一系列的预先编写好的模块。

#### 如何组织一个程序
一般来说，一个Python程序包括了多个含有Python语句的文本文件。程序是作为一个主体的、顶层的文件来构造的。这个顶层的主体文件可能配合有多个模块文件来提供支持。

在Python中，顶层文件包含了程序的主要的控制流程。模块文件就是工具的库。

在Python中，一个文件导入了一个模块来获得这个模块定义的工具的访问权，这些工具被认作是这个模块的属性。

#### 导入和属性

下图是一个包含有三个文件的Python程序的草图。文件a是顶层文件，在运行时将会从上至下执行其中的语句；文件b和文件c是模块，通常模块中的语句并不直接运行。

![](_static/images/Part_5_Modules_and_Packages.assets/1553614087306.png)

导入的概念贯穿了Python，任何文件都能从任何其他文件中导入其工具。例如，模块a导入了模块b，模块b导入了模块c，模块c又导入了其他模块。这就形成了 ***导入链*** 。

#### 标准库模块
Python自带了很多实用的莫夸，成为标准链接库。这些模块包含了平台不相关的常见程序设计任务。


### 22.3 import如何工作
在Python中，***导入*** 操作其实是运行时的运算。

程序第一次`import`指定文件时，会执行3个步骤：
1. **搜索模块文件**：找到`import`语句所引用的模块文件。
2. **编译（可选）**：Python会检查文件的时间戳，如果发现字节码文件比源文件旧，会自动重新编译字节码；否则，跳过编译步骤。如果Python在搜索路径上只发现了字节码文件，而没有源文件，就会直接加载字节码文件。
3. **运行**：import操作的最后步骤是执行模块的字节码。文件中所有语句会从头至尾依次执行。

> 注意：以上3个步骤仅会在模块第一次导入时进行；在这之后，重复导入相同模块时，会跳过这3个步骤，而只提取内存中已加载的模块对象。

> 因为导入操作的最后一步（第3步）实际上是执行文件的程序代码，所以如果模块文件中任何顶层代码确实做了什么实际的工作，你就会在导入时看见其结果。

从技术上讲，Python把载入的模块存储到一个名为`sys.modules`的字典中，并在一次导入操作的开始检查该表。如果模块不存在，将会执行这3个步骤。

> 实际上,如果想要看看已经导入了哪些模块,可以导入`sys`模块，并打印`list(sys.modules.keys())`。

### 22.4 字节码文件：`__pycache__` in Python 3.2+

第五版，暂略


### 22.5 模块搜索路径

`sys.path`是Python的模块搜索路径，它由以下4个路径组件构成：
1. 程序的主目录
2. PYTHONPATH目录（如果已经设置）：如果设置了PYTHONPATH，Python会从左至右搜索PYTHONPATH环境变量中所有的目录。
3. 标准链接库目录
4. 任何`.pth`文件的内容（如果存在的话）：Python允许用户把有效的目录在后缀名为`.pth`的文本文件中一行一行地列出目录，然后依次对其中的目录进行搜索。
5. 第三方扩展的`Lib/site-packages`目录：Python会自动添加其标准库的`site-packages`子目录到模块搜索路径。

Python在程序启动时，会自动根据以上4个路径组件对`sys.path`进行配置。同时，也可以手动对`sys.path`进行调整，以修改模块搜索路径。

#### 配置搜索路径


#### 搜索路径的变动


#### `sys.path`列表
可以通过打印`sys.path`列表来查看搜索路径的实际配置。

在导入模块时，Python会由左至右搜索这列表中的每个目录。

#### 模块文件选择
记住，文件名的后缀（例如，`.py`或`.pyc`）是刻意从import语句中省略的。

Python会选择在搜索路径中第一个符合导入文件名的文件。

> 如果在相同目录中找到`b.py`和`b.so`，会发生什么事？
>
> 答：建议在同一目录中保持模块名唯一！

#### 导入钩子（import hook）和ZIP文件
使用导入钩子（import hook）刻意重新定义 Python 中 import 操作所做的事。例如，使用钩子可以在导入时自动解压ZIP文件并归档文件。

更多细节参考Python标准库中关于内置函数`__import__`的说明。

#### 优化字节码文件（Optimized byte code files）
Python 也支持优化字节码文件`.pyo`。这种文件在创建和执行时要加上`-O`命令行标志。这种字节码文件比普通的`.pyc`字节码文件运行速度稍快一点（通常快5%），但不常使用。

> Python的第三方扩展通常使用标准链接库中的`distutils`工具来自动安装，所以不需要路径设置，就能使用它们的代码。



---


## 第23章 模块编码基础
### 23.1 模块的创建

任何保存有Python源代码，且以`.py`为后缀名的文本文件，都被认为是Python模块。所以保存Python源代码到以`.py`为后缀名的文本文件就是在创建模块。

因为模块名在Python程序中会变成变量名，因此，模块的命名应遵循变量名的命名规则。


### 23.2 模块的使用
#### import语句
import语句使用一个变量名引用整个模块对象，所以必须通过模块名称来得到该模块的属性：
```python
>>> import module1               # Get module as a whole (one or more)
>>> module1.printer('Hello world!')      # Qualify to get names
Hello world!
```

#### from语句
from语句会把变量名复制到另一个作用域，所以它就可以让我们直接在脚本中使用复制后的变量名，而不需要通过模块：
```python
>>> from module1 import printer # Copy out a variable (one or more)
>>> printer('Hello world!') # No need to qualify name
Hello world!
```

#### from \* 语句
使用`*`时，会取得模块顶层所有赋了值的变量名的拷贝。
```python
>>> from module1 import * # Copy out _all_ variables
>>> printer('Hello world!')
Hello world!
```

#### import和from是赋值语句
就像`def`一样，`import`和`from`是可执行的语句，而不是编译期间的声明，而且它们可以嵌套在`if`测试中，出现在函数`def`之中等，直到Python程序执行到这些语句时才会进行解析。

和`def`一样，`import`和`from`都是隐性的赋值语句：
- `import`将整个模块对象赋值给一个变量名。
- `from`将一个或多个变量名赋值给另一个模块中同名的对象。

之前讨论过的关于赋值语句方面的内容，也适用于模块的导入。例如，以from复制的变量名会变成对共享对象的引用。思考下面的small.py模块。
```python
x = 1
y = [1, 2]
```

```python
% python
>>> from small import x, y # Copy two names out
>>> x = 42 # Changes local x only
>>> y[0] = 42 # Changes shared mutable in place
```

此处，x并不是一个共享的可变对象，但y是。导入者中的变量名y和被导入者都引用相同的列表对象，所以在其中一个地方的修改，也会影响另一个地方的这个对象。
```python
>>> import small # Get module name (from doesn't)
>>> small.x # Small's x is not my x
1
>>> small.y # But we share a changed mutable
[42, 2]
```

#### 跨文件变量名的改变
```python
% python
>>> from small import x, y # Copy two names out
>>> x = 42 # Changes my x only
```
以from复制而来的变量名和其来源的的文件之间是没有关系的。为了实际修改另一个文件中的全局变量名，必须使用import：
```python
>>> import small # Get module name
>>> small.x = 42 # Changes x in other module
```

> 注意：这与前一小节中对`y[0]`的修改是不同的。这里修改了一个对象`small`，而不是一个变量名。


#### import和from的等价性
from只是把变量名从一个模块复制到另一个模块，并不会对模块名本身进行赋值。所以我们需要在from后执行import语句，来获取模块的变量名。从概念上来讲，一个像这样的from语句：
```python
from module import name1, name2 # Copy these two names out (only)
```
与下面这些语句是等效的：
```python
import module # Fetch the module object
name1 = module.name1 # Copy names out by assignment
name2 = module.name2
del module # Get rid of the module name
```

#### from语句潜在的陷阱
`from module import *`形式会把一个命名空间融入到另一个，所以会使得模块的命名空间的分割特性失效。


### 23.3 模块的命名空间
简而言之，模块就是命名空间，而存在于模块之内的变量名就是模块对象的属性。

#### 文件生成命名空间
那么，文件是如何变为命名空间的呢？简而言之，在模块文件顶层（也就是不在函数或类的主体内）每一个赋值了的变量名都会变成该模块的属性。

- **模块语句会在首次导入时执行**。
- **顶层的赋值语句（例如，`=`和`def`）会变为模块的属性**。
- **模块的命名空间能够通过属性`__dict__`或`dir(Module)`获取**：由导入而建立的模块的命名空间是字典。可通过模块对象相关联的内置属性`__dict__`来读取，也能通过`dir`函数查看。
- **模块是一个独立的作用域（本地变量就是全局变量）**：在模块导入后，模块文件的作用域就变成了模块对象的属性的命名空间。

#### 命名空间字典：`__dict__`

第五版，暂略

#### 属性名的点号运算

在Python中，可以使用点号运算语法`object.attribute`获取任意的object的attribute属性。点号运算是一个表达式，传回和对象相匹配的属性名的值。

以下是点号运算的规则：

- 点号运算：`X.Y`是指在当前范围内搜索X，然后搜索对象X之中的属性Y。
- 多层点号运算：`X.Y.Z`指的是寻找对象`X`之中的变量名`Y`，然后再找对象`X.Y`之中的`Z`。
- 点号运算的通用性：点号运算可用于任何具有属性的对象，如模块、类、C扩展类型等。


#### 导入和作用域

导入操作不会赋予被导入文件中的代码对上层代码（进行导入操作的文件）的可见度，即，被导入文件无法看见进行导入文件内的变量名。更确切的说法：

- 函数绝对无法看见其他函数内的变量名，除非它们从物理上处于这个函数内。
- 模块程序代码绝对无法看见其他模块内的变量名，除非明确地进行导入。

例如，有一个模块`moda.py`：

```python
X = 88
def f():
    global X
    X = 99
```

有另一个模块`modb.py`：

```python
X = 11

import modea
modea.f()
print(X, modea.X)
```

执行`modb.py`时，`moda.f`修改模块`moda`中的`X`，而不是`modb`中的`X`。`moda.f`的全局作用域一定是其所在的文件，无论这个函数是由哪个文件调用的：

```python
% python modeb.py
11 99
```




#### 命名空间的嵌套

利用属性的点号运算路径，有可能深入到任意嵌套的模块中并读取模块属性。

例如，有一个模块`mod3.py`如下：

```python
X = 3
```

另一个模块`mod2.py`如下：

```python
X = 2
import mod3

print(X, end=' ') # My global X
print(mod3.X) # mod3's X
```

还有一个模块`mod1.y`如下：

```python
X = 1
import mod2

print(X, end=' ') # My global X
print(mod2.X, end=' ') # mod2's X
print(mod2.mod3.X) # Nested mod3's X
```

利用`mod2.mod3.X`变量名路径，就可以深入到所导入的`mod2`内嵌套了的`mod3`。结果就是`mod1`可以看见三个文件内的`X`，因此，可以读取这三个全局范围。

### 23.4 重载模块

`imp`模块的`reload`函数会强制已加载的模块的代码重新载入并重新执行。此文件中新的代码的赋值语句会在适当的地方修改现有的模块对象。

> In Python 3.7.2, the `imp` module is deprecated in favour of `importlib`; see the module's documentation for alternative uses.

**为什么要这么麻烦去重载模块？`reload`函数可以修改程序的一些部分，而无须停止整个程序。因此，利用`reload`函数，可以立即看到对组件的修改的效果。**

#### reload基础

与`import`和`from`不同的是：

- `reload`是Python中的内置函数，而不是语句。
- 传给`reload`的必须是已经成功导入的模块对象，而不是变量名。
- `reload`在Python 3.0中位于`imp`模块之中，并且必须先导入。

当调用`reload`时，Python会重读模块文件的源代码，重新执行其顶层语句。

**`realod`会在适当的地方修改模块对象，而不会删除并重建模块对象。因此，程序中任何引用该模块对象的地方，自动会受到`reload`的影响：**

- `reload`会在模块当前命名空间内执行模块文件的新代码。
- 文件中顶层赋值语句会使得变量名换成新值。
- 重载会影响所有使用`import`读取了模块的客户端。
- 重载只会对以后使用`from`的客户端造成影响。





---

## 第24章 模块包

除了模块名之外，导入也可以指定目录路径。Python代码的目录就称为 ***包*** ，因此，这类导入就称为 ***包导入***。

实际上，包导入是把计算机上的目录变成另一个Python命名空间，而属性则对应于目录中所包含的子目录和模块文件。

### 24.1 包导入基础

```python
import dir1.dir2.mod
```

```python
from dir1.dir2.mod import x
```

假设，目录`dir1`所在的目录（即`dir1`的父目录）是`dir0`，则目录`dir0`必须存在于`sys.path`模块搜索路径中。

#### `__init__.py`包文件

使用包导入必须遵循一条约束：包导入语句的路径中的每个目录内都必须存在`__init__.py`这个文件，否则导入包会失败。

也就是说，上面的例子中：

- 目录`dir1`和`dir2`内都必须包含`__init__.py`这个文件。
- 而目录`dir1`的父目录`dir0`则不需要包含`__init__.py`文件；如果包含，也会被忽略。
- 目录`dir0`必须存在于模块搜索路径`sys.path`中。

结果就是，这个例子的目录结构应该是这样的：  

```
root@localhost# tree dir0 -v
dir0
└── dir1
    ├── __init__.py
    └── dir2
        ├── __init__.py
        └── mod.py

2 directories, 3 files
root@localhost#
```

`__init__.py`文件可以包含Python程序代码，就像普通模块文件一样，但也可以是空文件。

通常情况下，`__init__.py`文件扮演了包初始化的钩子、替目录产生模块命名空间以及使用目录导入时实现`from *`（即`from xxx import *`）行为的角色：

- **包的初始化**：Python首次导入某个目录时，会自动执行该目录下`__init__.py`文件中的所有程序代码。所以，该文件可以放置包内文件所需要初始化的代码。
- **模块命名空间初始化**：
- **`from *`语句的行为**：作为一个高级功能，你可以在`__init__.py`文件内使用`__all__`列表来定义包（目录）以`from *`形式导入时应该导入什么。如果没有设定`__all__`，`from *`语句不会自动加载嵌套于该目录内的子模块，而是只加载该目录的`__init__.py`文件中赋值语句定义的变量名（包括该文件中程序代码明确导入的子模块）。



### 24.2 包导入实例
有如下目录结构：
```
root@localhost# tree dir0 -v
dir0
└── dir1
    ├── __init__.py
    └── dir2
        ├── __init__.py
        └── mod.py

2 directories, 3 files
root@localhost#
```

其中文件`dir1\__init__.py`包含如下内容：
```python
# dir1\__init__.py
print('dir1 init')
x = 1
```

其中文件`dir1\dir2\__init__.py`包含如下内容：
```python
# dir1\dir2\__init__.py
print('dir2 init')
y = 2
```

其中文件`dir1\dir2\mod.py`包含如下内容：
```python
# dir1\dir2\mod.py
print('in mod.py')
z = 3
```

将`dir1`目录的父目录`dir0`添加入模块搜索路径：
```python
Python 3.7.2 (default, Mar 25 2019, 20:38:07)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path.append('/mnt/c/dir0')
```

开始导入包（目录）中的模块：
```python
>>> import dir1.dir2.mod      # 第一次导入时，会遍历包（目录），并运行各个包（目录）中的__init__.py
dir1 init
dir2 init
in mod.py
>>> import dir1.dir2.mod         # 重复导入则不会运行__init__.py
>>> import imp
__main__:1: DeprecationWarning: the imp module is deprecated in favour of importlib; see the module's documentation for alternative uses
>>> imp.reload(dir1)              # 使用reload函数重载包（目录）dir1
dir1 init
<module 'dir1' from '/mnt/c/dir0/dir1/__init__.py'>
>>>
>>> imp.reload(dir1.dir2)          # 使用reload函数重载包（目录）dir2
dir2 init
<module 'dir1.dir2' from '/mnt/c/dir0/dir1/dir2/__init__.py'>
>>> imp.reload(dir1.dir2.mod)      # 使用reload函数重载模块mod
in mod.py
<module 'dir1.dir2.mod' from '/mnt/c/dir0/dir1/dir2/mod.py'>
```

也可以单独导入某个包（目录）：
```python
>>> import dir1         # 首次单独导入包（目录）时，只会触发该目录的__init__.py
dir1 init
>>> import dir1.dir2    # 首次单独导入包（目录）时，只会触发该目录的__init__.py
dir2 init
>>> import dir1.dir2.mod   # 首次导入包中的模块也会执行该模块
in mod.py
>>> import dir1         # 重复导入包（目录），则不会触发执行该目录的__init__.py
>>> import dir1.dir2    # 重复导入包（目录），则不会触发执行该目录的__init__.py
>>> import dir1.dir2.mod   # 重复导入则不会执行该模块
>>>
```

包（目录）的重载不会遍历其子包（目录）：
```python
>>> import imp
__main__:1: DeprecationWarning: the imp module is deprecated in favour of importlib; see the module's documentation for alternative uses
>>> imp.reload(dir1)
dir1 init
<module 'dir1' from '/mnt/c/dir0/dir1/__init__.py'>
>>>
>>> imp.reload(dir1.dir2)
dir2 init
<module 'dir1.dir2' from '/mnt/c/dir0/dir1/dir2/__init__.py'>
>>> imp.reload(dir1.dir2.mod)
in mod.py
<module 'dir1.dir2.mod' from '/mnt/c/dir0/dir1/dir2/mod.py'>
```

> Since Python 3.4, the `imp` module is deprecated in favour of `importlib`; see the module's documentation for alternative uses.

注意：从Python 3.4开始，`imp`模块已经被标记为 **is pending deprecation**。在Python 3.7.2中已被标记为 **deprecated**，建议使用`importlib`模块的`reload`函数。
```python
root@LEGEND-PC:/mnt/c# python
Python 3.7.2 (default, Mar 25 2019, 20:38:07)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path.append('/mnt/c/dir0')
>>> import dir1.dir2.mod
dir1 init
dir2 init
in mod.py
>>> import importlib
>>> importlib.reload(dir1)
dir1 init
<module 'dir1' from '/mnt/c/dir0/dir1/__init__.py'>
>>> importlib.reload(dir1.dir2)
dir2 init
<module 'dir1.dir2' from '/mnt/c/dir0/dir1/dir2/__init__.py'>
>>> importlib.reload(dir1.dir2.mod)
in mod.py
<module 'dir1.dir2.mod' from '/mnt/c/dir0/dir1/dir2/mod.py'>
>>>
```

导入后，`import`语句内的路径会变成脚本的嵌套对象路径。如，`mod`是对象，嵌套在对象`dir2`中，而`dir2`又嵌套在对象`dir1`中：
```python
>>> dir1
<module 'dir1' from '/mnt/c/dir0/dir1/__init__.py'>
>>> dir1.dir2
<module 'dir1.dir2' from '/mnt/c/dir0/dir1/dir2/__init__.py'>
>>> dir1.dir2.mod
<module 'dir1.dir2.mod' from '/mnt/c/dir0/dir1/dir2/mod.py'>
```

实际上，路径中的每个目录名称都变成赋值了模块对象的变量，而模块对象的命名空间则是由该目录内的`__init__.py`文件中所有赋值语句进行初始化的。
```python
>>> dir1.x    # dir1.x引用了变量x，x是在dir1/__init__.py中赋值的
1
>>> dir1.dir2.y   # dir1.dir2.y引用了变量y，y是在dir1/dir2/__init__.py中赋值的
2
>>> dir1.dir2.mod.z  # mod.z引用的变量z则是在mod.py内赋值的
3
```

#### 包对应的from语句和import语句
```python
>>> from dir1.dir2 import mod # Code path here only
dir1 init
dir2 init
in mod.py
>>> mod.z # Don't repeat path
3
>>> from dir1.dir2.mod import z
>>> z
3
>>> import dir1.dir2.mod as mod # Use shorter name (see Chapter 25)
>>> mod.z
3
>>> from dir1.dir2.mod import z as modz # Ditto if names clash (see Chapter 25)
>>> modz
3
```


### 24.3 为什么使用包导入

在较大的程序中，包程序让导入更具信息性，并可以作为组织工具，简化模块的搜索路径，而且可以解决模糊性。

当有多个同名模块时，将模块保存到不同目录中，并将其创建为包，可以在导入时保证同名模块的唯一性。

### 24.4 包的相对导入

#### Python 3.0中的变化

略

#### 相对导入基础知识

在 Python 3.X 和 Python 2.6 中，`from`语句可以使用前面的点号`.`来指定，它们需要位于同一包中的模块（所谓的***包相对导入***），而不是位于模块导入搜索路径上某处的模块（即所谓 ***绝对导入***）。也就是说：

- 在 Python 3.X 和 Python 2.6中，我们可以使用`from`语句前面的点号来表示，导入应该性对于外围的包。这样的导入将只是在包的内部搜索，并且不会搜索位于导入搜索路径`sys.path`中某处的同名模块。
- 在Python 3.X中，不带点号的导入默认是绝对的。在缺少任何特殊的点号语法的时候，Python忽略包目录自身（即导入搜索路径的相对部分），并在`sys.path`搜索路径上进行绝对查找。

例如，在Python 3.X中，导入与当前文件位于同一包下的`spam`模块：

```python
from . import spam
```

导入与包含这条导入语句的文件位于同一包下的模块`spam`中的变量名`name`：

```python
from .spam import name
```


#### 为什么使用相对导入

#### 相对导入的作用域

- **相对导入只适用于包内导入**：这种功能的模块搜索路径修改只针对位于包内的模块文件中的`import`语句，即包内导入（intrapackage imports）。
- **相对导入只适用于`from`语句**：这一功能的新语法只适用于`from`语句，而不适用于`import`语句。其特点为，一个以一个或多个点号开头的`from`语句中的模块名。模块名包含嵌入的点号，但没有以点号开头，这样的导入是包导入，而不是相对导入。

#### 模块查找规则总结

**关于包和相对导入，Python 3.X中的模块搜索可以总结为如下：**

- 简单模块名（例如，`A`）通过搜索`sys.path`路径列表上的每个目录来查找，从左到右进行。这个列表由系统模块设置和用户配置组成。
- 包就是目录，这些目录中存在一个特殊的`__init__.py`文件的Python模块。这使得可以在导入操作中使用像`A.B.C`这样的目录路径语法。例如，导入`A.B.C`，目录`A`位于相对于`sys.path`的普通模块导入搜索，目录`B`是`A`中的另一个包（子目录），`C`是`B`中的一个模块或者其他可导入项。
- 在一个包的文件中，普通`import`和`from`语句就像其他地方的导入一样，使用相同的`sys.path`搜索规则。然而，在包中使用`from`语句和开头的点号的导入操作是相对于包的。也就是说，只有包目录会被检查，并且普通的`sys.path`查找不会被使用。例如，`from . import A`，模块搜索被限制在包含该`from`语句出现的文件的目录之中。


#### 相对导入的应用
##### 在包之外导入
`/tmp/code/`目录下有一个和标准库模块`string.py`同名的模块文件，其内容如下：
```python
print('string' * 8)
```

如果`/tmp/code/`为当前工作目录，并想在此情况下导入模块`string`，则当前工作目录下的`string.py`模块就会被导入，因为模块搜索路径中的第一条是当前工作目录（CWD）：
```python
root@localhost:/tmp/code# python
Python 3.7.2 (default, Mar 25 2019, 20:38:07)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import string
stringstringstringstringstringstringstringstring
>>> string
<module 'string' from '/tmp/code/string.py'>
>>>
```



**注意：如果不在作为包的一部分的模块文件中，是不允许使用相对导入语法的。**

```python
root@LEGEND-PC:/tmp/code# python
Python 3.7.2 (default, Mar 25 2019, 20:38:07)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from . import string
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: cannot import name 'string' from '__main__' (unknown location)
```

由于目录`/tmp/code`不包含`__init__.py`文件，所以此目录不是包，也就不允许使用针对包的相对导入语法。


##### 包内的导入


#### Pitfalls of Package-Relative Imports: Mixed Use
第五版，暂略


### 24.5 Python 3.3 Namespace Packages
第五版，暂略

---


## 第25章 高级模块话题
### 25.1 模块设计概念


### 25.2 在模块中隐藏数据
正如我们所看到的，Python模块会导出其文件顶层所赋值的所有变量名。在Python中，无法声明哪个变量名在模块外可见，哪个变量名在模块外不可见。实际上，如果客户端想的话，是没有办法阻止客户端修改一个模块中的变量名的。

在Python中，模块内的数据隐藏是一种惯例，而不是一种语法约束。

#### 最小化`from *`的破坏：`_X`和`__all__`
有种特殊的情况，把下划线放在变量名前面（例如`_X`），可以防止客户端使用`from *`语句导入模块时，把其中的这些变量名复制出去。

> 下划线不是“私有”声明。你还是可以使用其他导入形式看见并修改这类变量名，例如，使用`import`语句。

例如，有如下模块文件`unders.py`：
```python
# unders.py
a, _b, c, _d = 1, 2, 3, 4
```

可以看到，通过`from *`语句无法将变量名`_b`和`_d`导入：
```python
Python 3.7.2 (default, Mar 25 2019, 20:38:07)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from unders import *         # Load non _X names only
>>> a, c
(1, 3)
>>> _b
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name '_b' is not defined
>>> _d
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name '_d' is not defined
>>>
```

但使用`import`语句可以导入所有变量名：
```python
>>> import unders         # But other importers get every name
>>> unders._b
2
```


此外，也可以在模块顶层把变量名的字符串列表赋值给变量`__all__`，以达到类似于`_X`下划线命名惯例的隐藏效果。例如，有如下模块文件`alls.py`：
```python
# alls.py
__all__ = ['a', '_c'] # __all__ has precedence over _X
a, b, _c, _d = 1, 2, 3, 4
```

使用`from *`语句，只有`__all__`列表中的变量名会被导入：
```python
>>> from alls import * # Load __all__ names only
>>> a, _c
(1, 3)
>>> b
NameError: name 'b' is not defined
```

但使用`import`语句可以导入所有变量名：
```python
>>> from alls import a, b, _c, _d # But other importers get every name
>>> a, b, _c, _d
(1, 2, 3, 4)
>>> import alls
>>> alls.a, alls.b, alls._c, alls._d
(1, 2, 3, 4)
```

就像`_X`惯例一样，`__all__`列表只对`from *`语句这种形式有效，它并不是私有声明。


### 25.3 启用未来的语言特性：`__future__`


### 25.4 混合使用方式：`__name__`和`__main__`


### 25.5 实例：Dual Mode Code


### 25.6 修改模块搜索路径


### 25.7 import语句和from语句的as扩展


### 25.8 实例：模块即对象


### 25.9 用名称字符串导入模块


### 25.10 实例：过渡性模块重载


### 25.11 模块陷阱