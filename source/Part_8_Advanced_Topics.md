# 第8部分 高级话题

## 第37章 Unicode和字节字符串

### 37.1 Python 3.X 中字符串的变化

Python 2.X的`str`和`unicode`类型已经融入了Python 3.X的`str`和`bytes`类型，并且增加了一种新的可变的类型`bytearray`。



### 37.2 字符串基础

#### 字符编码方式

字节和字符串之间的来回转换由两个术语定义：

- ***编码*** 是根据一个想要的编码名称，把一个字符串翻译为其原始字节形式。
- ***解码*** 是根据其编码名称，把一个原始字节串翻译为字符串形式的过程。

对于Python程序员来说，编码指定为包含了编码名的字符串。Python带有大约100种不同的编码，参见Python库参考可以找到一个完整的列表。导入`encodings`模块并运行`help(encodings)`可以查看这些编码名称。



#### Python的字符串类型

Python 3.X中有3种字符串对象类型：

- `str`表示Unicode文本，用于文本数据。
- `bytes`表示二进制数据。
- `bytearray`是一种可变的`bytes`类型。

这3种字符串类型都支持类似的操作集，但是它们都有不同的角色。

##### **`str`类型**

Python 3.X的 `str` 类型定义了一个**不可变的字符序列**，它可能是像ASCII这样的每个字符一个字节的常规文本，或者是像UTF-8 Unicode这样可能包含多字节字符的字符集文本。

##### **`bytes`类型**

Python 3.X实现了字符串和Unicode的结合，但很多程序仍然需要处理图像、声音，以及用来与设备接口的打包数据，或者想要用Python的`struct`模块处理C程序。因此，为了支持真正的二进制数据的处理，还引入了一种新的类型`bytes`。

`bytes`类型定义为一个**8位整数的不可变序列**，表示绝对的字节值。该类型支持几乎所有`str`类型的操作。

##### **`bytearray`类型**

`bytearray`类型是`bytes`类型的一个变体，它是可变的并且**支持原处修改**。它支持`str`和`bytes`所支持的常见的字符串操作，以及和列表相同的很多原处修改操作（例如，append和extend方法，以及向索引赋值）。



#### 文本和二进制文件

**Python 3.X现在对文本文件和二进制文件之间做了一个明显的独立于平台的区分：**

- **文本文件：** 当一个文件以文本模式打开的时候，读取其数据会自动将其内容解码（每个平台一个默认的或一个提供的编码名称），并且将其返回为一个`str`，写入会接受一个`str`，并且在将其传输到文件之间自动编码它。文本模式的文件还支持统一的行尾转换和额外的编码特定参数。根据编码名称，文本文件也自动处理文件开始处的字节顺序标记序列。
- **二进制文件：** 通过在内置的`open`调用的模式字符串参数添加一个`b`，以二进制模式打开一个文件的时候，读取其数据不会以任何方式解码它，而是直接返回其内容raw并且未经修改，作为一个`bytes`对象；写入类似地接受一个`bytes`对象，并且将
  其传送到文件中而未经修改。二进制模式文件也接受一个`bytearray`对象作为写入
  文件中的内容。

**以何种模式打开一个文件将决定脚本使用何种类型的对象来表示其内容：**

- 如果正在处理图像文件，其他程序创建的、而且必须解压的打包数据，或者一些设
  备数据流，则使用`bytes`和二进制模式文件处理它更合适。如果想要更新数据而不
  在内存中产生其副本，也可以选择使用`bytearray`。
- 如果你要处理的内容实质是文本的内容，例如程序输出、HTML、国际化文本或
  CSV或XML文件，可能要使用`str`和文本模式文件。

注意：内置函数`open`的模式字符串参数（函数的第二个参数）在Python 3.0中变得至关重要，因为其内容不仅指定了一个文件处理模式，而且暗示了一个Python对象类型。例如，模式`rb`、`wb`和`rb+`暗示`bytes`，而`r`、`w+`和`rt`暗示`str`。

### 37.3 编写基础字符串

#### Python 3.X中的字符串常量

Python 3.X中，所有当前字符串常量形式，'xxx',"xxx"和三引号字符串块，都产生一个`str`；在它们任何一种前面添加一个`b`或`B`，则会创建一个`bytes`。

