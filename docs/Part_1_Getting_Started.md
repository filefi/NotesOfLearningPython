# 第1部分 使用入门
## 第一章 问答环节

### 为什么使用Python
- 软件质量：Python更注重可读性、一致性和软件质量。
- 提高开发者效率：简洁的语法、动态类型、无需编译、内置工具包等特性大大提高开发效率。
- 程序的可移植性：Python程序几乎可以不做任何修改就运行在所有主流计算机平台上。
- 标准库的支持：Python内置众多预编译并可移植的功能模块。
- 组件集成：Python脚本可通过灵活的集成机制轻松地与应用程序的其他部分进行通信。Python代码可以调用C和C++的库，也可以被C和C++所调用，可以与Java组件集成，可以与COM和.NET等框架进行通信。
- 享受乐趣：简单易用使得Python编程成为一种乐趣而非重复劳动。


### Python的缺点：慢

- 目前Python的标准实现方式是将源代码的语句编译为字节码的形式，之后再将字节码解释运行。
- Python已经过很多次优化，在处理某个文件或构建一个GUI界面等任务时，Python代码会立即发送至Python解释器内部已经编译过的C代码，以提高效率。
- Python开发效率带来的效益往往比执行速度带来的损失更重要。
- 在需要更快执行速度的工作领域，可以通过分离一部分需要优化速度的代码，将其转换为编译好的扩展，并在整个系统中使用Python脚本将这部分扩展连接起来。典型例子：NumPy，它采用双语言混编策略，将Python变为一个高效并简单易用的科学计算编程工具。

### 谁在使用Python
- YouTube
- NASA
- 豆瓣
- 知乎
- ......


### 使用Python可以做些什么
- 系统编程
- GUI编程
- Internet编程
- 组件集成：Python可以通过C/C++系统进行扩展，并能够嵌套C/C++系统的特性，使其成为一种万能胶水语言，可以脚本化处理其他系统和组件的行为。
- 数据库编程
- 快速原型：对于Python程序来说，使用Python或C编写的组件看起来都是一样的，所以可以利用Python做系统原型设计，之后再将组件移植到C/C++等编译语言上。
- 数值计算和科学计算编程
- 游戏、图像、人工智能、XML、机器人等


### Python如何获得支持
- Python拥有一个庞大且活跃的开发社区。
- Python开发者使用一个源代码控制系统在线协同地工作（git？）。修改遵从一个正式PEP（Python Enhancement Proposal）协议并且必须经过Python的扩展回归测试系统。
- 一个非正式的组织PSF（Python Software Foundation，Python软件基金会）负责组织会议并处理知识产权的问题。
- 世界各地举办了大量的Python会议，O'Reilly的OSCON和PSF的PyCon是其中最大的会议。


### Python有哪些技术上的优点

#### 面向对象
- Python是一门面向对象语言
- Python的OOP特性使它可以对C++、Java和C#的类型进行子类的定制。
- Python既支持面向对象变成也支持面向过程变成。

#### 免费
- Python的使用和分发是完全免费的。
- 如果你愿意，甚至可以销售它的源代码。

#### 可移植
- Python的标准实现是有可移植的ANSI C编写的，可以在目前所有主流平台上编译和运行。

#### 功能强大
- **动态类型**：Python在运行过程中随时跟踪对象的类型，不需要在代码中进行复杂的类型和大小的声明。
- **自动内存管理**：Python自动进行对象分配，并在对象不再使用时对其进行销毁（垃圾回收）。
- **大型程序支持**：Python包含模块、类和异常等工具，允许你使用OOP重用和定制代码。
- **内置对象支持**：Python提供了常用的数据结构作为语言基本的组成部分。
- **内置工具**：Python自带了许多强大的标准工具。
- **库工具**：Python预置了许多预编码的库工具。
- **第三方工具**

#### 可混合
Python可以以多种方式与其他语言编写的组件“粘接”在一起，例如Python的C语言API可以让Python灵活地调用C程序。

