# 第7部分 异常和工具

## 第33章 异常基础

在Python中，异常会根据错误自动地被触发，也能由代码触发和截获。异常由4个语句处理：

- `try`/`except`：捕捉由Python或你引发的异常并恢复；
- `try`/`finally`：无论异常是否发生，执行清理行为；
- `raise`：手动在代码中触发异常；
- `assert`：有条件地在程序代码中触发异常；
- `with`/`as`：在Python 2.6及后续版本中实现的上下文管理器。

### 为什么使用异常

借助异常，可以在一个步骤内跳至异常处理器，终止开始的所有函数调用而进入异常管理器。在异常处理器中编写代码，来响应在适当时候引发的异常。

异常处理器（`try`语句）作为标记，当程序前进到某处代码时，产生异常，因而会使Python立即跳到那个标识，而放弃留下该标识之后所调用的任何激活的函数。

#### 异常的角色

在Python中，异常通常可以用于很多用途，其中最常见的几种角色：

- 错误处理
- 事件通知：异常也可用于发出有效状态的信号，而不需在程序传递结果标志位，或者刻意对其进行测试。例如，搜索的程序可能在失败时引发异常，而不是返回一个整数结果代码。
- 特殊情况处理：有时，发生了某种很罕见的情况，很难调整代码去处理。通常会在异常处理器中处理这些罕见的情况。
- 终止行为：`try`/`finally`语句可确保无论程序是否有异常，一定会进行需要的结束操作。
- 非常规控制流程：因为异常可以看做是一种高级的“goto”，它可以作为实现非常规的控制流程的基础。Python中没有“goto”语句，但是，异常有时候可以充当类似的角色。

### 异常处理：简明扼要

#### 默认异常处理器

```python
>>> def fetcher(obj, index):
        return obj[index]
>>> x = 'spam'
>>> fetcher(x, 3) # Like x[3]
'm'
>>> fetcher(x, 4) # Default handler - shell interface
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
File "<stdin>", line 2, in fetcher
IndexError: string index out of range
```

因为我们的代码没有刻意捕捉这个`IndexError`异常，所以它将会一直向上传递到程序顶层，并启用默认的异常处理器：打印标准出错消息，并立刻终止程序。这些消息包括引发的异常和堆栈跟踪：也就是异常发生时激活的程序行和函数清单。

#### 捕获异常

如果你不想要默认的异常行为，就需要把调用包装在`try`语句内，自行捕捉异常。

```python
>>> def catcher():
        try:
            fetcher(x, 4)
        except IndexError:
            print('got exception')
        print('continuing')
>>> catcher()
got exception
continuing
>>>
```

现在，当`try`代码块执行时触发异常，Python会自动跳至异常处理器（指出引发异常名称的`except`分句下面的代码）。在异常捕获和处理后，程序在捕捉了整个`try`语句后继续执行。

#### 引发异常

我们可以人为手动引发异常，直接执行`raise`语句：

```python
>>> raise IndexError
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
IndexError
```

同样的，手动引发的异常也可以捕获或忽略：

```python
>>> try:
        raise IndexError # Trigger exception manually
    except IndexError:
        print('got exception')

got exception
```

`assert`语句也可以用来有条件的引发异常：

```python
>>> assert False, 'Nobody expects the Spanish Inquisition!'
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
AssertionError: Nobody expects the Spanish Inquisition!
```



#### 用户自定义的异常

用户自定义的异常能够通过类编写，它继承自一个内置的异常类`Exception`。基于类的异常允许脚本建立异常类型、继承行为以及附加状态信息。

```python
>>> class AlreadyGotOne(Exception): pass # User-defined exception
>>> def grail():
...     raise AlreadyGotOne()            # Raise an instance
>>> try:
...     grail()
... except AlreadyGotOne:                # Catch class name
...     print('got exception')
...
got exception
>>>
```



#### 终止行为

`try`/`finally`的组合，可以定义一定会在最后执行时的收尾行为，无论`try`代码块中是否发生了异常：

```python
>>> def after():
        try:
            fetcher(x, 4)
        finally:
            print('after fetch')
        print('after try?')
>>> after()
after fetch
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
File "<stdin>", line 3, in after
File "<stdin>", line 2, in fetcher
IndexError: string index out of range
>>>
```

在使用某些类型的对象的时候，Python 2.6以上和Python 3.X提供了上下文管理器`with`/`as`语句来确保终止行为的发生，以作为一种`try`/`finally`的替代方法。

```python
>>> with open('lumberjack.txt', 'w') as file: # Always close file on exit
        file.write('The larch!\n')
```