```python
C:\code> C:\python33\python
>>> B = b'spam'                   # 3.X bytes literal make a bytes object (8-bit bytes)
>>> S = 'eggs'                    # 3.X str literal makes a Unicode text string
>>> type(B), type(S)
(<class 'bytes'>, <class 'str'>)
>>> B                             # bytes: sequence of int, prints as character string
b'spam'
>>> S
'eggs'
```

`bytes`对象实际上是较小的整数的一个序列，尽管它尽可能地将自己的内容打印成字符：

```python
>>> B[0], S[0] # Indexing returns an int for bytes, str for str
(115, 'e')
>>> B[1:], S[1:] # Slicing makes another bytes or str object
(b'pam', 'ggs')
>>> list(B), list(S)
([115, 112, 97, 109], ['e', 'g', 'g', 's']) # bytes is really 8-bit small ints
```

就像`str`一样，`bytes`对象是不可修改的：

```python
>>> B[0] = 'x' # Both are immutable
TypeError: 'bytes' object does not support item assignment
>>> S[0] = 'x'
TypeError: 'str' object does not support item assignment
```

`bytes`前缀对于单引号，双引号和三引号字符串常量都有效：

```python
>>> # bytes prefix works on single, double, triple quotes, raw
>>> B = B"""
... xxxx
... yyyy
... """
>>> B
b'\nxxxx\nyyyy\n'
```



##### Python 3.3 中的Python 2.X Unicode 常量

在Python 2.6中，为了兼容性而使用`b'xxx'`，但是它与`'xxx'`是相同的，并且产生一个`str`，并且，`bytes`只是`str`的同义词：

```python
C:\code> C:\python33\python
>>> U = u'spam' # 2.X Unicode literal accepted in 3.3+
>>> type(U) # It is just str, but is backward compatible
<class 'str'>
>>> U
'spam'
>>> U[0]
's'
>>> list(U)
['s', 'p', 'a', 'm']
```



#### 字符串类型转换

Python 3.X中，`str`和`bytes`类型对象不在表达式中自动地混合，并且当传递给函数的时候不会自动地相互转换。期待一个`str`对象作为参数的函数，通常不能接受一个`bytes`；反之亦然。

因此，Python 3.0基本上要求遵守一种类型或另一种类型，或者手动执行显式转换：

- `str.encode()`和`bytes(S, encoding)`把一个字符串转换为其raw bytes形式，并且
  在此过程中根据一个`str`创建一个`bytes`。
- `bytes.decode()`和`str(B, encoding)`把raw bytes转换为其字符串形式，并且在此
  过程中根据一个`bytes`创建一个`str`。

```python
>>> S = 'eggs'
>>> S.encode()                 # str->bytes: encode text into raw bytes
b'eggs'
>>> bytes(S, encoding='ascii') # str->bytes, alternative
b'eggs'
>>> B = b'spam'
>>> B.decode()                 # bytes->str: decode raw bytes into text
'spam'
>>> str(B, encoding='ascii')   # bytes->str, alternative
'spam'
```

```python
>>> import sys
>>> sys.platform                  # Underlying platform
'win32'
>>> sys.getdefaultencoding()      # Default encoding for str here
'utf-8'
>>> bytes(S)
TypeError: string argument without an encoding
>>> str(B)                        # str without encoding
"b'spam'"                         # A print string, not conversion!
>>> len(str(B))
7
>>> len(str(B, encoding='ascii')) # Use encoding to convert to str
4
```



### 37.4 编写 Unicode 字符串

Python的字符串常量支持"\xNN"十六进制字节值转义以及"\uNNNN"和"\UNNNNNNNN" Unicode转义。在Unicode转义中，第一种形式给出了4个十六进制位以编码1个2字节（16位）字符码，而第二种形式给出8个十六进制位表示4字节（32位）代码。

#### 编写ASCII文本

ASCII文本是一种简单的Unicode，存储为表示字符的字节值的一个序列：

```python
C:\misc> c:\python30\python
>>> ord('X')            # 'X' has binary value 88 in the default encoding
88
>>> chr(88)             # 88 stands for character 'X'
'X'
>>> S = 'XYZ'           # A Unicode string of ASCII text
>>> S
'XYZ'
>>> len(S)              # 3 characters long
3
>>> [ord(c) for c in S] # 3 bytes with integer ordinal values
[88, 89, 90]
```

#### 编写非ASCII文本