#### 简单易用
- 简单
- 小巧
- 灵活

#### 简单易学
- 简单的核心语法

### Python和其他语言比较起来怎么样
- 比Tcl强大
- 比Perl更简洁的语法和更简单的设计
- 比Java更简单、更易于使用
- 比C++更简单、更易于使用，但通常不与C++竞争
- 比Visual Basic更强大也更具跨平台特性
- 比PHP更易懂且用途更广
- 比Ruby更加成熟

---

## 第2章 Python如何运行程序

### Python解释器简介
- 安装到机器上的Python包含2个最小组件：*解释器* 和 支持的*库*。
- *解释器* 是一种让Python程序（脚本、源文件、字节码）运行起来的程序。
- 根据不同情况，解释器可采用C实现或Java实现等。

### 程序执行

#### 程序员视角

- 一个Python程序就是一个包含Python语句的文本文件。
- Python从头至尾按照顺序一个接一个地运行文件中的语句。
- Python文件是以.py作为其扩展名。

#### Python的视角
1. 字节码编译
2. Python虚拟机解释运行字节码

##### 字节码编译
- 当程序执行时，Python内部会先将源代码编译成与平台无关的字节码形式。
- 必须纯文本的源代码，字节码的运行速度要快得多。
- 如果Python有写入权限，那么它将把程序的字节码保存为以.pyc为扩展名的文件。
- 如果在上次运行程序之后没有对源代码进行修改，则下次运行时，Python将会跳过字节码编译过程直接加载.pyc文件；反之，如果修改了源代码，下次运行时Python会首先重新编译字节码。

##### Python虚拟机（PVM）
一旦程序编译为字节码，之后的字节码发送到Python虚拟机上来执行。

### 执行模块的变体
#### Python实现的替代者
- CPython
- Jython
- IronPython
- PyPy
- Stackless Python

#### 执行优化工具
CPython、Jython和IronPython都是通过同样的方式实现Python语言的，即通过把源代码编译为字节码，然后在适合的虚拟机上执行这些字节码。
然而，其他的系统，如Psyco及时编译器（JIT）和Shedskin C++转换器，则试着优化基本执行模块。

#### 冻结二进制文件
通过一些第三方工具，将Python程序转换为可执行程序（在Python世界中，被称为冻结二进制文件）。
冻结二进制文件能够将程序的字节码、PVM以及任何程序所需要的支持文件捆绑在一起形成一个单独的可执行二进制程序。这个程序可以轻松地向用户分发。

目前主要有3种工具能够生成冻结二进制文件：
- py2exe（windows下使用）
- PyInstaller（支持Windows、Linux/UNIX、Mac）
- freeze

#### 其他执行选项
- Stackless Python
- Cython是一种混合语言，它为Python代码结合了调用C函数以及使用变量、参数和类属性的C类型声明的能力。Cython代码可以编译成使用Python/C API的C代码，随后可以完整地编译。



---

## 第3章 如何运行程序

### 交互提示模式下编写代码
最简单的运行Python程序的办法就是在Python交互命令行中输入这些程序。

#### 交互地运行代码
- Python交互对话刚开始时将会打印两行信息文本，然后显示等待输入新的Python语句或表达式的提示符`>>>`。
- 交互模式会自动答应输出表达式的结果。
```python
Python 3.4.3 (default, Nov 12 2018, 22:25:49)
[GCC 4.8.4] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> print('hello world!')
hello world!
>>> print(2**8)
256
>>>
```



#### 为什么使用交互提示模式

- 实验：由于代码是立即执行的，交互模式是很好的学习和实验环境。
- 测试：交互模式是一个测试程序组件的地方。

#### 使用交互提示模式
注意事项：
- 只能输入Python语句

- 在交互提示模式中会自动打印结果，但源文件则不会

- 留意提示符的变换和复合语句的缩进

- 在交互提示模式中，用一个空行结束复合语句

    