**注意：这只在处理某些对象类型的时候才适用。因此`try`/`finally`是一种更加通用的终止结构。**





---



## 第34章 异常编码细节

### try/except/else语句

在Python 2.5之后，`try`/`except`/`else`和`try`/`finally`语句可以混合在一个`try`语句中。

`try`是复合语句，它的最完整的形式如下所示：

```python
try:
    statements               # Run this main action first
except name1:
    statements               # Run if name1 is raised during try block
except (name2, name3):
    statements               # Run if any of these exceptions occur
except name4 as var:
    statements               # Run if name4 is raised, assign instance raised to var
except:
    statements               # Run for all other exceptions raised
else:
    statements               # Run if no exception was raised during try block
```

在这个语句中，`try`首行底下的代码块代表此语句的主要动作：试着执行的程序代码。`except`子句定义`try`代码块内引发的异常的处理器。而`else`子句（如果编写了的话）则是提供没发生异常时要执行的代码。

#### try语句分句

编写`try`语句时，有一些分句可以在`try`语句代码块后出现。

从语法上来讲，`except`分句数目没有限制，但是应该只有一个`else`。

下表列出了所有可能的分句形式：

|   Clause form   |   Interpretation   |
| ---- | ---- |
|    `except:`    |    Catch all (or all other) exception types.      |
|    `except name:`    |    Catch a specific exception only.      |
|    `except name as value:`    |    Catch the listed exception and assign its instance.      |
|    `except (name1, name2):`    |    Catch any of the listed exceptions.      |
|    `except (name1, name2) as value:`    |    Catch any listed exception and assign its instance.      |
|    `else:`    |    Run if no exceptions are raised in the try block.      |
|    `finally:`    |    Always perform this block on exit.      |

以下是多个`except`分句的例子：

```python
try:
    action()
except NameError:
    ...
except IndexError:
    ...
except KeyError:
    ...
except (AttributeError, TypeError, SyntaxError):
    ...
else:
    ...
```

在这个例子中，如果`action`函数执行时引发了异常，Python会搜索第一个与异常名称相符的`except`。Python会从头到尾以及由左至右查看`except`子句，然后执行第一个项目的`except`下的语句。如果没有符合的，异常会向这个`try`以外传递。注意：在本例中，只有当`action`中没有发生异常时，`else`才会执行；如果发生了异常，但没有相符的`except`时，则不会执行。

空的`except:`子句可以捕获任何异常：

```python
try:
    action()
except:
    ...        # Catch all possible exceptions
```

Python 3.0引入了一个替代方法，捕获名为`Exception`的异常几乎与一个空的`except`具有相同的效果，除了与系统退出相关的异常：

```python
try:
    action()
except Exception:
    ...           # Catch all possible exceptions, except exits
```



#### try/else 分句

`else`子句让我们可以在不设置和检查布尔标志的情况下，知道`try`语句块中是否发生了异常。

```python
try:
    ...run code...
except IndexError:
    ...handle exception...
else:
    ...no exception occurred...
```

#### 例子：默认行为

如果没有手动捕捉异常，异常就会向上传递到Python进程的顶层，并执行Python默认异常处理逻辑，即终止执行中的程序，并打印标准出错消息。

```python
# bad.py

def gobad(x, y):
    return x / y
    
def gosouth(x):
    print(gobad(x, 0))
    
gosouth(1)
```

```bash
% python bad.py
Traceback (most recent call last):
File "bad.py", line 7, in <module>
gosouth(1)
File "bad.py", line 5, in gosouth
print(gobad(x, 0))
File "bad.py", line 2, in gobad
return x / y
ZeroDivisionError: division by zero
```



#### 例子：捕捉内置异常

```python
# kaboom.py

def kaboom(x, y):
    print(x + y)               # Trigger TypeError
try:
    kaboom([0, 1, 2], 'spam')
except TypeError:              # Catch and recover here
    print('Hello world!')
print('resuming here')         # Continue here if exception or not
```

```bash
% python kaboom.py
Hello world!
resuming here
Keep in mind that once
```

### 34.2 try / finally 语句



#### 例子：利用try/finally编写终止行为



### 统一 try/except/finally 语句



####  统一 try 语句语法



#### 通过嵌套合并 finally 和 except



#### 合并 try 的例子



### `raise`语句

#### 利用`raise` 传递异常



#### Python 3.X 异常链：raise from



### assert语句



#### 例子：收集约束条件（但不是错误）



### `with`/`as`上下文管理器（context manager）

#### 基本使用



#### 上下文管理协议





---



## 第35章 异常对象



## 第36章 异常的设计