```python
>>> chr(0xc4) # 0xC4, 0xE8: characters outside ASCII's range
'Ä'
>>> chr(0xe8)
'è'
>>> S = '\xc4\xe8' # Single byte 8-bit hex escapes
>>> S
'Äè'
>>> S = '\u00c4\u00e8' # 16-bit Unicode escapes
>>> S
'Äè'
>>> len(S) # 2 characters long (not number of bytes!)
2
```

#### 编码和解码非ASCII文本

```python
>>> S = '\u00c4\u00e8'
>>> S
'Äè'
>>> len(S)
2
>>> S.encode('ascii')
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1:
ordinal not in range(128)
>>> S.encode('latin-1')      # One byte per character
b'\xc4\xe8'
>>> S.encode('utf-8')        # Two bytes per character
b'\xc3\x84\xc3\xa8'
>>> len(S.encode('latin-1')) # 2 bytes in latin-1, 4 in utf-8
2
>>> len(S.encode('utf-8'))
4
```

也可以从一个文件读入raw字节并且将其解码回一个Unicode字符串：

```python
>>> B = b'\xc4\xe8' # Text encoded per Latin-1
>>> B
b'\xc4\xe8'
>>> len(B) # 2 raw bytes, two encoded characters
2
>>> B.decode('latin-1') # Decode to text per Latin-1
'Äè'
>>> B = b'\xc3\x84\xc3\xa8' # Text encoded per UTF-8
>>> len(B) # 4 raw bytes, two encoded characters
4
>>> B.decode('utf-8') # Decode to text per UTF-8
'Äè'
>>> len(B.decode('utf-8')) # Two Unicode characters in memory
2
```

#### 转换编码

可以把一个字符串转换为不同于源字符集默认的一种编码，但必须显式地提供一个编码名称以进行编码和解码：

```python
>>> S = 'AÄBèC'
>>> S
'AÄBèC'
>>> S.encode() # Default utf-8 encoding
b'A\xc3\x84B\xc3\xa8C'
>>> T = S.encode('cp500') # Convert to EBCDIC
>>> T
b'\xc1c\xc2T\xc3'
>>> U = T.decode('cp500') # Convert back to Unicode
>>> U
'AÄBèC'
>>> U.encode() # Default utf-8 encoding again
b'A\xc3\x84B\xc3\xa8C'
```

记住，只有当手动编写非ASCII Unicode的时候，才必须用到特殊的Unicode和十六进制
字符转义

#### 源文件字符集编码声明

对于在脚本文件中编码的字符串，Python默认地使用UTF-8编码，但是，它允许我们通过包含一个注释来指明想要的编码，从而将默认值修改为支持任意的字符集。这个注释必须拥有如下的形式，并且在Python 2.X或Python 3.X中必须作为脚本的第一行或第二行出现：
```python
-*- coding: latin-1 -*-
```



### 37.5 使用 3.X `bytes` 对象

#### 方法调用

```python
>>> B = b'spam'             # b'...' bytes literal
>>> B.find(b'pa')
1
>>> B.replace(b'pa', b'XY') # bytes methods expect bytes arguments
b'sXYm'
>>> B.split(b'pa')          # bytes methods return bytes results
[b's', b'm']
>>> B
b'spam'
>>> B[0] = 'x'              # bytes objects are immutable
TypeError: 'bytes' object does not support item assignment
```

字符串格式化在Python 3.X中只对`str`有效，对`bytes`无效：

```python
>>> '%s' % 99
'99'
>>> b'%s' % 99
TypeError: unsupported operand type(s) for %: 'bytes' and 'int'
>>> '{0}'.format(99)
'99'
>>> b'{0}'.format(99)
AttributeError: 'bytes' object has no attribute 'format'
```

#### 序列操作

```python
>>> B = b'spam' # A sequence of small ints
>>> B # Prints as ASCII characters (and/or hex escapes)
b'spam'
>>> B[0] # Indexing yields an int
115
>>> B[-1]
109
>>> chr(B[0]) # Show character for int
's'
>>> list(B) # Show all the byte's int values
[115, 112, 97, 109]
>>> B[1:], B[:-1]
(b'pam', b'spa')
>>> len(B)
4
>>> B + b'lmn'
b'spamlmn'
>>> B * 4
b'spamspamspamspam'
```

#### 创建bytes对象的其他方式