### 系统命令行和文件
为了能够永久的保存程序，需要在文件中写入代码，这样的文件通常叫做***模块***。
***模块***是一个包含了Python语句的简单文本文件。
一旦编写完成，可以让Python编译器多次运行，且可以以多种方式运行。

#### 使用命令行运行文件
```
C:\tmp> python test.py
```

#### 使用命令行和文件的注意事项
- 注意Windows上的默认扩展名
- 在源文件中import模块文件时，不需要指定扩展名，比如模块文件名为`script.py`，在导入此模块时，则为`import script`。
- 交互模式下会自动输出，但源文件模式下则需要使用`print()`来进行输出。

#### UNIX可执行脚本(#!)
如果在Python、Linux及其他的UNIX类系统上使用Python，可以将Python代码编写为可执行脚本，就像使用Shell语言编写的`csh`或`sh`脚本一样。
注意事项如下：
- 第一行以`#!`开始，其后紧跟Python解释器的路径，或者使用`#!/usr/bin/env python`让系统通过环境变量自动查找python虚拟机的路径。
- 必须给予脚本可执行权限。

```python
#!/usr/bin/env python

print("hello world!")

```

### 点击文件图标
在Windows下，注册表使通过点击图标打开文件变得很容易。
当Python程序文件点击打开时Python自动注册为所运行的那个程序。

#### 使用input函数暂停脚本
在Windows下以窗口模式运行Python模块时，程序飞快运行完后，窗口会立即关闭，这样不利于查看输出，可以在程序末尾添加一个`input`函数来中断程序。



### 模块导入和重载

每个以扩展名`.py`结尾的Python源代码文件都是一个模块。其他的文件可以通过导入一个模块读取这个模块的内容。从本质上讲，导入就是载入另一个文件，并能够读取那个文件的内容。

正因为如此，**导入文件是运行文件的另一种方式。但只有在第一次导入时会运行，再次重复导入操作并不会再次运行文件。**

如果想要在Python同一会话中再次运行文件（不停止和重新启动会话），在Python 2.x中可以使用内置的`reload`函数，此函数在Python 3.X中被移到了`imp`模块中。

> 注意：`reload`函数是不可传递的，重载一个模块时，不会重载此模块所导入的任何模块。

```python
# python3.x 需要先从模块imp中导入reload函数
from imp import reload
reload(script)
```

> 注意：用一个`from`载入的模块名字不会通过一个reload直接更新，但是用一条`import`语句访问的名字则会。如果你的模块名字似乎不会在一次重载后改变，尝试使用`import`和`module.attribute`名字引用。

#### 模块的显要特性：属性

导入模块将得到模块文件中在顶层所定义的所有变量名。

一个模块文件的变量名可以通过2个Python语句读取: `import`和`from import`，也可以通过`reload`函数调用。

比如有一个名为 `mymodule.py` 的模块文件，其内容为：

```python
title = "The Meaning of Life"
```

使用`import`语句将模块作为一个整体载入，得到具有属性的模块，并使用 `module.attribute` 方式来获取模块的属性：

```python
root@localhost:/tmp# python
>>> import mymodule
>>> print(mymodule.title)
The Meaning of Life
>>>
```

使用 `from import` 语句从模块文件中**复制**变量到当前模块中：

```python
root@localhost:/tmp# python
>>> from mymodule import title
>>> print(title)
The Meaning of Life
>>>
```

从技术上讲，`from import` 复制了模块的属性，以便属性能够成为接收者的直接变量。因此，能够直接以变量名 `title` （一个变量）引用导入字符串变量而不是 `mymodule.title` （一个属性引用）。

无论使用的是`import`语句还是`from import`语句去执行导入操作，模块文件中的所有语句都会被执行，并且在顶层文件中得到了所导入组件的变量名的访问权。

现在有一个Python脚本文件`test.py`，其内容如下：
```python
a = 'dead'
b = 'parrot'
c = 'sketch'
print(a, b, c)
```

直接运行`test.py`：
```python
root@LEGEND-PC:/tmp# python test.py
('dead', 'parrot', 'sketch')
```

在交互模式下导入`test.py`：

- 注意到导入模块后，模块便自动运行了，打印输出了结果。
- 注意在同一环境中再次使用from import导入该模块，该模块没有再次执行！

```python
root@LEGEND-PC:/tmp# python3
Python 3.4.3 (default, Nov 12 2018, 22:25:49)
[GCC 4.8.4] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import test
dead parrot sketch
# 注意到导入模块后，模块便自动运行了，打印输出了结果。
>>> test.a
'dead'
>>> test.b
'parrot'
>>> test.c
'sketch'
>>> from test import a,b,c
# 注意到导入模块后，模块便自动运行了，打印输出了结果。
>>> b
'parrot'
>>> c
'sketch'
>>> a
'dead'
>>> b,c
('parrot', 'sketch')
>>>
```

使用`dir()`函数可以查看所导入的模块的所有属性名的列表。其中以双下划线开头并结尾的变量名是由Python预定义的内置变量名，对于解释器来说有特定的意义。
```python
>>> dir(test)
['__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'a', 'b', 'c']
>>>
```

##### 模块和命名空间

每个模块文件都是一个独立完备的命名空间，可以解决变量或属性命名冲突的问题，不同模块文件中的变量名即使相同，也不会冲突。

> 注意：`from import` 语句把变量从一个文件复制到另一个文件，这可能会导致在导入的文件中相同名称的变量被覆盖（并且，发生这种情况时不会发出警告）。



### 使用exec运行模块文件

`exec`函数不会导入模块，而是每次都重新运行模块文件的最新版本，

```python
exec(open('myscript.py').read())
```

注意：`exec()`的工作机制就好像是在调用它的地方粘贴了代码，所以和`from import`语句一样，有可能造成同名变量的覆盖。例如，下面例子中变量a,b,c的值都被exec()调用所覆盖了。
```python
root@LEGEND-PC:/tmp# python3
Python 3.4.3 (default, Nov 12 2018, 22:25:49)
[GCC 4.8.4] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> a = 111
>>> b = 222
>>> c = 333
>>> exec(open('test.py').read())
dead parrot sketch
>>> # 可以看到变量a,b,c的值都被exec()调用所覆盖了
```

Python 2.X还包含一个内置函数`execfile`可以直接运行给定模块文件名的模块：

```python
# python 2.x包含内置函数execfile
execfile('myscript.py')
```



### IDLE用户界面

IDLE提供了做Python开发的GUI，而且它是Python系统的一个标准并免费的部分。

由于IDEL是使用Tkinter GUI工具包开发的Python程序,可以在几乎任何Python平台上运行.

#### 使用IDLE

- 当保存文件时,必须明确地添加`.py`
- 通过选择在文本编辑窗口Run-> Run Module运行脚本，而不是通过交互模式的导入和重载。
- 你只需要重载交互测试的模块。
- 可以对IDLE的配置进行自定义。
- IDEL中没有清屏选项。

#### 高级IDLE工具

除了基本的编辑和运行功能，IDLE还提供了很多高级的特性，包括指向点击（point-and-click）程序调试和对象浏览器。

### 其他的IDE

一些最常用的IDE：

- Eclipse和PyDev
- Komodo
- NetBeans IDE Python版
- PythonWin
- PyCharm

### 其他启动选项

除了在命令行中运行、点击图标、import、exec、使用IDLE等方法，还有其他运行Python的方法，其中大多数都有专门或有限的用途。

- 嵌入式调用
- 冻结二进制的可执行性
- 文本编辑器启动的选择
- 其他的启动选择

> **调试Python代码**
>
> 这里快速介绍调试Python程序时的常用策略：
>
> - 阅读出错消息并修改标记的行和文件
> - 插入`print`语句
> - 使用IDE GUI调试器
> - 使用pdb命令行调试器