```python
>>> B = b'abc'                    # Literal
>>> B
b'abc'
>>> B = bytes('abc', 'ascii')     # Constructor with encoding name
>>> B
b'abc'
>>> ord('a')
97
>>> B = bytes([97, 98, 99])       # Integer iterable
>>> B
b'abc'
>>> B = 'spam'.encode()           # str.encode() (or bytes())
>>> B
b'spam'
>>>
>>> S = B.decode()                # bytes.decode() (or str())
>>> S
'spam'
```

#### 混合字符串类型

```python
# Must pass expected types to function and method calls
>>> B = b'spam'
>>> B.replace('pa', 'XY')
TypeError: expected an object with the buffer interface
>>> B.replace(b'pa', b'XY')
b'sXYm'
>>> B = B'spam'
>>> B.replace(bytes('pa'), bytes('xy'))
TypeError: string argument without an encoding
>>> B.replace(bytes('pa', 'ascii'), bytes('xy', 'utf-8'))
b'sxym'

# Must convert manually in 3.X mixed-type expressions
>>> b'ab' + 'cd'
TypeError: can't concat bytes to str
>>> b'ab'.decode() + 'cd' # bytes to str
'abcd'
>>> b'ab' + 'cd'.encode() # str to bytes
b'abcd'
>>> b'ab' + bytes('cd', 'ascii') # str to bytes
b'abcd'
```



### 37.6 使用 3.X/2.6+ `bytearray` 对象

```python
# Creation in 3.X: text/binary do not mix
>>> S = 'spam'
>>> C = bytearray(S)
TypeError: string argument without an encoding
>>> C = bytearray(S, 'latin1') # A content-specific type in 3.X
>>> C
bytearray(b'spam')
>>> B = b'spam' # b'..' != '..' in 3.X (bytes/str)
>>> C = bytearray(B)
>>> C
bytearray(b'spam')
```

创建之后，可以像列表一样修改：

```python
# Mutable, but must assign ints, not strings
>>> C[0]
115
>>> C[0] = 'x' # This and the next work in 2.6/2.7
TypeError: an integer is required
>>> C[0] = b'x'
TypeError: an integer is required
>>> C[0] = ord('x') # Use ord() to get a character's ordinal
>>> C
bytearray(b'xpam')
>>> C[1] = b'Y'[0] # Or index a byte string
>>> C
bytearray(b'xYam')
```

处理`bytearray`对象可以使用字符串和列表的方法：

```python
# Mutable method calls
>>> C
bytearray(b'xYam')
>>> C.append(b'LMN') # 2.X requires string of size 1
TypeError: an integer is required
>>> C.append(ord('L'))
>>> C
bytearray(b'xYamL')
>>> C.extend(b'MNO')
>>> C
bytearray(b'xYamLMNO')
```

所有常见的序列操作和字符串方法都在`bytearray`上有效：

```python
# Sequence operations and string methods
>>> C
bytearray(b'xYamLMNO')
>>> C + b'!#'
bytearray(b'xYamLMNO!#')
>>> C[0]
120
>>> C[1:]
bytearray(b'YamLMNO')
>>> len(C)
8
>>> C.replace('xY', 'sp') # This works in 2.X
TypeError: Type str doesn't support the buffer API
>>> C.replace(b'xY', b'sp')
bytearray(b'spamLMNO')
>>> C
bytearray(b'xYamLMNO')
>>> C * 4
bytearray(b'xYamLMNOxYamLMNOxYamLMNOxYamLMNO')
```

最后，下面例子展示了`bytes`和`bytearray`对象是`int`的序列，而`str`对象是字符的序列：

```python
# Binary versus text
>>> B # B is same as S in 2.6/2.7
b'spam'
>>> list(B)
[115, 112, 97, 109]
>>> C
bytearray(b'xYamLMNO')
>>> list(C)
[120, 89, 97, 109, 76, 77, 78, 79]
>>> S
'spam'
>>> list(S)
['s', 'p', 'a', 'm']
```



### 37.7 使用文本和二进制文件

**文本模式意味着`str`对象，二进制模式意味着`bytes`对象：**

- ***文本模式文件*** 根据Unicode编码来解释文件内容，要么是平台的默认编码，要么是我们传递进的编码名。通过传递一个编码名来打开文件，我们可以强行进行Unicode文件的各种类型的转换。文本模型的文件也执行通用的行末转换：默认地，所有的行末形式映射为脚本中的一个单个的'\n'字符，而不管在什么平台上运行。正如前面所描述的，文本文件也负责阅读和写入在某些Unicode编码方案中存储文件开始处的字节顺序标记（Byte Order Mark，BOM）。
- ***二进制模式文件*** 不会返回原始的文件内容，而是作为表示字节值的整数的一个序列，没有编码或解码，也没有行末转换。

#### 文本文件基础

```python
C:\code> C:\python33\python
# Basic text files (and strings) work the same as in 2.X
>>> file = open('temp', 'w')
>>> size = file.write('abc\n') # Returns number of characters written
>>> file.close() # Manual close to flush output buffer
>>> file = open('temp') # Default mode is "r" (== "rt"): text input
>>> text = file.read()
>>> text
'abc\n'
>>> print(text)
abc
```

#### Python 3.X中的文本和二进制模式

Python 3.X文本模式：

```python
C:\code> C:\python33\python
# Write and read a text file
>>> open('temp', 'w').write('abc\n') # Text mode output, provide a str
4
>>> open('temp', 'r').read() # Text mode input, returns a str
'abc\n'
>>> open('temp', 'rb').read() # Binary mode input, returns a bytes
b'abc\r\n'
```

Python 3.X二进制模式：

```python
# Write and read a binary file
>>> open('temp', 'wb').write(b'abc\n') # Binary mode output, provide a bytes
4
>>> open('temp', 'r').read() # Text mode input, returns a str
'abc\n'
>>> open('temp', 'rb').read() # Binary mode input, returns a bytes
b'abc\n'
```

```python
# Write and read truly binary data
>>> open('temp', 'wb').write(b'a\x00c') # Provide a bytes
3
>>> open('temp', 'r').read() # Receive a str
'a\x00c'
>>> open('temp', 'rb').read() # Receive a bytes
b'a\x00c'
```

实际上，Python 3.X中的大多数API接受一个`bytes`，也允许一个`bytearray`：

```python
# bytearrays work too
>>> BA = bytearray(b'\x01\x02\x03')
>>> open('temp', 'wb').write(BA)
3
>>> open('temp', 'r').read()
'\x01\x02\x03'
>>> open('temp', 'rb').read()
b'\x01\x02\x03'
```

#### 类型和内容错误匹配

如果试图向一个文本文件写入一个`bytes`或者向二进制文件写入一个`str`，将会得到错误：

```python
# Types are not flexible for file content
>>> open('temp', 'w').write('abc\n') # Text mode makes and requires str
4
>>> open('temp', 'w').write(b'abc\n')
TypeError: can't write bytes to text stream
>>> open('temp', 'wb').write(b'abc\n') # Binary mode makes and requires bytes
4
>>> open('temp', 'wb').write('abc\n')
TypeError: can't write str to binary stream
```

在Python 3.X中，文本模式的输入文件需要一个`str`而不是一个`bytes`用于内容，因此，在Python 3.X中，没有方法把真正的二进制数据写入一个文本模式文件中；由于Python 3.X中的文本模式输入文件必须能够针对每个Unicode编码来解码内容，因此，没有办法在文本模式中读取真正的二进制数据：

```python
# Can't read truly binary data in text mode
>>> chr(0xFF) # FF is a valid char, FE is not
'ÿ'
>>> chr(0xFE)
UnicodeEncodeError: 'charmap' codec can't encode character '\xfe' in position 1...
>>> open('temp', 'w').write(b'\xFF\xFE\xFD') # Can't use arbitrary bytes!
TypeError: can't write bytes to text stream
>>> open('temp', 'w').write('\xFF\xFE\xFD') # Can write if embeddable in str
3
>>> open('temp', 'wb').write(b'\xFF\xFE\xFD') # Can also write in binary mode
3
>>> open('temp', 'rb').read() # Can always read as binary bytes
b'\xff\xfe\xfd'
>>> open('temp', 'r').read() # Can't read text unless decodable!
UnicodeEncodeError: 'charmap' codec can't encode characters in position 2-3: ...
```



### 37.8 使用Unicode文件





### 37.9 Python 3.X 中其他字符串工具的变化







---



## 第38章 管理属性







---



## 第39章 装饰器







---



## 第40章 元类







---



## 第41章 所有美好的事物

