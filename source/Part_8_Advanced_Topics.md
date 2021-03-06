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

#### 在Python 3.X中读取和写入Unicode

在Python 3.X中，有两种办法可以把字符串转换为不同的编码：

- 用方法调用手动地转换
- 在文件输入输出上自动地转换

##### 手动编码

```python
C:\misc> c:\python30\python
>>> S = 'A\xc4B\xe8C' # 5-character string, non-ASCII
>>> S
'AÄBèC'
>>> len(S)
5
# Encode manually with methods
>>> L = S.encode('latin-1') # 5 bytes when encoded as latin-1
>>> L
b'A\xc4B\xe8C'
>>> len(L)
5
>>> U = S.encode('utf-8') # 7 bytes when encoded as utf-8
>>> U
b'A\xc3\x84B\xc3\xa8C'
>>> len(U)
7
```

##### 文件输入输出自动转换

###### 文件输出编码

```python
# Encoding automatically when written
>>> open('latindata', 'w', encoding='latin-1').write(S) # Write as latin-1
5
>>> open('utf8data', 'w', encoding='utf-8').write(S) # Write as utf-8
5
>>> open('latindata', 'rb').read() # Read raw bytes
b'A\xc4B\xe8C'
>>> open('utf8data', 'rb').read() # Different in files
b'A\xc3\x84B\xc3\xa8C'
```

###### 文件输入编码

```python
# Decoding automatically when read
>>> open('latindata', 'r', encoding='latin-1').read() # Decoded on input
'AÄBèC'
>>> open('utf8data', 'r', encoding='utf-8').read() # Per encoding type
'AÄBèC'
>>> X = open('latindata', 'rb').read() # Manual decoding:
>>> X.decode('latin-1') # Not necessary
'AÄBèC'
>>> X = open('utf8data', 'rb').read()
>>> X.decode() # UTF-8 is default
'AÄBèC'
```

###### 解码错误匹配

试图以文本模式打开一个真正的二进制数据文件，即便使用了正确的对象类型，也不可能在Python 3.X中有效：

```python
>>> file = open(r'C:\Python33\python.exe', 'r')
>>> text = file.read()
UnicodeDecodeError: 'charmap' codec can't decode byte 0x90 in position 2: ...
>>> file = open(r'C:\Python33\python.exe', 'rb')
>>> data = file.read()
>>> data[:20]
b'MZ\x90\x00\x03\x00\x00\x00\x04\x00\x00\x00\xff\xff\x00\x00\xb8\x00\x00\x00'
```

#### 在Python 3.X中处理BOM

一些编码方式在文件的开始处存储了一个特殊的字节顺序标记（BOM）序列，来指定数据的大小尾方式或声明编码类型。如果编码名暗示了BOM的时候，Python在输入和将其写入输出的时候都会忽略该标记，但是有时候必须使用一个特定的编码名称来迫使显式地处理BOM。

当用Python代码写入一个Unicode文件，我们需要一个更加显式的编码名称来强迫UTF-8中带有BOM——“utf-8”不会写入（或忽略）BOM，但“utf-8-sig”会这么做：

```python
>>> open('temp.txt', 'w', encoding='utf-8').write('spam\nSPAM\n')
10
>>> open('temp.txt', 'rb').read() # No BOM
b'spam\r\nSPAM\r\n'
>>> open('temp.txt', 'w', encoding='utf-8-sig').write('spam\nSPAM\n')
10
>>> open('temp.txt', 'rb').read() # Wrote BOM
b'\xef\xbb\xbfspam\r\nSPAM\r\n'
>>> open('temp.txt', 'r').read()
'ï>>¿spam\nSPAM\n'
>>> open('temp.txt', 'r', encoding='utf-8').read() # Keeps BOM
'\ufeffspam\nSPAM\n'
>>> open('temp.txt', 'r', encoding='utf-8-sig').read() # Skips BOM
'spam\nSPAM\n'
```

注意，尽管“utf-8”没有抛弃BOM，但不带BOM的数据可以用“utf-8”和“utf-8-sig”读取——如果你不确定一个文件中是否有BOM，使用后者进行输入：
```python
>>> open('temp.txt', 'w').write('spam\nSPAM\n')
>>> 10
>>> open('temp.txt', 'rb').read() # Data without BOM
>>> b'spam\r\nSPAM\r\n'
>>> open('temp.txt', 'r').read() # Any utf-8 works
>>> 'spam\nSPAM\n'
>>> open('temp.txt', 'r', encoding='utf-8').read()
>>> 'spam\nSPAM\n'
>>> open('temp.txt', 'r', encoding='utf-8-sig').read()
>>> 'spam\nSPAM\n'
```

#### Unicode文件名和流

第五版，暂略



### 37.9 Python 3.X 中其他字符串工具的变化

#### re模块匹配模块

```python
C:\code> C:\python33\python
>>> import re
>>> S = 'Bugger all down here on earth!' # Line of text
>>> B = b'Bugger all down here on earth!' # Usually from a file
>>> re.match('(.*) down (.*) on (.*)', S).groups() # Match line to pattern
('Bugger all', 'here', 'earth!') # Matched substrings
>>> re.match(b'(.*) down (.*) on (.*)', B).groups() # bytes substrings
(b'Bugger all', b'here', b'earth!')
```

但是，注意，像在其他的API中一样，我们不能在Python 3.0调用的参数中混合`str`和`bytes`类型：

```python
C:\code> C:\python33\python
>>> import re
>>> S = 'Bugger all down here on earth!'
>>> B = b'Bugger all down here on earth!'
>>> re.match('(.*) down (.*) on (.*)', B).groups()
TypeError: can't use a string pattern on a bytes-like object
>>> re.match(b'(.*) down (.*) on (.*)', S).groups()
TypeError: can't use a bytes pattern on a string-like object
>>> re.match(b'(.*) down (.*) on (.*)', bytearray(B)).groups()
(bytearray(b'Bugger all'), bytearray(b'here'), bytearray(b'earth!'))
>>> re.match('(.*) down (.*) on (.*)', bytearray(B)).groups()
TypeError: can't use a string pattern on a bytes-like object
```



#### Struct二进制数据模块

Python 3.X的`struct`模块，用来从字符串创建和提取打包的二进制数据，打包的数据只是作为`bytes`和`bytearray`对象显示。

下面是在Python的两个版本中的应用，它根据二进制类型声明把3个对象打包到一个字符串中（它们创建了一个4字节的整数、一个4字节的字符串和一个两字节的整数）：

```python
C:\code> C:\python33\python
>>> from struct import pack
>>> pack('>i4sh', 7, b'spam', 8) # bytes in 3.X (8-bit strings)
b'\x00\x00\x00\x07spam\x00\x08'
```

```python
C:\code> C:\python27\python
>>> from struct import pack
>>> pack('>i4sh', 7, 'spam', 8) # str in 2.X (8-bit strings)
'\x00\x00\x00\x07spam\x00\x08'
```



#### pickle对象序列化模块

`pickle`模块的Python 3.X版本总是创建一个`bytes`对象，而不管默认的或传入的“协议”（数据格式化层级）。我们通过使用该模块的`dumps`调用来返回一个对象的`pickle`字符串，从而查看这一点：

```python
C:\code> C:\python33\python
>>> import pickle # dumps() returns pickle string
>>> pickle.dumps([1, 2, 3]) # Python 3.X default protocol=3=binary
b'\x80\x03]q\x00(K\x01K\x02K\x03e.'
>>> pickle.dumps([1, 2, 3], protocol=0) # ASCII protocol 0, but still bytes!
b'(lp0\nL1L\naL2L\naL3L\na.'
```



#### XML解析工具

Python自身附带一个完整的XML解析工具包，所以它支持SAX和DOM解析模式，此外，还有一个名为ElementTree的包——这是特定于Python的专用于解析和构造XML的一个API。除了基本的解析，开源领域还提供了额外的XML工具，例如XPath、Xquery、XSLT，等等。







---



## 第38章 管理属性

### 38.1 为什么管理属性（Why Manage Attributes）

有时候，为了灵活性，需要编写了一个程序，这个程序可以直接使用一个`name`属性，并在之后按需对该属性进行修改。例如，在以某种方式设置或修改名称的时候，需要进行逻辑验证。

#### 插入在属性访问时运行的代码

最佳的解决方案是，如果需要的话，在属性访问时自动运行代码。当获取属性值以及在存储属性值对其进行验证或修改的时候，这样的工具允许脚本动态地计算属性值。主要包括以下4种实现技术：

- `__getattr__`和`__setattr__`方法，把未定义的属性获取和所有的属性赋值指向通用的处理器方法。
- `__getattribute__`方法，把所有属性获取都指向Python 2.6的新式类和Python 3.X的所有类中的一个泛型处理器方法。
- `property`内置函数，把特定属性访问定位到get和set处理器函数，也叫做 ***特性***（Property）。
- 描述符协议，把特定属性访问定位到具有任意get和set处理器方法的类的实例。



### 38.2 特性（Properties）

特性协议允许我们把一个特定属性的get和set操作指向我们所提供的函数或方法，使得我们能够插入在属性访问的时候自动运行的代码，拦截属性删除，并且如果愿意的话，还可为属性提供文档。

通过`property`内置函数来创建特性并将其分配给类属性。就像方法函数以及其他类属性一样，可以通过子类和实例继承属性。它们的访问拦截功能通过`self`实例参数提供。

一个 *特性（property）* 管理一个单个的、特定的属性；它允许我们控制访问和赋值操作，并且允许我们自由地把一个属性从简单的数据改变为一个计算，而不会影响已有的代码。

**特性** 和 **描述符** 有很大的关系，它们基本上是描述符的一种受限制的形式。

#### 基础知识

可以通过把一个内置函数的结果赋给一个类属性来创建一个特性：
```python
attribute = property(fget, fset, fdel, doc)
```

这个内置函数的任何参数都不是必需的，如果没有传递参数，所有参数的默认值为`None`。如果前面3个参数，`None`意味着不支持相应的操作，并且尝试使用会自动引发一个`AttributeError`异常。

当这3个参数被使用时，我们给`fget`传递一个函数来拦截属性的获取，给`fset`传递一个函数来进行赋值，给`fdel`传递一个函数来进行属性删除。技术上来讲，所有这3个参数可以接受任何可调用对象，包括类方法。当之后被调用时，`fget`函数返回计算过的属性值，`fset`和`fdel`无返回值（其实是返回`None`），并且所有这3个函数都可能引发异常来拒绝访问请求。

如果提供了`doc`参数，该参数接收一个该属性的文档字符串；否则，特性（property）会复制`fget`函数的文档字符串（通常该字符串为`None`）。

这个内置的函数调用返回一个特性对象，我们将它赋给在类的作用域中要管理的属性名称，它将被每个实例所继承。

#### 第一个例子

如下的类使用一个特性来记录对一个名为`name`的属性的访问，实际存储的数据名为`_name`，以便不会和特性搞混了：

```python
# prop-person.py

class Person:                       # Add (object) in 2.X
    def __init__(self, name):
        self._name = name
    def getName(self):
        print('fetch...')
        return self._name
    def setName(self, value):
        print('change...')
        self._name = value
    def delName(self):
        print('remove...')
        del self._name
    name = property(getName, setName, delName, "name property docs")
    
bob = Person('Bob Smith')           # bob has a managed attribute
print(bob.name)                     # Runs getName
bob.name = 'Robert Smith'           # Runs setName
print(bob.name)
del bob.name                        # Runs delName

print('-'*20)
sue = Person('Sue Jones')           # sue inherits property too
print(sue.name)
print(Person.name.__doc__)          # Or help(Person.name)
```

当这段代码运行的时候，两个实例继承了该特性，就好像它们是附加到其类的另外两个属性一样。然而，捕获了它们的属性访问：

```
c:\code> py −3 prop-person.py
fetch...
Bob Smith
change...
fetch...
Robert Smith
remove...
--------------------
fetch...
Sue Jones
name property docs
```

就像所有的类属性一样，实例和较低的子类都继承特性。如果我们把例子修改为如下所示，则输出是同样的。

```python
class Super:
    def __init__(self, name):
        self._name = name
    def getName(self):
        print('fetch...')
        return self._name
    def setName(self, value):
        print('change...')
        self._name = value
    def delName(self):
        print('remove...')
        del self._name
    name = property(getName, setName, delName, "name property docs")
    
class Person(Super):
    pass # Properties are inherited (class attrs)

bob = Person('Bob Smith')           # bob has a managed attribute
print(bob.name)                     # Runs getName
bob.name = 'Robert Smith'           # Runs setName
print(bob.name)
del bob.name                        # Runs delName

print('-'*20)
sue = Person('Sue Jones')           # sue inherits property too
print(sue.name)
print(Person.name.__doc__)          # Or help(Person.name)
```

关于继承，特性的方式和常规方法是一样的；由于它们能够访问`self`实例参数，所以能够访问实例状态信息和方法，而不用考虑子类的深度。

#### 计算的属性（Computed Attributes）

通常特性可以更多事情。例如，当获取属性的时候，动态地计算属性的值。下面的例子展示了这一点：

```python
class PropSquare:
    def __init__(self, start):
        self.value = start
    def getX(self):             # On attr fetch
        return self.value ** 2
    def setX(self, value):      # On attr assign
        self.value = value
    X = property(getX, setX)    # No delete or docs
    
P = PropSquare(3)  # Two instances of class with property
Q = PropSquare(32) # Each has different state information

print(P.X)         # 3 ** 2
P.X = 4
print(P.X)         # 4 ** 2
print(Q.X)         # 32 ** 2 (1024)
```

注意，我们已经创建了两个不同的实例——因为特性方法自动地接收一个`self`参数，所以它们都访问了存储在实例中的状态信息。

#### 使用装饰器编写特性

内置函数`property`可以充当一个装饰器，来定义一个函数，当获取一个属性的时候自动运行该函数：

```python
class Person:
    @property
    def name(self): ... # Rebinds: name = property(name)
```

运行的时候，装饰的方法自动传递给`property`内置函数的第一个参数。这其实只是创建一个特性并手动绑定属性名的一种替代语法：

```python
class Person:
    def name(self): ...
    name = property(name)
```

对于Python 2.6和3.0，property对象也有`getter`、`setter`和`deleter`方法，这些方法指定相应的特性访问器方法，以及返回特性自身的一个副本。我们也可以使用这些方法，通过装饰常规方法来指定特性的组成部分，尽管`getter`部分通常由创建特性自身的行为自动填充：

```python
# prop-person-deco.py

class Person:
    def __init__(self, name):
        self._name = name

    @property
    def name(self): # name = property(name)
        "name property docs"
        print('fetch...')
        return self._name

    @name.setter
    def name(self, value): # name = name.setter(name)
        print('change...')
        self._name = value

    @name.deleter
    def name(self): # name = name.deleter(name)
        print('remove...')
        del self._name

bob = Person('Bob Smith') # bob has a managed attribute
print(bob.name) # Runs name getter (name 1)
bob.name = 'Robert Smith' # Runs name setter (name 2)
print(bob.name)
del bob.name # Runs name deleter (name 3)

print('-'*20)
sue = Person('Sue Jones') # sue inherits property too
print(sue.name)
print(Person.name.__doc__) # Or help(Person.name)
```

实际上，这段代码等同于本小节的第一个示例——在这个例子中，装饰只是编写特性的一种替代方法。当运行这段代码时，结果是相同的：

```
c:\code> py −3 prop-person-deco.py
fetch...
Bob Smith
change...
fetch...
Robert Smith
remove...
--------------------
fetch...
Sue Jones
name property docs
```




### 38.3 描述符（Descriptors）

**描述符**提供了拦截属性访问的一种替代方法。实际上，**特性**是一种**描述符**。从技术上讲，`property`内置函数只是创建一个特定类型的描述符的一种简化方式，而这种描述符在属性访问时运行方法函数。

从功能上讲，描述符协议允许我们把一个特定属性的 get 和 set 操作指向我们提供的一个单独类对象的方法：它们提供了一种方式来插入在访问属性的时候自动运行的代码，并且它们允许我们拦截属性删除并且为属性提供文档（如果愿意的话）。



#### 基础知识

描述符作为单独的类编写，并且针对想要拦截的属性访问操作提供特定命名的访问器方法——当以相应的方式访问分配给描述符类实例的属性时，描述符类中的获取、设置和删除等方法自动运行：

```python
class Descriptor:
    "docstring goes here"
    def __get__(self, instance, owner): ...    # Return attr value
    def __set__(self, instance, value): ...    # Return nothing (None)
    def __delete__(self, instance): ...        # Return nothing (None)
```

带有任何这些方法的类都可以看作是描述符。当描述符的一个实例分配给另一个类的属性后，如果属性被访问，会自动调用它们。如果这些方法中的任何一个空缺，通常意味着不支持相应类型的访问。然而，和特性不同，省略一个`__set__`意味着允许这个描述符的属性名称在一个实例中被重新定义。要使得一个属性是只读的，我们必须定义`__set__`来捕获赋值并引发一个异常。



##### 描述符方法参数

所有3种描述符方法，都传递了描述符类实例（self）以及描述符实例所附加的客户类的实例（instance）。

`__get__`访问方法还额外地接收一个`owner`参数，指定了描述符实例要附加到的客户类。

```python
>>> class Descriptor:          # Add "(object)" in 2.X
        def __get__(self, instance, owner):
            print(self, instance, owner, sep='\n')
>>> class Subject:             # Add "(object)" in 2.X
        attr = Descriptor()    # Descriptor instance is class attr
>>> X = Subject()
>>> X.attr
<__main__.Descriptor object at 0x0281E690>
<__main__.Subject object at 0x028289B0>
<class '__main__.Subject'>
>>> Subject.attr
<__main__.Descriptor object at 0x0281E690>
None
<class '__main__.Subject'>
```

注意在第一个属性获取中自动传递到`__get__`方法中的参数，当获取`X.attr`的时候，就好像发生了如下的转换（尽管这里的`Subject.attr`没有再次调用`__get__`）：

```python
X.attr -> Descriptor.__get__(Subject.attr, X, Subject)
```



##### 只读描述符

和特性不同，使用描述符直接忽略`__set__`方法不足以让属性成为只读的，因为可以通过给实例属性赋一个新的值，从而选择性地覆盖类对象中的属性。

在下面的例子中，对`X.a`的属性赋值在实例对象`X`中存储了`a`，由此，隐藏了存储在类C中的描述符：

```python
>>> class D:
        def __get__(*args): print('get')
>>> class C:
        a = D()   # Attribute a is a descriptor instance
>>> X = C()
>>> X.a           # Runs inherited descriptor __get__
get
>>> C.a
get
>>> X.a = 99      # Stored on X, hiding C.a!
>>> X.a
99
>>> list(X.__dict__.keys())
['a']
>>> Y = C()
>>> Y.a           # Y still inherits descriptor
get
>>> C.a
get
```

要让基于描述符的属性成为只读的，捕获描述符类中的赋值并引发一个异常来阻止属性赋值——当要赋值的属性是一个描述符的时候，Python有效地绕过了常规实例层级的赋值行为，并且把操作指向描述符对象：

```python
>>> class D:
        def __get__(*args): print('get')
        def __set__(*args): raise AttributeError('cannot set')

>>> class C:
        a = D()

>>> X = C()
>>> X.a            # Routed to C.a.__get__
get
>>> X.a = 99       # Routed to C.a.__set__
AttributeError: cannot set
```

#### 第一个实例

如下代码定义了一个描述符，来拦截对其客户类中的名为`name`的一个属性的访问。其方法使用它们的`instance`参数来访问主体实例中的状态信息，其中指定了实际存储的名称字符串。

```python
# desc-person.py

class Name: # Use (object) in 2.X
    "name descriptor docs"
    def __get__(self, instance, owner):
        print('fetch...')
        return instance._name
    def __set__(self, instance, value):
        print('change...')
        instance._name = value
    def __delete__(self, instance):
        print('remove...')
        del instance._name
        
class Person: # Use (object) in 2.X
    def __init__(self, name):
        self._name = name
    name = Name()                       # Assign descriptor to class's attr
    
bob = Person('Bob Smith')               # bob has a managed attribute
print(bob.name)                         # Runs Name.__get__
bob.name = 'Robert Smith'               # Runs Name.__set__
print(bob.name)
del bob.name                            # Runs Name.__delete__

print('-'*20)
sue = Person('Sue Jones')               # sue inherits descriptor too
print(sue.name)
print(Name.__doc__)                     # Or help(Name)
```

**注意，我们必须像这样把描述符赋给一个类属性——如果赋给一个`self`实例属性，它将无法工作。**

当描述符的`__get__`方法运行的时候，它传递了3个对象来定义其上下文：

- `self`是`Name`类实例
- `instance`是`Person`类实例
- `owner`是`Person`类实例

当这段代码运行描述符的方法来拦截对该属性的访问的时候，和特性的版本具有相同的输出：

```
c:\code> py −3 desc-person.py
fetch...
Bob Smith
change...
fetch...
Robert Smith
remove...
--------------------
fetch...
Sue Jones
name descriptor docs
```

还和特性示例中相似，我们的描述符类实例是一个类属性，并且因此由客户类和任何子类的所有实例所继承。例如，如果我们把示例中的`Person`类修改为如下的样子，脚本的输出是相同的：

```python
class Super:
    def __init__(self, name):
        self._name = name
        name = Name()
class Person(Super): # Descriptors are inherited (class attrs)
    pass
```

还要注意到，当一个描述符类在客户类之外无用的话，将描述符的定义嵌入客户类之中，这在语法上是完全合理的。这里，我们的示例看上去就像使用一个嵌套的类：

```python
class Person:
    def __init__(self, name):
        self._name = name

    class Name:                  # Using a nested class
        "name descriptor docs"
        def __get__(self, instance, owner):
            print('fetch...')
            return instance._name
        def __set__(self, instance, value):
            print('change...')
            instance._name = value
        def __delete__(self, instance):
            print('remove...')
            del instance._name
    name = Name()
```

当按照这种方式编码时，`Name`变成了`Person`类声明的作用域中的一个局部变量，这样，它不会与类之外的任何名称冲突。但是，测试代码的最后一行必须改为从其新位置获取文档字符串：

```python
print(Person.Name.__doc__)     # Differs: not Name.__doc__ outside class
```



#### 计算的属性

描述符也可以用来在每次获取属性的时候计算它们的值。如下例子说明了这点：这里使用了一个描述符，从而在每次获取属性值的时候对其值自动求平方：

```python
# desc-computed.py

class DescSquare:
    def __init__(self, start):             # Each desc has own state
        self.value = start
    def __get__(self, instance, owner):    # On attr fetch
        return self.value ** 2
    def __set__(self, instance, value):    # On attr assign
        self.value = value                 # No delete or docs
  
class Client1:
    X = DescSquare(3)          # Assign descriptor instance to class attr

class Client2:
    X = DescSquare(32)         # Another instance in another client class
                               # Could also code two instances in same class

c1 = Client1()
c2 = Client2()

print(c1.X)      # 3 ** 2
c1.X = 4
print(c1.X)      # 4 ** 2
print(c2.X)      # 32 ** 2 (1024)
```



#### 在描述符中使用状态信息

实际上，描述符可以使用实例状态和描述符状态，或者二者的任何组合：

- 描述符状态用来管理内部用于描述符工作的数据。
- 实例状态记录了和客户类相关的信息，以及可能由客户类创建的信息。

例如，如下的描述符把信息附加到自己的实例，因此，它不会与客户类的实例上的信息冲突：

```python
# desc-state-desc.py

class DescState:                           # Use descriptor state, (object) in 2.X
    def __init__(self, value):
        self.value = value
    def __get__(self, instance, owner):    # On attr fetch
        print('DescState get')
        return self.value * 10
    def __set__(self, instance, value):    # On attr assign
        print('DescState set')
        self.value = value

# Client class
class CalcAttrs:
    X = DescState(2)                       # Descriptor class attr
    Y = 3                                  # Class attr
    def __init__(self):
        self.Z = 4                         # Instance attr

obj = CalcAttrs()
print(obj.X, obj.Y, obj.Z)                 # X is computed, others are not
obj.X = 5                                  # X assignment is intercepted
CalcAttrs.Y = 6                            # Y reassigned in class
obj.Z = 7                                  # Z assigned in instance
print(obj.X, obj.Y, obj.Z)

obj2 = CalcAttrs()                         # But X uses shared data, like Y!
print(obj2.X, obj2.Y, obj2.Z)
```

当这段代码运行的时候，`X`在获取时被计算，但它的值对所有客户实例都是相同的，因为它使用的是描述符级别（descriptor-level）的状态：
```
c:\code> py −3 desc-state-desc.py
DescState get
20 3 4
DescState set
DescState get
50 6 7
DescState get
50 6 4
```

对描述符存储或使用附加到客户类的实例的一个属性，而不是自己的属性，这也是可行的。**至关重要的一点是，与存储在描述符自身中的数据不同，这允许每个客户类实例都有不同的数据。**

如下例子中的描述符假设实例有一个数据`_X`通过客户类附加，并且使用它来计算她所表示的属性的值：

```python
# desc-state-inst.py

class InstState: # Using instance state, (object) in 2.X
    def __get__(self, instance, owner):
        print('InstState get')             # Assume set by client class
        return instance._X * 10
    def __set__(self, instance, value):
        print('InstState set')
        instance._X = value

# Client class
class CalcAttrs:
    X = InstState()                        # Descriptor class attr
    Y = 3                                  # Class attr
    def __init__(self):
    self._X = 2                            # Instance attr
    self.Z = 4                             # Instance attr

obj = CalcAttrs()
print(obj.X, obj.Y, obj.Z)                 # X is computed, others are not
obj.X = 5                                  # X assignment is intercepted
CalcAttrs.Y = 6                            # Y reassigned in class
obj.Z = 7                                  # Z assigned in instance
print(obj.X, obj.Y, obj.Z)

obj2 = CalcAttrs()                         # But X differs now, like Z!
print(obj2.X, obj2.Y, obj2.Z)
```

和之前一样，`X`被分配给管理访问的描述符。此描述符本身没有任何信息，但它使用一个假定存在于实例中的属性`_X`，以此避免与描述符本身的变量名`X`冲突。当这个版本的代码运行时，结果与之前相似；但由于不同的状态策略，描述符属性的值可以因不同客户实例而不同：

```
c:\code> py −3 desc-state-inst.py
InstState get
20 3 4
InstState set
InstState get
50 6 7
InstState get
20 6 4
```

**描述符和实例状态都有各自的用途。实际上，这是 *描述符* 优于 *特性* 的一个通用优点——因为它们都有自己的状态，所以可以很容易地在内部保存数据，而不用将数据添加到客户实例对象的命名空间中。**

作为总结，下面同时使用这两种状态信息存储方式：`self.data`保存每个属性的信息，而`instance.data`保存每个客户实例各自的信息：

```python
>>> class DescBoth:
        def __init__(self, data):
            self.data = data
        def __get__(self, instance, owner):
            return '%s, %s' % (self.data, instance.data)
        def __set__(self, instance, value):
            instance.data = value
>>> class Client:
        def __init__(self, data):
            self.data = data
        managed = DescBoth('spam')
>>> I = Client('eggs')
>>> I.managed # Show both data sources
'spam, eggs'
>>> I.managed = 'SPAM' # Change instance data
>>> I.managed
'spam, SPAM'
```

我们可以使用`dir`和`getattr`访问像特性和描述符的“虚拟”属性，即使它们不存在于实例的命名空间字典中。

```python
>>> I.__dict__
{'data': 'SPAM'}
>>> [x for x in dir(I) if not x.startswith('__')]
['data', 'managed']

>>> getattr(I, 'data')
'SPAM'
>>> getattr(I, 'managed')
'spam, SPAM'

>>> for attr in (x for x in dir(I) if not x.startswith('__')):
        print('%s => %s' % (attr, getattr(I, attr)))

data => SPAM
managed => spam, SPAM
The more generic __getattr__
```




#### 特性和描述符是如何相关的

特性和描述符有很强的相关性——`property`内置函数只是创建描述符的一种方便方式。我们可以使用如下的一个描述符类来模拟`property`内置函数：

```python
# prop-desc-equiv.py

class Property:
    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel                                  # Save unbound methods
        self.__doc__ = doc                                # or other callables

    def __get__(self, instance, instancetype=None):
        if instance is None:
            return self
        if self.fget is None:
            raise AttributeError("can't get attribute")
        return self.fget(instance)                        # Pass instance to self
                                                          # in property accessors
    def __set__(self, instance, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(instance, value)

    def __delete__(self, instance):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(instance)

class Person:
    def getName(self): print('getName...')
    def setName(self, value): print('setName...')
    name = Property(getName, setName)                     # Use like property()

x = Person()
x.name
x.name = 'Bob'
del x.name
```

这个`Property`类捕获了带有描述符协议的属性访问，并且把请求定位到创建类的时候在描述符状态中传入和保存的函数或方法。

```
c:\code> py −3 prop-desc-equiv.py
getName...
setName...
AttributeError: can't delete attribute
```



##### Descriptors and slots and more

注意，Python的`__slots__`是由描述符实现的。通过创建类级别的（class-level）描述符来截取对`slot`名称的访问，并将这些名称映射到实例中连续的存储空间，从而避免了实例的属性字典。然而，与显式的`property`调用不同，当一个`__slots__`属性出现在类中，slots背后的许多神奇之处都是在类创建时自动和隐式地编排的。

>  **注意： 在第39章中，我们还将使用描述符来实现应用于函数和方法的函数装饰器。正如你将在那里见到的，由于描述符接收描述符和主体类实例，它们在这种情况下工作得很好，尽管嵌套函数通常是一种更简单的解决方案。我们还将部署描述符作为拦截内置操作方法的一种方式。**



### 38.4 `__getattr__`和`__getattribute__`

`__getattr__`和`__getattribute__`操作符重载方法提供了另一种方法来拦截类实例的属性获取。就像特性和描述符一样，它们也允许我们插入一些特殊的代码。当访问属性的时候，这些代码会自动运行。

**属性获取拦截表现为两种形式，可用两个不同的方法来编写：**

- `__getattr__`针对未定义的属性运行——也就是，属性没有存储在实例上，或者没有从其类之一继承的时候。
- `__getattribute__`针对每个属性运行，因此，当使用它的时候，必须小心避免通过把属性访问传递给超类而导致递归循环。

`__getattr__`和`__getattribute__`方法也比特性和描述符更加通用——它们可以用来拦截对任何（几乎所有的）实例属性的获取，而不仅仅只是分配给它们的那些特定名称。因此，这两个方法很适合于通用的基于委托的编码模式——它们可以用来实现包装（也称为，代理，*proxy*）对象，该对象管理对一个嵌套对象的所有属性访问。**相反，如果使用特性和描述符，我们则必须为想要拦截的每个属性都定义一个特性或描述符。**

`__getattr__`和`__getattribute__`方法只拦截属性获取，而不拦截属性赋值。要捕获赋值对属性的更改，我们必须编写一个`__setattr__`方法——这是一个操作符重载方法，只对每个属性获取运行，必须小心避免由于通过实例命名空间字典指向属性赋值而导致的递归循环。

`__delattr__`重载方法来拦截属性删除。该方法也要求必须以同样的方式避免循环。**相反，特性和描述符通过设计(design)来捕获访问（get）、设置（set）和删除（delete）操作。**



#### 基础知识

简而言之，如果一个类定义了或继承了如下方法，那么当一个实例用于后面的注释所提到的情况时，它们将自动运行：

```python
def __getattr__(self, name): # On undefined attribute fetch [obj.name]
def __getattribute__(self, name): # On all attribute fetch [obj.name]
def __setattr__(self, name, value): # On all attribute assignment [obj.name=value]
def __delattr__(self, name): # On all attribute deletion [del obj.name]
```

所有这些之中，`self`通常是主体实例对象，`name`是将要访问的属性的字符串名，`value`是要赋给该属性的对象。两个get方法通常返回一个属性的值，另两个方法不返回（即，返回`None`）。

例如，要捕获每个属性获取，我们可以使用`__getattr__`和`__getattrbute__`方法；要捕获属性赋值，可以使用`__setattr__`：

```python
class Catcher:
    def __getattr__(self, name):
        print('Get: %s' % name)
    def __setattr__(self, name, value):
        print('Set: %s %s' % (name, value))

X = Catcher()
X.job                # Prints "Get: job"
X.pay                # Prints "Get: pay"
X.pay = 99           # Prints "Set: pay 99"
```

在这个例子中，使用`__getattribute__`也同样奏效，但要求必须是新式类，并且存在发生循环的可能性：

```python
class Catcher(object):                # Need (object) in 2.X only
    def __getattribute__(self, name): # Works same as getattr here
        print('Get: %s' % name)       # But prone to loops on general
    ...rest unchanged...
```

这样的代码结构可以用来实现我们在第31章介绍的委托设计模式。由于所有的属性通常都指向我们的拦截方法，所以我们可以验证它们并将其传递到嵌入的、管理的对象中。特性和描述符没有这样的类似功能，做不到对每个可能的包装对象中每个可能的属性编写访问器。

##### 避免属性拦截方法中的循环

这些方法通常都容易使用，它们唯一复杂的方面就是潜在的循环（也称为，递归）。由于`__getattribute__`和`__setattr__`针对所有的属性运行，因此，它们的代码要注意在访问其他属性的时候避免再次调用自己并触发一次递归循环。

例如，在一个`__getattribute__`方法代码内部的另一次属性获取，将会再次触发`__getattribute__`，并且代码将会循环直到内存耗尽：

```python
def __getattribute__(self, name):
    x = self.other # LOOPS!
```

技术上来说，这个方法比这个例子中所展示的更容易出现循环。一个`self`属性引用在定义了这个方法的类中的任何地方运行，都将触发`__getattribute__`，并且，根据类的逻辑，同样也存在潜在的循环。因为拦截每个属性的获取是这个方法的目的，所以这通常是期望的行为。但是你应该知道这个方法会捕获所有的属性获取，而不论它们的代码被编写在哪里。当代码被编写在`__getattribute__`中，这几乎总是会导致循环。

要避免这个问题，把获取指向一个更高的超类，而不是跳过这个层级的版本——`object`类总是一个新式类的超类，并且它在这里可以很好地起作用：

```python
def __getattribute__(self, name):
    x = object.__getattribute__(self, 'other') # Force higher to avoid me
```

对于`__setattr__`，情况是类似的。在这个方法内赋值任何属性，都会再次触发`__setattr__`并创建一个类似的循环：

```python
def __setattr__(self, name, value):
    self.other = value             # Recurs (and might LOOP!)
```

这里也是一样的，尽管`self`属性赋值出现在`__setattr__`中更容易导致循环，但如果在定义了`__setattr__`方法的类中的任何地方有`self`属性赋值，那么这些`self`属性赋值也会触发`__setattr__`。

要避免这个问题，把属性作为实例的`__dict__`命名空间字典中的一个键来赋值。这样就避免了直接的属性赋值：

```python
def __setattr__(self, name, value):
    self.__dict__['other'] = value     # Use attr dict to avoid me
```

尽管这不是传统的做法，但`__setattr__`方法也可以将自己的属性赋值传递给一个更高层级的超类，来避免循环，就像`__getattribute__`一样。**注意，这个方案有时候是更优的选择。**

```python
def __setattr__(self, name, value):
    object.__setattr__(self, 'other', value) # Force higher to avoid me
```

然而，相比之下，我们不能`__getattribute__`中使用`__dict__`技巧来避免循环：

```python
def __getattribute__(self, name):
    x = self.__dict__['other'] # Loops!
```

**获取`__dict__`属性本身会再次触发`__getattribute__`，导致一个递归循环。**

`__delattr__`方法针对每次属性删除而调用（就像针对每次属性赋值调用`__setattr__`一样）。因此，我们必须小心，在删除属性的时候要避免循环，通过使用同样的技术：命名空间字典或者超类方法调用。

> As noted in Chapter 30, attributes implemented with new-style class features such as slots and properties are not physically stored in the instance’s `__dict__` namespace dictionary (and slots may even preclude its existence entirely). Because of this, code that wishes to support suchattributes should code `__setattr__` to assign with the `object.__setattr__` scheme shown here, not by `self.__dict__` indexing. Namespace `__dict__` operations suffice for classes known to storedata in instances, like this chapter’s self-contained examples; general tools, though, should prefer object.



#### 第一个实例

这里是与用来说明特性和描述符的示例一样的示例，不过这次是用属性操作符重载方法实现的。由于这些方法如此通用，所以我们在这里测试属性名来获知何时将要访问一个管理的属性；其他的则允许正常传递：

```python
class Person:                               # Portable: 2.X or 3.X
    def __init__(self, name):               # On [Person()]
        self._name = name                   # Triggers __setattr__!

    def __getattr__(self, attr):            # On [obj.undefined]
        print('get: ' + attr)
        if attr == 'name':                  # Intercept name: not stored
            return self._name               # Does not loop: real attr
        else:                               # Others are errors
            raise AttributeError(attr)

    def __setattr__(self, attr, value):     # On [obj.any = value]
        print('set: ' + attr)
        if attr == 'name':
            attr = '_name'                  # Set internal name
        self.__dict__[attr] = value         # Avoid looping here

    def __delattr__(self, attr):            # On [del obj.any]
        print('del: ' + attr)
        if attr == 'name':
            attr = '_name'                  # Avoid looping here too
        del self.__dict__[attr]             # but much less common


bob = Person('Bob Smith')                   # bob has a managed attribute
print(bob.name)                             # Runs __getattr__
bob.name = 'Robert Smith'                   # Runs __setattr__
print(bob.name)
del bob.name                                # Runs __delattr__

print('-'*20)
sue = Person('Sue Jones')                   # sue inherits property too
print(sue.name)
#print(Person.name.__doc__)                 # No equivalent here
```

注意，`__init__`构造函数中的属性赋值也触发了`__setattr__`，这个方法捕获了每次属性赋值，即便是类自身之中的那些。运行这段代码的时候，会产生同样的输出，但这一次，它是Python的常规操作符重载机制与我们的属性拦截方法的结果：

```
c:\code> py −3 getattr-person.py
set: _name
get: name
Bob Smith
set: name
get: name
Robert Smith
del: name
--------------------
set: _name
get: name
Sue Jones
```

还要注意，与特性和描述符不同，这里没有为属性直接声明指定的文档，管理的属性存在于我们拦截方法的代码之中，而不是在不同的对象中。

##### 使用`__getattribute__`

用下面的代码替换示例中的`__getattr__`，以使用`__getattribute__`实现相同的结果。由于它会捕获所有的属性获取，这个版本必须通过把新的获取传递到超类来避免循环，并且通常不能假设未知的名称是错误：

```python
# getattribute-person.py

class Person:                               # Portable: 2.X or 3.X
    def __init__(self, name):               # On [Person()]
        self._name = name                   # Triggers __setattr__!

    # Replace __getattr__ with this
    def __getattribute__(self, attr):               # On [obj.any]
        print('get: ' + attr)
        if attr == 'name':                          # Intercept all names
            attr = '_name'                          # Map to internal name
        return object.__getattribute__(self, attr)  # Avoid looping here

    def __setattr__(self, attr, value):     # On [obj.any = value]
        print('set: ' + attr)
        if attr == 'name':
            attr = '_name'                  # Set internal name
        self.__dict__[attr] = value         # Avoid looping here

    def __delattr__(self, attr):            # On [del obj.any]
        print('del: ' + attr)
        if attr == 'name':
            attr = '_name'                  # Avoid looping here too
        del self.__dict__[attr]             # but much less common

bob = Person('Bob Smith')                   # bob has a managed attribute
print(bob.name)                             # Runs __getattr__
bob.name = 'Robert Smith'                   # Runs __setattr__
print(bob.name)
del bob.name                                # Runs __delattr__

print('-'*20)
sue = Person('Sue Jones')                   # sue inherits property too
print(sue.name)
```

当运行这个版本时，输出是类似的。但是，由于`__init__`中的赋值触发了`__setattr__`，而`__setattr__`中获取`__dict__`属性又触发了`__getattribute__`，所以我们得到一个额外的`__getattribute__`调用：

```
c:\code> py −3 getattribute-person.py
set: _name
get: __dict__
get: name
Bob Smith
set: name
get: __dict__
get: name
Robert Smith
del: name
get: __dict__
--------------------
set: _name
get: __dict__
get: name
Sue Jones
```

#### 计算的属性

与介绍特性和描述符时的情况相同，下面的代码创建了一个虚拟的属性`X`，当获取的时候自动计算它：

```python
# getattr-computed.py

class AttrSquare:
    def __init__(self, start):
        self.value = start               # Triggers __setattr__!

    def __getattr__(self, attr):         # On undefined attr fetch
        if attr == 'X':
            return self.value ** 2       # value is not undefined
        else:
            raise AttributeError(attr)

    def __setattr__(self, attr, value):  # On all attr assignments
        if attr == 'X':
            attr = 'value'
        self.__dict__[attr] = value      # 使用__dict__来避免循环


A = AttrSquare(3)      # 2 instances of class with overloading
B = AttrSquare(32)     # Each has different state information

print(A.X)             # 3 ** 2
A.X = 4
print(A.X)             # 4 ** 2
print(B.X)             # 32 ** 2 (1024)
```

运行这段代码，会产生与前面我们使用特性和描述符的时候相同的输出，但是，这段脚本的机制是基于通用的属性拦截方法：

```
c:\code> py −3 getattr-computed.py
9
16
1024
```



##### 使用`__getattribute__`

如前所述，我们可以用`__getattribute__`而不是`__getattr__`实现同样的效果。

```python
class AttrSquare: # Add (object) for 2.X
    def __init__(self, start):
        self.value = start                        # Triggers __setattr__!
    def __getattribute__(self, attr):             # On all attr fetches
        if attr == 'X':
            return self.value ** 2                # Triggers __getattribute__ again!
        else:
            return object.__getattribute__(self, attr)
    def __setattr__(self, attr, value):           # On all attr assignments
        if attr == 'X':
            attr = 'value'
        object.__setattr__(self, attr, value)     # 调用超类方法来避免循环
```

当这个版本运行的时候，结果再次相同。

**注意，隐式的指向在类的方法内部进行：**

- 构造函数中的 `self.value=start` 触发`__setattr__`。
- `__getattribute__`中`self.value`再次触发`__getattribute__`。

实际上，每次我们获取属性X的时候，`__getattribute__`都运行了两次。这并没有在`__getattr__`版本中发生，因为`value`属性没有定义。如果你关心速度并且要避免这一点，修改`__getattribute__`以使用超类来获取`value`：

```python
def __getattribute__(self, attr):
    if attr == 'X':
        return object.__getattribute__(self, 'value') ** 2
```



#### `__getattr__`和`__getattribute__`比较

为了概括`__getattr__`和`__getattribute__`之间的编码区别，下面的例子使用了这两者来实现3个属性——`attr1`是一个类属性；`attr2`是一个实例属性；`attr3`是一个虚拟的管理属性，当获取时计算它：

```python
class GetAttr:
    attr1 = 1
    def __init__(self):
        self.attr2 = 2
    def __getattr__(self, attr): # On undefined attrs only
        print('get: ' + attr) # Not on attr1: inherited from class
        if attr == 'attr3': # Not on attr2: stored on instance
            return 3
        else:
            raise AttributeError(attr)
            
X = GetAttr()
print(X.attr1)
print(X.attr2)
print(X.attr3)
print('-'*20)


class GetAttribute(object): # (object) needed in 2.X only
    attr1 = 1
    def __init__(self):
        self.attr2 = 2
    def __getattribute__(self, attr): # On all attr fetches
        print('get: ' + attr) # Use superclass to avoid looping here
        if attr == 'attr3':
            return 3
        else:
            return object.__getattribute__(self, attr)
        
X = GetAttribute()
print(X.attr1)
print(X.attr2)
print(X.attr3)
```

运行时，`__getattr__`版本拦截对`attr3`的访问，因为它是未定义的。另一方面，
`__getattribute__`版本拦截所有的属性获取，并且必须将那些没有管理的属性访问指向超类获取器以避免循环：

```
c:\code> py −3 getattr-v-getattr.py
1
2
get: attr3
3
--------------------
get: attr1
1
get: attr2
2
get: attr3
3
```

尽管`__getattribute__`拦截所有的属性获取，`__getattr__`拦截未定义属性的访问，但是实际上它们只是一个主题的不同变体。如果属性没有物理地存储，`__getattribute__`和`__getattr__`具有相同的效果。



#### 管理技术比较

为了概括我们在本章介绍的4种属性管理方法之间的编码区别，让我们快速地来看看使
用每种技术的一个更全面的计算属性的示例。

如下的版本使用特性来拦截并计算名为`square`和`cube`的属性。注意它们的基本值是如何存储到以下划线开头的名称中的，因此，它们不会与特性本身的名称冲突：

```python
# Two dynamically computed attributes with properties
class Powers(object):                  # Need (object) in 2.X only
    def __init__(self, square, cube):
        self._square = square          # _square is the base value
        self._cube = cube              # square is the property name

    def getSquare(self):
        return self._square ** 2
    def setSquare(self, value):
        self._square = value
    square = property(getSquare, setSquare)

    def getCube(self):
        return self._cube ** 3
    cube = property(getCube)


X = Powers(3, 4)
print(X.square)     # 3 ** 2 = 9
print(X.cube)       # 4 ** 3 = 64
X.square = 5
print(X.square)     # 5 ** 2 = 25
```

要用描述符做到同样的事情，我们需要用完整的类定义属性。注意，描述符把基础值存储为实例状态，因此，它们必须再次使用下划线开头，以便不会与描述符的名称冲突：

```python
# Same, but with descriptors (per-instance state)
class DescSquare(object):
    def __get__(self, instance, owner):
        return instance._square ** 2
    def __set__(self, instance, value):
        instance._square = value

class DescCube(object):
    def __get__(self, instance, owner):
        return instance._cube ** 3

class Powers(object):              # Need all (object) in 2.X only
    square = DescSquare()
    cube = DescCube()
    def __init__(self, square, cube):
        self._square = square           # "self.square = square" works too,
        self._cube = cube               # because it triggers desc __set__!

X = Powers(3, 4)
print(X.square)      # 3 ** 2 = 9
print(X.cube)        # 4 ** 3 = 64
X.square = 5
print(X.square)      # 5 ** 2 = 25
```

要使用`__getattr__`访问拦截来实现同样的结果，我们再次用下划线开头的名称存储基础值，这样对被管理的名称访问是未定义的，并且由此调用我们的方法。我们还需要编写一个`__setattrr__`来拦截赋值，并且注意避免其潜在的循环：

```python
# Same, but with generic __getattr__ undefined attribute interception
class Powers:
    def __init__(self, square, cube):
        self._square = square
        self._cube = cube
        
    def __getattr__(self, name):
        if name == 'square':
            return self._square ** 2
        elif name == 'cube':
            return self._cube ** 3
        else:
            raise TypeError('unknown attr:' + name)
            
    def __setattr__(self, name, value):
        if name == 'square':
            self.__dict__['_square'] = value     # Or use object
        else:
            self.__dict__[name] = value

X = Powers(3, 4)
print(X.square)     # 3 ** 2 = 9
print(X.cube)       # 4 ** 3 = 64
X.square = 5
print(X.square)     # 5 ** 2 = 25
```

最后一个选项，使用`__getattribute__`来编写，类似于前一个版本。由于我们现在捕获了每一个属性，因此必须把基础值获取指向超类以避免循环：

```python
# Same, but with generic __getattribute__ all attribute interception
class Powers(object): # Need (object) in 2.X only
    def __init__(self, square, cube):
        self._square = square
        self._cube = cube

    def __getattribute__(self, name):
        if name == 'square':
            return object.__getattribute__(self, '_square') ** 2
        elif name == 'cube':
            return object.__getattribute__(self, '_cube') ** 3
        else:
            return object.__getattribute__(self, name)

    def __setattr__(self, name, value):
        if name == 'square':
            object.__setattr__(self, '_square', value) # Or use __dict__
        else:
            object.__setattr__(self, name , value)

X = Powers(3, 4)
print(X.square)     # 3 ** 2 = 9
print(X.cube)       # 4 ** 3 = 64
X.square = 5
print(X.square)     # 5 ** 2 = 25
```

正如你所见到的，每种技术的编码形式都有所不同，但是，所有4种方法在运行的时候
都产生同样的结果：

```python
9
64
25
```



#### 拦截内置操作属性

注意，`__getattr__`和`__getattribute__`方法分别拦截未定义的属性获取和所有的属性获取，这使得它们很适合用于基于委托的编码模式。对于常规命名的属性来说，确实是这样的。但对于隐式地使用内置操作获取的方法名属性，`__getattr__`和`__getattribute__`方法可能根本不会运行。这意味着操作符重载方法调用不能委托给被包装的对象，除非包装类自己重新定义这些方法。

例如，针对`__str__`、`__add__`和`__getitem__`方法的属性获取分别通过打印、`+`表达式和索引隐式地运行，而不会指向Python 3.X中的类属性拦截方法。换句话说，在Python 3.X的类中（以及Python 2.X的新式类中），没有直接的方法来通用地拦截像打印和加法这样的内置操作。

如下的例子包含了`__getattr__`和`__getattribute__`方法的类，我们在其实例上测试各种属性类型和内置操作：

```python
# file getattr-bultins.py

class GetAttr:
    eggs = 88                       # eggs stored on class, spam on instance
    def __init__(self):
        self.spam = 77
    def __len__(self):              # len here, else __getattr__ called with __len__
        print('__len__: 42')
        return 42
    def __getattr__(self, attr):    # Provide __str__ if asked, else dummy func
        print('getattr: ' + attr)
        if attr == '__str__':
            return lambda *args: '[Getattr str]'
        else:
            return lambda *args: None


class GetAttribute(object):   # object required in 2.X, implied in 3.X
    eggs = 88                 # In 2.X all are isinstance(object) auto
    def __init__(self):       # But must derive to get new-style tools,
        self.spam = 77        # incl __getattribute__, some __X__ defaults
    def __len__(self):
        print('__len__: 42')
        return 42
    def __getattribute__(self, attr):
        print('getattribute: ' + attr)
        if attr == '__str__':
            return lambda *args: '[GetAttribute str]'
        else:
            return lambda *args: None


for Class in GetAttr, GetAttribute:
    print('\n' + Class.__name__.ljust(50, '='))
    
    X = Class()
    X.eggs                     # Class attr
    X.spam                     # Instance attr
    X.other                    # Missing attr
    len(X)                     # __len__ defined explicitly
    
    # New-styles must support [], +, call directly: redefine
    try: X[0]                  # __getitem__?
    except: print('fail []')
        
    try: X + 99                # __add__?
    except: print('fail +')
        
    try: X()                   # __call__? (implicit via built-in)
    except: print('fail ()')
        
    X.__call__()               # __call__? (explicit, not inherited)
    print(X.__str__())         # __str__? (explicit, inherited from type)
    print(X)                   # __str__? (implicit via built-in)
```



```
c:\code> py −3 getattr-builtins.py
GetAttr===========================================
getattr: other
__len__: 42
fail []
fail +
fail ()
getattr: __call__
<__main__.GetAttr object at 0x02987CC0>
<__main__.GetAttr object at 0x02987CC0>
GetAttribute======================================
getattribute: eggs
getattribute: spam
getattribute: other
__len__: 42
fail []
fail +
fail ()
getattribute: __call__
getattribute: __str__
[GetAttribute str]
<__main__.GetAttribute object at 0x02987CF8>
```

**我们可以跟踪这些输出，从而了解到脚本中的打印，看看这是如何工作的：**

- 在Python 3.X中，`__str__`访问有两次未能被`__getattr__`捕获：一次是针对内置打印，一次是针对显式获取，因为从该类继承了一个默认方法（实际上，该类来自内置`object`，它是每个类的一个超类）。
- `__str__`只有一次未能被`__getattribute__`捕获，在内置打印操作中，显式获取绕过了继承的版本。
- `__call__`在Python 3.X中用于内置调用表达式的两次都没有捕获，但是，当显式获取的时候，它两次都拦截到了；和`__str__`不同，没有继承的`__call__`默认版本能够超越`__getattr__`。
- `__len__`被两个类都捕获了，直接原因是，它在类自身中是一个显式定义的方法——它的名称指明了，在Python 3.X中，如果我们删除了类的`__len__`方法，它不会指向`__getattr__`或`__getattribute__`。
- 所有其他的内置操作在Python 3.X中都没有被两种方案拦截。



##### 回顾基于委托的（delegation-based）管理器

第28章的面向对象教程展示了一个`Manager`类，它使用对象嵌套和方法委托来定制它的超类，而不是使用继承。这里再次引用那段代码，删除了一些不相关的测试：

```python
class Person:
    def __init__(self, name, job=None, pay=0):
        self.name = name
        self.job = job
        self.pay = pay
    def lastName(self):
        return self.name.split()[-1]
    def giveRaise(self, percent):
        self.pay = int(self.pay * (1 + percent))
    def __repr__(self):
        return '[Person: %s, %s]' % (self.name, self.pay)

class Manager:
    def __init__(self, name, pay):
        self.person = Person(name, 'mgr', pay) # Embed a Person object
    def giveRaise(self, percent, bonus=.10):
        self.person.giveRaise(percent + bonus) # Intercept and delegate
    def __getattr__(self, attr):
        return getattr(self.person, attr)      # Delegate all other attrs
    def __repr__(self):
        return str(self.person)                # Must overload again (in 3.X)
    
if __name__ == '__main__':
    sue = Person('Sue Jones', job='dev', pay=100000)
    print(sue.lastName())
    sue.giveRaise(.10)
    print(sue)
    tom = Manager('Tom Jones', 50000)   # Manager.__init__
    print(tom.lastName())               # Manager.__getattr__ -> Person.lastName
    tom.giveRaise(.10)                  # Manager.giveRaise -> Person.giveRaise
    print(tom)                          # Manager.__repr__ -> Person.__repr__
```

像`Manager`这样的基于委托的类，在Python 3.X中必须重定义某些操作符重载方法（例如`__str__`）才能将它们指向嵌套的对象。也就是说，当操作符重载方法是一个对象的接口的一部分时，包装类必须通过在本地重新定义它们来容纳它们。



### 38.5 实例：属性验证

来看一个更实际的示例，以所有的4种属性管理方案来编写代码。我们将要使用的这个示例定义了一个`CardHolder`对象，它带有4个属性，其中3个属性是要管理的。管理的属性在获取或存储的时候要验证或转换值。对于同样的测试代码，所有4个版本都产生同样的结果，但是，它们以不同的方式实现了它们的属性。

#### 使用特性来验证

```python
# File validate_properties.py

class CardHolder(object): # Need "(object)" for setter in 2.X
    acctlen = 8 # Class data
    retireage = 59.5
    def __init__(self, acct, name, age, addr): # 注意，__init__构造函数方法内部的属性赋值也触发了特性的setter方法。
        self.acct = acct   # Instance data
        self.name = name   # These trigger prop setters too!
        self.age = age     # __X mangled to have class name
        self.addr = addr   # addr is not managed
        
    # remain has no data
    def getName(self):
        return self.__name
    def setName(self, value):
        value = value.lower().replace(' ', '_')
        self.__name = value
    name = property(getName, setName)
    
    def getAge(self):
        return self.__age
    def setAge(self, value):
        if value < 0 or value > 150:
            raise ValueError('invalid age')
        else:
            self.__age = value
    age = property(getAge, setAge)

    def getAcct(self):
        return self.__acct[:-3] + '***'
    def setAcct(self, value):
        value = value.replace('-', '')
        if len(value) != self.acctlen:
            raise TypeError('invald acct number')
        else:
            self.__acct = value
    acct = property(getAcct, setAcct)

    def remainGet(self): # Could be a method, not attr
        return self.retireage - self.age # Unless already using as attr
    remain = property(remainGet)
```



##### 测试代码

我们将对这个例子的所有4个版本使用这段同样的测试代码。当它运行的时候，我们创建了管理的属性类的两个实例，并且获取和修改其各种属性。期待失效的操作包装在`try`语句中：

```python
# File validate_tester.py

from __future__ import print_function # 2.X

def loadclass():
    import sys, importlib
    modulename = sys.argv[1] # Module name in command line
    module = importlib.import_module(modulename) # Import module by name string
    print('[Using: %s]' % module.CardHolder) # No need for getattr() here
    return module.CardHolder

def printholder(who):
    print(who.acct, who.name, who.age, who.remain, who.addr, sep=' / ')
    
    
if __name__ == '__main__':
    CardHolder = loadclass()
    bob = CardHolder('1234-5678', 'Bob Smith', 40, '123 main st')
    printholder(bob)
    bob.name = 'Bob Q. Smith'
    bob.age = 50
    bob.acct = '23-45-67-89'
    printholder(bob)

    sue = CardHolder('5678-12-34', 'Sue Jones', 35, '124 main st')
    printholder(sue)
    try:
        sue.age = 200
    except:
        print('Bad age for Sue')

    try:
        sue.remain = 5
    except:
        print("Can't set sue.remain")

    try:
        sue.acct = '1234567'
    except:
        print('Bad acct for Sue')
```

如下是我们的self测试代码的输出。并且，这对这个示例的所有版本都是一样的。

```
c:\code> py −3 validate_tester.py validate_properties
[Using: <class 'validate_properties.CardHolder'>]
12345*** / bob_smith / 40 / 19.5 / 123 main st
23456*** / bob_q._smith / 50 / 9.5 / 123 main st
56781*** / sue_jones / 35 / 24.5 / 124 main st
Bad age for Sue
Can't set sue.remain
Bad acct for Sue
```



#### 使用描述符来验证

##### Option 1: 使用共享的描述符实例状态进行验证（Validating with shared descriptor instance state）

```python
# File validate_descriptors1.py: using shared descriptor state

class CardHolder(object): # Need all "(object)" in 2.X only
    acctlen = 8 # Class data
    retireage = 59.5
    
    def __init__(self, acct, name, age, addr):   # 注意：__init__构造函数方法内部的属性赋值会触发描述符的__set__操作符方法
        self.acct = acct    # Instance data
        self.name = name    # These trigger __set__ calls too!
        self.age = age      # __X not needed: in descriptor
        self.addr = addr    # addr is not managed

    # remain has no data
    class Name(object):
        def __get__(self, instance, owner):    # Class names: CardHolder locals
            return self.name
        def __set__(self, instance, value):
            value = value.lower().replace(' ', '_')
            self.name = value     # 赋值给描述符的属性。实际的name值附加到了描述符对象，而不是客户类实例。
    name = Name()   # 在CardHolder客户类中，名为name的属性总是一个描述符对象，而不是数据。

    class Age(object):
        def __get__(self, instance, owner):
            return self.age # Use descriptor data
        def __set__(self, instance, value):
            if value < 0 or value > 150:
                raise ValueError('invalid age')
            else:
                self.age = value
    age = Age()
    
    class Acct(object):
        def __get__(self, instance, owner):
            return self.acct[:-3] + '***'
        def __set__(self, instance, value):
            value = value.replace('-', '')
            if len(value) != instance.acctlen:          # Use instance class data
                raise TypeError('invald acct number')
            else:
                self.acct = value
    acct = Acct()
    
    class Remain(object):
        def __get__(self, instance, owner):
            return instance.retireage - instance.age    # Triggers Age.__get__
        def __set__(self, instance, value):  # 实现__set__并引发异常，以实现只读描述符
            raise TypeError('cannot set remain')        # Else set allowed here
    remain = Remain()   # remain是只读属性，且完全虚拟的，并根据需要计算。
```

```
C:\code> python validate_tester.py validate_descriptors1
...same output as properties, except class name...
```



##### Option 2: 使用每客户类实例状态来进行验证（Validating with per-client-instance state）

Unlike in the prior property-based variant, though, in this case the actual name value is attached to the descriptor object, not the client class instance. Although we could store this value in either instance or descriptor state, the latter avoids the need to mangle names with underscores to avoid collisions. In the `CardHolder` client class, the attribute called `name` is always a descriptor object, not data.

Importantly, the downside of this scheme is that state stored inside a descriptor itself is class-level data that is effectively shared by all client class instances, and so cannot vary between them. That is, storing state in the descriptor instance instead of the owner (client) class instance means that the state will be the same in all owner class instances. Descriptor state can vary only per attribute appearance.

To see this at work, in the preceding descriptor-based `CardHolder` example, try printing attributes of the bob instance after creating the second instance, `sue`. The values of sue’s managed attributes (`name`, `age`, and `acct`) overwrite those of the earlier object `bob`, because both share the same, single descriptor instance attached to their class:

```python
# File validate_tester2.py

from __future__ import print_function # 2.X

from validate_tester import loadclass
CardHolder = loadclass()

bob = CardHolder('1234-5678', 'Bob Smith', 40, '123 main st')
print('bob:', bob.name, bob.acct, bob.age, bob.addr)

sue = CardHolder('5678-12-34', 'Sue Jones', 35, '124 main st')
print('sue:', sue.name, sue.acct, sue.age, sue.addr) # addr differs: client data
print('bob:', bob.name, bob.acct, bob.age, bob.addr) # name,acct,age overwritten?
```



```
c:\code> py −3 validate_tester2.py validate_descriptors1
[Using: <class 'validate_descriptors1.CardHolder'>]
bob: bob_smith 12345*** 40 123 main st
sue: sue_jones 56781*** 35 124 main st
bob: sue_jones 56781*** 35 123 main st
```



```python
# File validate_descriptors2.py: using per-client-instance state

class CardHolder(object): # Need all "(object)" in 2.X only
    acctlen = 8 # Class data
    retireage = 59.5
    
    def __init__(self, acct, name, age, addr):
        self.acct = acct # Client instance data
        self.name = name # These trigger __set__ calls too!
        self.age = age # __X needed: in client instance
        self.addr = addr # addr is not managed
        
    # remain managed but has no data
    class Name(object):
        def __get__(self, instance, owner): # Class names: CardHolder locals
            return instance.__name
        def __set__(self, instance, value):
            value = value.lower().replace(' ', '_')
            instance.__name = value
    name = Name() # class.name vs mangled attr
    
    class Age(object):
        def __get__(self, instance, owner):
            return instance.__age # Use descriptor data
        def __set__(self, instance, value):
            if value < 0 or value > 150:
                raise ValueError('invalid age')
            else:
                instance.__age = value
    age = Age()  # class.age vs mangled attr
    
    class Acct(object):
        def __get__(self, instance, owner):
            return instance.__acct[:-3] + '***'
        def __set__(self, instance, value):
            value = value.replace('-', '')
            if len(value) != instance.acctlen: # Use instance class data
                raise TypeError('invald acct number')
            else:
                instance.__acct = value
    acct = Acct() # class.acct vs mangled name
    
    class Remain(object):
        def __get__(self, instance, owner):
            return instance.retireage - instance.age  # Triggers Age.__get__
        def __set__(self, instance, value):
            raise TypeError('cannot set remain')      # Else set allowed here
    remain = Remain()
```



```
c:\code> py −3 validate_tester2.py validate_descriptors2
[Using: <class 'validate_descriptors2.CardHolder'>]
bob: bob_smith 12345*** 40 123 main st
sue: sue_jones 56781*** 35 124 main st
bob: bob_smith 12345*** 40 123 main st

c:\code> py −3 validate_tester.py validate_descriptors2
...same output as properties, except class name...
```





#### 使用`__getattr__`验证

这个替代方法代码量最少。当然，清晰与否比代码大小更重要，但额外的代码有时候意味着额外的开发和维护工作。可能这里更重要的是角色：像`__getattr__`这样的通用工具可能更适合于通用委托，而特性和描述符更直接是为了管理特定属性而设计。

```python
# File validate_getattr.py
class CardHolder:
    acctlen = 8 # Class data
    retireage = 59.5
    
    def __init__(self, acct, name, age, addr):  # 注意:__init__构造函数方法中的属性赋值会触发类的__setattr__方法
        self.acct = acct   # Instance data
        self.name = name   # These trigger __setattr__ too
        self.age = age     # _acct not mangled: name tested
        self.addr = addr   # addr is not managed
    
    # remain has no data
    def __getattr__(self, name):
        if name == 'acct':                            # On undefined attr fetches
            return self._acct[:-3] + '***'            # name, age, addr are defined
        elif name == 'remain':
            return self.retireage - self.age          # Doesn't trigger __getattr__
        else:
            raise AttributeError(name)
    
    def __setattr__(self, name, value):
        if name == 'name':                            # On all attr assignments
            value = value.lower().replace(' ', '_')   # addr stored directly
        elif name == 'age':                           # acct mangled to _acct
            if value < 0 or value > 150:
                raise ValueError('invalid age')
        elif name == 'acct':
            name = '_acct'
            value = value.replace('-', '')
            if len(value) != self.acctlen:
                raise TypeError('invald acct number')
        elif name == 'remain':
            raise TypeError('cannot set remain')   # 将remain实现为只读属性
        self.__dict__[name] = value                   # Avoid looping (or via object)
```

When this code is run with either test script, it produces the same output (with a different class name):

```
c:\code> py −3 validate_tester.py validate_getattr
...same output as properties, except class name...

c:\code> py −3 validate_tester2.py validate_getattr
...same output as instance-state descriptors, except class name...
```





#### 使用`__getattribute__`验证

```python
# File validate_getattribute.py
class CardHolder(object): # Need "(object)" in 2.X only
    acctlen = 8 # Class data
    retireage = 59.5
    
    def __init__(self, acct, name, age, addr):
        self.acct = acct # Instance data
        self.name = name # These trigger __setattr__ too
        self.age = age   # acct not mangled: name tested
        self.addr = addr # addr is not managed
        
    # remain has no data
    def __getattribute__(self, name):
        superget = object.__getattribute__ # Don't loop: one level up
        if name == 'acct': # On all attr fetches
            return superget(self, 'acct')[:-3] + '***'
        elif name == 'remain':
            return superget(self, 'retireage') - superget(self, 'age')
        else:
            return superget(self, name) # name, age, addr: stored

    def __setattr__(self, name, value):
        if name == 'name': # On all attr assignments
            value = value.lower().replace(' ', '_') # addr stored directly
        elif name == 'age':
            if value < 0 or value > 150:
                raise ValueError('invalid age')
        elif name == 'acct':
            value = value.replace('-', '')
            if len(value) != self.acctlen:
                raise TypeError('invald acct number')
        elif name == 'remain':
            raise TypeError('cannot set remain')
        self.__dict__[name] = value  # Avoid loops, orig names
```







---



## 第39章 装饰器

### 39.1 什么是装饰器

**装饰** 是为函数和类指定管理代码的一种方式。**装饰器** 本身的形式是处理其他的可调用对象的可调用对象（如函数）。

Python装饰器以2种相关的形式呈现：

- **函数装饰器** 在函数定义的时候进行名称重绑定，提供一个逻辑层来管理函数和方法，或随后对它们的调用。
- **类装饰器** 在类定义的时候进行名称重绑定，提供一个逻辑层来管理类，或管理随后调用它们所创建的实例。

简而言之，装饰器提供了一种方法，在函数和类定义语句的末尾插入自动运行代码——对于函数装饰器，在`def`的末尾；对于类装饰器，在`class`的末尾。

#### 管理调用和实例

通常的用法中，这种自动运行的代码可能用来增强对函数和类的调用。它通过安装随后调用的包装器（*wrapper*，也称为，代理，*proxy*）对象来实现这一点：

- **调用代理（Call proxies）：** *函数装饰器* 安装包装器（wrapper）对象，来拦截随后的函数调用并根据需要处理它们；通常将调用传递给最初的（original）函数以运行被管理的动作。
- **接口代理（Interface proxies）：**  *类装饰器* 安装包装器（wrapper）对象，来拦截随后的实例创建调用并根据需要处理它们；通常将调用传递给最初的（original）类以创建一个被管理的实例。

在`def`和`class`语句的末尾，装饰器通过自动把函数和类名重绑定到其他的可调用对象来实现这些效果。当随后调用的时候，这些可调用对象可以执行诸如对函数调用跟踪和计时、管理对类实例属性的访问等任务。

#### 管理函数和类

使用包装器来拦截随后对函数和类的调用，这并非使用装饰器的唯一方法：

- **函数管理器（function managers）：** *函数装饰器* 也可以用来管理函数对象，而不是随后对它们的调用——例如，把一个函数注册到一个API。然而，我们在这里主要关注更为常见的用法，即调用包装器应用程序。
- **类管理器（Class managers）：** *类装饰器* 也可以用来直接管理类对象，而不是实例创建调用（instance creation calls）——例如，用新的方法扩展类。因为这些用法和元类有很大的重合，我们将会发现，这两种工具都是在类创建过程的最后运行，但类装饰器通常提供一种较轻量的解决方案。

**换句话说，*函数装饰器* 可以用来管理函数调用和函数对象，*类装饰器* 可以用来管理类实例和类自身。**通过返回装饰的对象自身而不是一个包装器，装饰器变成了针对函数和类的一种简单的后创建步骤（post-creation step）。

#### 使用和定义装饰器

根据你的工作形式，你可能成为装饰器的用户或提供者。如果只是使用装饰器，那么不需要知道装饰器如何编码就可以完成任务。对于更为通用的任务，程序员可以编写自己的任意装饰器。例如，函数装饰器可能通过添加跟踪调用、在调试时执行参数验证测试、自动获取和释放线程锁、统计调用函数的次数以进行优化等的代码来扩展函数。

***函数装饰器*** 设计用来只增强一个特定函数或方法调用，而不是一个完整的对象接口。***类装饰器*** 可以拦截实例创建调用，所以用来实现任意的对象接口扩展或管理任务。例如，定制的类装饰器可以跟踪或验证对一个对象的每个属性引用。

#### 为什么使用装饰器

装饰器提供了一种显式的语法，它使得意图明确，可以最小化扩展代码的冗余，并且有助于确保正确的API使用：

- 装饰器有一种非常明确的语法，这使得它们比那些可能任意地远离主体函数或类的辅助函数调用更容易为人们发现。
- 当主体函数或类定义的时候，装饰器应用一次；在对类或函数的每次调用的时候，不必添加额外的代码（在未来可能必须改变）。
- 由于前面两点，装饰器使得一个API的用户不太可能忘记根据API需求扩展一个函数或类。



### 39.2 基础知识

#### 函数装饰器

函数装饰器只是一种语法糖：通过在一个函数的`def`语句的末尾来运行另一个函数，把最初的函数名重新绑定到结果。

##### 用法

函数装饰器是一种关于函数的运行时声明，函数的定义需要遵守此声明。装饰器在紧挨着定义一个函数或方法的`def`语句之前的一行编写，并且它由`@`符号以及紧随其后的对于元函数的一个引用组成——这是管理另一个函数的一个函数（或其他的可调用对象）。

**装饰器是一个单参数的可调用对象，它返回与F具有相同数目的参数的一个可调用对象。** 

在编码方面，函数装饰器自动将如下的语法：

```python
@decorator      # Decorate function
def F(arg):
...
F(99)           # Call function
```

映射为这一对等的形式：

```python
def F(arg):
...
F = decorator(F)     # Rebind function name to decorator result
F(99)                # Essentially calls decorator(F)(99)
```

这一自动名称重绑定在`def`语句上有效，不管它针对一个简单的函数或是类中的一个方法。当随后调用`F`函数的时候，它自动调用装饰器所返回的对象，该对象可能是实现了所需的包装逻辑的另一个对象，或者是最初的函数本身。

换句话说，调用一个被装饰的函数，将会调用装饰器所返回的对象。即，装饰实际把如下的第一行映射为第二行（尽管装饰器实际上只运行一次，在装饰的时候）：

```python
func(6, 7)
decorator(func)(6, 7)
```

##### 实现

**装饰器** 自身是一个 **返回可调用对象的可调用对象** 。也就是说，它返回了一个可调用对象，当随后装饰的函数通过其最初的名称调用的时候，将会调用这个可调用对象。并且，装饰器可以是任意类型的可调用对象，并且返回任意类型的可调用对象。

例如，要在一个函数创建之后接入装饰协议以管理函数，我们需要编写如下形式的装饰器：

```python
def decorator(F):
    # Process function F
    return F

@decorator
def func(): ...   # func = decorator(func)
```

更典型的用法，是插入逻辑以拦截对函数的随后调用，我们可以编写一个装饰器来返回和最初函数不同的一个对象：

```python
def decorator(F):
    # Save or use function F
    # Return a different callable: nested def, class with __call__, etc.

@decorator
def func(): ... # func = decorator(func)
```

这个装饰器在装饰的时候调用，并且当随后调用最初的函数名的时候，它所返回的调用对象将被调用。 **装饰器自身接受被装饰的函数，返回的调用对象会接受随后传递给被装饰函数的名称的任何参数。** 

更概括地说，有一种常用的编码模式可以包含这一思想——装饰器返回了一个包装器(wrapper)，包装器把最初的函数保持到一个封闭的作用域中：

```python
def decorator(F):         # On @ decoration
    def wrapper(*args):   # On wrapped function call
        # Use F and args
        # F(*args) calls original function
    return wrapper

@decorator                # func = decorator(func)
def func(x, y):           # func is passed to decorator's F
    ...

func(6, 7)                # 6, 7 are passed to wrapper's *args
```

为了对类做同样的事情，我们可以重载调用操作`__call__`方法，并且使用实例属性而不是封闭的作用域：

```python
class decorator:
    def __init__(self, func):         # On @ decoration
        self.func = func
    def __call__(self, *args):        # On wrapped function call
        # Use self.func and args
        # self.func(*args) calls original function


@decorator
def func(x, y):                       # func = decorator(func)
    ...                               # func is passed to __init__


func(6, 7)                            # 6, 7 are passed to __call__'s *args
```

现在，随后再调用`func`的时候，它确实会调用装饰器所创建的实例的`__call__`运算符重载方法；然后，`__call__`方法可能运行最初的`func`，因为它在一个实例属性中仍然可用。当按照这种方式编写代码的时候，每个装饰的函数都会产生一个新的实例来保持状态。

##### 支持方法装饰

尽管前面的基于类的代码对于拦截简单函数调用有效，但当它应用于类方法函数的时候，并不是很有效：

```python
class decorator:
    def __init__(self, func):         # func is method without instance
        self.func = func
    def __call__(self, *args):        # self is decorator instance
        # self.func(*args) fails!     # C instance not in args!

class C:
    @decorator
    def method(self, x, y):           # method = decorator(method)  这相当于实例化并赋值给实例
        ...                           # Rebound to decorator instance
```

当按照这种方式编码的时候，装饰的方法重绑定到装饰器类的一个实例，而不是一个简单的函数。

为了支持函数和方法，嵌套函数的替代方法工作得更好：

```python
def decorator(F):           # F is func or method without instance
    def wrapper(*args):     # class instance in args[0] for method
        # F(*args) runs func or method
    return wrapper


@decorator
def func(x, y):             # func = decorator(func)
    ...
    
func(6, 7)                  # Really calls wrapper(6, 7)


class C:
    @decorator
    def method(self, x, y): # method = decorator(method)
        ...                 # Rebound to simple function


X = C()
X.method(6, 7)              # Really calls wrapper(X, 6, 7)
```

注意，嵌套函数可能是支持函数和方法的装饰的最直接方式，但是不一定是唯一的方式。

#### 类装饰器

类装饰器与函数装饰器密切相关，实际上，它们使用相同的语法和非常相似的编码模式。然而，不是包装单个的函数或方法，类装饰器是管理类的一种方式，或者用管理或扩展类所创建的实例的额外逻辑，来包装实例构建调用。

##### 用法

从语法上讲，类装饰器就像前面的`class`语句一样（就像前面函数定义中出现的函数装饰器）。装饰器必须是返回一个可调用对象的一个单参数的可调用对象，类装饰器语法如下：

```python
@decorator    # Decorate class
class C:
    ...

x = C(99)     # Make an instance
```

等同于下面的语法——类自动地被传递给装饰器函数，并且装饰器的结果返回来分配给类名：

```python
class C:
    ...

C = decorator(C)  # Rebind class name to decorator result
x = C(99)         # Essentially calls decorator(C)(99)
```

直接的效果就是，随后调用类名会创建一个实例，该实例会触发装饰器所返回的可调用对象，而不是调用最初的类自身。

#### 实现

新的类装饰器使用函数装饰器所使用的众多相同的技术来编写，尽管有些技术可能涉及两个级别的扩展，以此来管理实例构造调用和实例接口访问。由于类装饰器也是返回一个可调用对象的一个可调用对象，因此大多数函数和类的组合已经足够了。

尽管先编码，但装饰器的结果是当随后创建一个实例的时候才运行的。例如，要在一个类创建之后直接管理它，返回最初的类自身：

```python
def decorator(C):
    # Process class C
    return C

@decorator
class C: ...             # C = decorator(C)
```

插入一个包装器层来拦截随后的实例创建调用，而是返回一个不同的可调用对象：

```python
def decorator(C):
    # Save or use class C
    # Return a different callable: nested def, class with __call__, etc.
    
@decorator
class C: ...            # C = decorator(C)
```

这样的一个类装饰器返回的可调用对象，通常创建并返回最初的类的一个新的实例，以某种方式来扩展对其接口的管理。例如，下面的实例插入一个对象来拦截一个类实例的未定义的属性：

```python
def decorator(cls):                             # On @ decoration
    class Wrapper:
        def __init__(self, *args):              # On instance creation
            self.wrapped = cls(*args)
        def __getattr__(self, name):            # On attribute fetch
            return getattr(self.wrapped, name)
    return Wrapper

@decorator
class C:                                       # C = decorator(C)
    def __init__(self, x, y):                  # Run by Wrapper.__init__
        self.attr = 'spam'

x = C(6, 7)                                    # Really calls Wrapper(6, 7)
print(x.attr)                                  # Runs Wrapper.__getattr__, prints "spam"
```

就像函数装饰器一样，类装饰器通常可以编写为一个创建并返回可调用的对象的“工厂”函数，或者使用`__init__`或`__call__`方法来拦截所有调用操作的类，或者是由此产生的一些组合。工厂函数通常在封闭的作用域引用中保持状态，类通常在属性中保持状态。

##### 支持多个实例

如下模式中的任意一种都支持多个包装的实例：

```python
def decorator(C):                     # On @ decoration
    class Wrapper:
        def __init__(self, *args):    # On instance creation: new Wrapper
            self.wrapped = C(*args)   # Embed instance in instance
    return Wrapper
```

```python
class Wrapper: 
    pass       # 省略具体实现
    
def decorator(C):                  # On @ decoration
    def onCall(*args):             # On instance creation: new Wrapper
        return Wrapper(C(*args))   # Embed instance in instance
    return onCall
```



#### 装饰器嵌套

这种形式的装饰器语法：

```python
@A
@B
@C
def f(...):
    ...
```

像下面这样运行：

```python
def f(...):
    ...
f = A(B(C(f)))
```

这里，最初的函数通过3个不同的装饰器传递，并且最终的可调用对象返回来分配给最初的名称。每个装饰器处理前一个的结果，这可能是最初的函数或一个插入的包装器。

就像对函数一样，多个类装饰器导致了多个嵌套的函数调用，并且可能导致围绕实例创建调用的包装器逻辑的多个层。例如，如下的代码：

```python
@spam
@eggs
class C:
    ...

X = C()
```

等价于如下代码：

```python
class C:
    ...
C = spam(eggs(C))

X = C()
```

例如，如下的什么也不做的装饰器只是返回被装饰的函数：

```python
def d1(F): return F
def d2(F): return F
def d3(F): return F

@d1
@d2
@d3
def func():          # func = d1(d2(d3(func)))
    print('spam')

func()               # Prints "spam"
```

**同样的语法在类上也有效！**

然而，当装饰器插入包装器函数对象，调用的时候它们可能扩展最初的函数——如下的代码将其结果连接到一个装饰器层中，随着它从内向外地运行层：

```python
def d1(F): return lambda: 'X' + F()   # 使用lambda函数来实现包装器层（每个层在一个封闭的作用域里保持了包装的函数）。
def d2(F): return lambda: 'Y' + F()   
def d3(F): return lambda: 'Z' + F()   

@d1
@d2
@d3
def func(): # func = d1(d2(d3(func)))
    return 'spam'

print(func()) # Prints "XYZspam"
```

实际上，包装器可以采取函数、可调用的类以及更多形式。当设计良好的时候，装饰器嵌套允许我们以种类多样的方式来组合扩展步骤。



#### 装饰器参数

函数装饰器和类装饰器似乎都能接受参数，尽管实际上这些参数传递给了真正返回装饰器的一个可调用对象，而装饰器反过来又返回一个可调用对象。例如，如下代码：

```python
@decorator(A, B)
def F(arg):
    ...
    
F(99)
```

自动地映射到其对等的形式，其中装饰器是一个可调用对象，它返回实际的装饰器。返回的装饰器又返回一个可调用的对象，这个对象随后运行以调用最初的函数名：

```python
def F(arg):
    ...
F = decorator(A, B)(F) # Rebind F to result of decorator's return value
F(99)                  # Essentially calls decorator(A, B)(F)(99)
```

装饰器参数在装饰发生之前就解析了，并且它们通常用来保持状态信息供随后的调用使用。例如，这个例子中的装饰器函数，可能采用如下的形式：

```python
def decorator(A, B):
    # Save or use A, B
    def actualDecorator(F):
        # Save or use function F
        # Return a callable: nested def, class with __call__, etc.
        return callable
    return actualDecorator
```

这个结构中的外围函数通常会把装饰器参数与状态信息分开保存，以便在实际的装饰器中使用，或者在它所返回的可调用对象中使用，或者在二者中都使用。这段代码在封闭的函数作用域引用中保存了状态信息参数，但是通常也可以使用类属性。

换句话说，装饰器参数往往意味着可调用对象的3个层级：接受装饰器参数的一个可调用对象，它返回一个可调用对象以作为装饰器，该装饰器返回一个可调用对象来处理对最初的函数或类的调用。这3个层级的每一个都可能是一个函数或类，并且可能以作用域或类属性的形式保存了状态。

简单实例：

```python
In [15]: def decorator(a, b):
    ...:     def actualDecorator(F):
    ...:         def func():
    ...:             print("a=%s, b=%s" % (a, b))
    ...:             print(type(F), F.__name__)
    ...:         return func
    ...:     return actualDecorator
    ...:

In [16]: @decorator(1,2)
    ...: def test():
    ...:     pass
    ...:

In [17]: test()
a=1, b=2
<class 'function'> test
```



#### 装饰器管理函数和类

装饰器还可以作为在函数和类创建之后通过一个可调用对象传递它们的一种协议。因此，它可以用来调用任意的创建后处理：

```python
def decorate(O):
    # Save or augment function or class O
    return O

@decorator
def F(): ... # F = decorator(F)

@decorator
class C: ... # C = decorator(C)
```

只要以这种方式返回最初装饰的对象，而不是返回一个包装器，我们就可以管理函数和类自身，而不只是管理随后对它们的调用。



### 39.3 编写函数装饰器

#### 跟踪调用

如下的代码定义并应用一个函数装饰器，来统计对装饰的函数的调用次数，并且针对每一次调用打印跟踪信息：

```python
# File decorator1.py

class tracer:
    def __init__(self, func): # On @ decoration: save original func
        self.calls = 0
        self.func = func
    def __call__(self, *args): # On later calls: run original func
        self.calls += 1
        print('call %s to %s' % (self.calls, self.func.__name__))
        self.func(*args)
        
@tracer
def spam(a, b, c):       # spam = tracer(spam)   注意，用这个类装饰的每个函数将创建一个新的实例，带有自己保存的函数对象和调用计数器。
    print(a + b + c)     # Wraps spam in a decorator object
```

注意，用这个类装饰的每个函数将创建一个新的实例，带有自己保存的函数对象和调用计数器。注意观察，`*args`参数语法如何用来打包和解包任意的多个传入参数。这一通用性使得这个装饰器可以用来包装带有任意多个参数的任何函数（这个版本还不能在类方法上工作，但是，我们将在本小节稍后修改这一点）。

```python
>>> from decorator1 import spam
>>> spam(1, 2, 3)            # Really calls the tracer wrapper object
call 1 to spam
6
>>> spam('a', 'b', 'c')      # Invokes __call__ in class
call 2 to spam
abc
>>> spam.calls               # Number calls in wrapper state information
2
>>> spam
<decorator1.tracer object at 0x02D9A730>
```

运行的时候，`tracer`类和装饰的函数分开保存，并且拦截对装饰的函数随后的调用，以便添加一个逻辑层来统计和打印每次调用。注意，调用的总数如何作为装饰的函数的一个属性显示——装饰的时候，`spam`实际上是tracer类的一个实例（对于进行类型检查的程序，可能还会衍生一次查找，但是通常是有益的）。

对于函数调用，`@`装饰语法可能比修改每次调用来说明额外的逻辑层要更加方便，并且它避免了意外地直接调用最初的函数。考虑如下所示的非装饰器的对等代码：

```python
calls = 0
def tracer(func, *args):
    global calls
    calls += 1
    print('call %s to %s' % (calls, func.__name__))
    func(*args)
    
def spam(a, b, c):
    print(a, b, c)
    
>>> spam(1, 2, 3)            # Normal nontraced call: accidental?
1 2 3
>>> tracer(spam, 1, 2, 3)    # Special traced call without decorators
call 1 to spam
1 2 3
```

这一替代方法可以用在任何函数上，且不需要特殊的`@`语法，但是和装饰器版本不同：

- 它在代码中调用函数的每个地方需要额外的语法。

- 它的意图可能不够明显，并且它不能确保额外的层将会针对常规调用而调用。

我们总是可以手动地重新绑定名称，所以装饰器不是必需的，但它们通常是最为方便的。



#### 状态信息保持选项

函数装饰器有各种选项来保持装饰的时候所提供的状态信息，以便在实际函数调用过程中使用。它们通常需要支持多个装饰的对象以及多个调用，但是，有多种方法来实现这些目标：**实例属性** 、**全局变量** 、**非局部变量** 和 **函数属性** ，都可以用于保持状态。

##### 类实例属性

例如，这里是前面的例子的一个扩展版本，其中添加了对关键字参数的支持，并且返回包装函数的结果，以支持更多的用例：

```python
class tracer:                               # State via instance attributes
    def __init__(self, func):               # On @ decorator
        self.calls = 0                      # Save func for later call
        self.func = func
    def __call__(self, *args, **kwargs):    # On call to original function
        self.calls += 1
        print('call %s to %s' % (self.calls, self.func.__name__))
        return self.func(*args, **kwargs)

@tracer
def spam(a, b, c):     # Same as: spam = tracer(spam)
    print(a + b + c)   # Triggers tracer.__init__

@tracer
def eggs(x, y):        # Same as: eggs = tracer(eggs)
    print(x ** y)      # Wraps eggs in a tracer object
    
spam(1, 2, 3)          # Really calls tracer instance: runs tracer.__call__
spam(a=4, b=5, c=6)    # spam is an instance attribute

eggs(2, 16)            # Really calls tracer instance, self.func is eggs
eggs(4, y=4)           # self.calls is per-decoration here
```

就像最初的版本一样，这里的代码使用类实例属性来显式地保存状态。包装的函数和调用计数器都是针对每个实例的信息——每个装饰都有自己的拷贝。注意`spam`和`eggs`函数的每一个都有自己的调用计数器，因为每个装饰都创建一个新的类实例。这个版本的输出如下所示：

```
c:\code> python decorator2.py
call 1 to spam
6
call 2 to spam
15
call 1 to eggs
65536
call 2 to eggs
256
```

尽管对于装饰函数有用，但是当应用于方法的时候，这种编码方案也有问题（随后更为详细地介绍）。



##### 封闭作用域和全局变量（Enclosing scopes and globals）

闭包函数（closure function），即具有封闭`def`作用域引用和嵌套`def`语句的函数，通常达到相同的效果，特别是针对静态数据，例如：被装饰是的最初的（original）函数。然而，在这个例子中，我们也需要封闭的作用域中的一个计数器，它随着每次调用而更改。由于Python 2.X不支持`nonlocal`语句，所以，在Python 2.X中，我们可以使用类和属性（正如我们前面所做的那样），或者使用全局声明把状态变量移出到 **全局作用域** ：

```python
calls = 0
    def tracer(func):                    # State via enclosing scope and global
        def wrapper(*args, **kwargs):    # Instead of class attributes
            global calls                 # calls is global, not per-function
            calls += 1
            print('call %s to %s' % (calls, func.__name__))
            return func(*args, **kwargs)
        return wrapper

@tracer
def spam(a, b, c):      # Same as: spam = tracer(spam)
    print(a + b + c)

@tracer
def eggs(x, y):         # Same as: eggs = tracer(eggs)
    print(x ** y)

spam(1, 2, 3)           # Really calls wrapper, assigned to spam
spam(a=4, b=5, c=6)     # wrapper calls spam

eggs(2, 16)             # Really calls wrapper, assigned to eggs
eggs(4, y=4)            # Global calls is not per-decoration here!
```

遗憾的是，把计数器移出到共同的全局作用域允许像这样修改它们，也意味着它们将为每个包装的函数所共享。和类实例属性不同，全局计数器是跨程序的，而不是针对每个函数的——对于任何跟踪的函数调用，计数器都会递增。如果你比较这个版本与前一个版本的输出，就可以看出其中的区别：

```
c:\code> python decorator3.py
call 1 to spam
6
call 2 to spam
15
call 3 to eggs
65536
call 4 to eggs
256
```



##### 封闭作用域和非本地变量（Enclosing scopes and nonlocals）

如果我们真的想要一个针对每个函数的计数器，要么像前面那样使用类，要么使用Python 3.X中的`nonlocal`语句。由于这一新的语句允许修改封闭的函数作用域变量，所以它们可以充当针对每次装饰的、可修改的数据：

```python
def tracer(func):                     # State via enclosing scope and nonlocal
    calls = 0                         # Instead of class attrs or global
    def wrapper(*args, **kwargs):     # calls is per-function, not global
        nonlocal calls
        calls += 1
        print('call %s to %s' % (calls, func.__name__))
        return func(*args, **kwargs)
    return wrapper

@tracer
def spam(a, b, c):      # Same as: spam = tracer(spam)
    print(a + b + c)

@tracer
def eggs(x, y):         # Same as: eggs = tracer(eggs)
    print(x ** y)

spam(1, 2, 3)           # Really calls wrapper, bound to func
spam(a=4, b=5, c=6)     # wrapper calls spam

eggs(2, 16)             # Really calls wrapper, bound to eggs
eggs(4, y=4)            # Nonlocal calls _is_ per-decoration here
```

现在，由于封装的作用域变量不能跨程序而成为全局的，所以每个包装的函数再次有了自己的计数器，就像是针对类和属性一样。这里是在Python 3.X下运行时新的输出：

```
c:\code> py −3 decorator4.py
call 1 to spam
6
call 2 to spam
15
call 1 to eggs
65536
call 2 to eggs
256
```



##### 函数属性

最后，如果你没有使用Python 3.X并且没有`nonlocal`语句，可能仍然能够针对某些可改变的状态使用函数属性来避免全局和类。从Python 2.1 开始，我们可以把任意属性分配给函数以附加它们，使用`func.attr = value`就可以了。因为工厂函数在每次调用时创建一个新的函数，所以它的属性能保存每次调用的状态信息。

在我们的例子中，可以直接对状态使用`wrapper.calls`。如下的代码与前面的`nonlocal`版本一样地工作，因为计数器再一次是针对每个装饰的函数的，但是，它也可以在Python 2.X下运行：

```python
def tracer(func):                      # State via enclosing scope and func attr
    def wrapper(*args, **kwargs):      # calls is per-function, not global
        wrapper.calls += 1
        print('call %s to %s' % (wrapper.calls, func.__name__))
        return func(*args, **kwargs)
    wrapper.calls = 0
    return wrapper

@tracer
def spam(a, b, c):       # Same as: spam = tracer(spam)
    print(a + b + c)

@tracer
def eggs(x, y):          # Same as: eggs = tracer(eggs)
    print(x ** y)
    
spam(1, 2, 3)            # Really calls wrapper, assigned to spam
spam(a=4, b=5, c=6)      # wrapper calls spam

eggs(2, 16)              # Really calls wrapper, assigned to eggs
eggs(4, y=4)             # wrapper.calls _is_ per-decoration here
```

注意，这种方法有效，只是因为名称`wrapper`保持在封闭的`tracer`函数的作用域中。



#### 类错误1：装饰类方法



##### 使用嵌套函数来装饰方法

如果想要函数装饰器在简单函数和类方法上都能工作，最直接的解决方法之一是：把自己的函数装饰器编写为嵌套的`def`，以便对于包装器类实例和主体类实例都不需要依赖于单个的`self`实例参数。

如下的替代方案使用Python 3.X的`nonlocal`。由于被装饰的方法重新绑定到简单的函数而不是实例对象，所以Python正确地传递了`Person`对象作为第一个参数，并且装饰器将其从`*args`中的第一项传递给真正的、装饰的方法的`self`参数：

```python
# calltracer.py
# A call tracer decorator for both functions and methods

def tracer(func):                     # Use function, not class with __call__
    calls = 0                         # Else "self" is decorator instance only!
    def onCall(*args, **kwargs):      # Or in 2.X+3.X: use [onCall.calls += 1]
        nonlocal calls
        calls += 1
        print('call %s to %s' % (calls, func.__name__))
        return func(*args, **kwargs)
    return onCall


if __name__ == '__main__':
    # Applies to simple functions
    @tracer
    def spam(a, b, c):                       # spam = tracer(spam)
        print(a + b + c)                     # onCall remembers spam
        
    @tracer
    def eggs(N):
        return 2 ** N
    
    spam(1, 2, 3)                            # Runs onCall(1, 2, 3)
    spam(a=4, b=5, c=6)
    print(eggs(32))
    
    # Applies to class-level method functions too!
    class Person:
        def __init__(self, name, pay):
            self.name = name
            self.pay = pay

        @tracer
        def giveRaise(self, percent):        # giveRaise = tracer(giveRaise)
            self.pay *= (1.0 + percent)      # onCall remembers giveRaise

        @tracer
        def lastName(self):                  # lastName = tracer(lastName)
            return self.name.split()[-1]

    print('methods...')
    bob = Person('Bob Smith', 50000)
    sue = Person('Sue Jones', 100000)
    print(bob.name, sue.name)
    sue.giveRaise(.10)                       # Runs onCall(sue, .10)
    print(int(sue.pay))
    print(bob.lastName(), sue.lastName())    # Runs onCall(bob), lastName in scopes
```

这个版本在函数和方法上都有效：

```
c:\code> py −3 calltracer.py
call 1 to spam
6
call 2 to spam
15
call 1 to eggs
4294967296
methods...
Bob Smith Sue Jones
call 1 to giveRaise
110000
call 1 to lastName
call 2 to lastName
Smith Jones
```



##### 使用描述符装饰方法

由于描述符的`__get__`方法在调用的时候接收描述符类和主体类实例，因此当我们需要装饰器的状态以及最初的类实例来分派调用的时候，它很适合于装饰方法。考虑如下的替代的跟踪装饰器，它也是一个描述符：

```python
class tracer(object): # A decorator+descriptor
    def __init__(self, func): # On @ decorator
        self.calls = 0 # Save func for later call
        self.func = func
    def __call__(self, *args, **kwargs): # On call to original func
        self.calls += 1
        print('call %s to %s' % (self.calls, self.func.__name__))
        return self.func(*args, **kwargs)
    def __get__(self, instance, owner): # On method attribute fetch
        return wrapper(self, instance)

class wrapper:
    def __init__(self, desc, subj): # Save both instances
        self.desc = desc # Route calls back to deco/desc
        self.subj = subj
    def __call__(self, *args, **kwargs):
        return self.desc(self.subj, *args, **kwargs) # Runs tracer.__call__

    
@tracer
def spam(a, b, c): # spam = tracer(spam)
    print(a + b + c)  # Uses __call__ only

@tracer
def eggs(N):
    return 2 ** N

# Applies to class-level method functions too!
class Person:
    def __init__(self, name, pay):
        self.name = name
        self.pay = pay
    @tracer      # giveRaise = tracer(giveRaise)
    def giveRaise(self, percent):      # Makes giveRaise a descriptor
        self.pay *= (1.0 + percent)
    @tracer
    def lastName(self):                  
        return self.name.split()[-1]
```

这和前面嵌套函数的代码一样有效。其操作因上下文而异：

- 被装饰的 **函数** 只调用其`__call__`方法，而不会调用其`__get___`方法。
- 被装饰的 **方法** 首先调用其`__get___`方法来解析`instance.method`方法名获取；`__get__`方法返回的`wrapper`类实例对象同时保存着装饰器（同时也是描述符）实例和主体类实例，然后调用`wrapper`类实例对象来完成调用表达式，并由此触发装饰器`tracer`的`__call__`方法。

例如，要测试代码的调用：

```python
>>> sue = Person('Sue Jones', 100000)
>>> sue
<__main__.Person object at 0x7f3b93a8b390>
>>> type(sue)
<class '__main__.Person'>
>>> type(sue.giveRaise)      # giveRaise 其实是tracer的实例，但获取tracer的类实例属性会触发tracer.__get__(), 从而导致返回了一个wrapper的实例，而调用wrapper的实例会触发wrapper.__call__()，并最终调用tracer.__call__();
<class '__main__.wrapper'>
>>> sue.giveRaise(.10)       # Runs tracer.__get__()  -->  wrapper.__call__()  -->  tracer.__call__()
call 1 to giveRaise
>>> sue.giveRaise(.10)
call 2 to giveRaise
>>> sue.giveRaise(.10)
call 3 to giveRaise
```

首先运行`tracer.__get__`，因为`Person`类的`giveRaise`属性已经通过函数装饰器重新绑定为一个描述符。然后，调用表达式触发返回的`wrapper`类实例对象的`__call__`方法，它返回来调用`tracer.__call__`。

**此外，我们也可以使用一个嵌套的函数和封闭的作用域引用来实现同样的效果。** 如下的版本和前面的版本一样的有效，通过为一个嵌套函数和作用域引用交换类和对象属性，但是，它所需的代码显著减少：

```python
class tracer(object):
    def __init__(self, func):              # On @ decorator
        self.calls = 0                     # Save func for later call
        self.func = func
    def __call__(self, *args, **kwargs):   # On call to original func
        self.calls += 1
        print('call %s to %s' % (self.calls, self.func.__name__))
        return self.func(*args, **kwargs)
    def __get__(self, instance, owner):              # On method fetch
        def wrapper(*args, **kwargs):                # Retain both inst
            return self(instance, *args, **kwargs)   # Runs tracer.__call__
        return wrapper
```

**如果你想要装饰器在简单函数和类方法上都有效，最好使用基于嵌套函数的编码模式，而不是带有调用拦截的类。**



#### 计时调用

为了展示函数装饰器的各种各样能力的一个特殊样例，让我们来看一种不同的应用场景。下一个装饰器将对一个被装饰的函数的调用进行计时——既有针对一次调用的时间，也有所有调用的总的时间。该装饰器应用于两个函数，以便比较列表解析和`map`内置调用所需的时间：

```python
# File timerdeco1.py
# Caveat: range still differs - a list in 2.X, an iterable in 3.X
# Caveat: timer won't work on methods as coded (see quiz solution)

import time, sys
force = list if sys.version_info[0] == 3 else (lambda X: X)

class timer:
    def __init__(self, func):
        self.func = func
        self.alltime = 0
    def __call__(self, *args, **kargs):
        start = time.clock()
        result = self.func(*args, **kargs)
        elapsed = time.clock() - start
        self.alltime += elapsed
        print('%s: %.5f, %.5f' % (self.func.__name__, elapsed, self.alltime))
        return result

@timer
def listcomp(N):
    return [x * 2 for x in range(N)]

@timer
def mapcall(N):
    return force(map((lambda x: x * 2), range(N)))


if __name__ = "__main__":
    result = listcomp(5)        # Time for this call, all calls, return value
    listcomp(50000)
    listcomp(500000)
    listcomp(1000000)
    print(result)
    print('allTime = %s' % listcomp.alltime)      # Total time for all listcomp 

    print('')
    result = mapcall(5)
    mapcall(50000)
    mapcall(500000)
    mapcall(1000000)
    print(result)
    print('allTime = %s' % mapcall.alltime)       # Total time for all mapcall calls

    print('\n**map/comp = %s' % round(mapcall.alltime / listcomp.alltime, 3))
```

当在 Python 3.X和2.X中运行这段代码时，这个文件的自测试代码的输出如下：

```
c:\code> py −3 timerdeco1.py
listcomp: 0.00001, 0.00001
listcomp: 0.00499, 0.00499
listcomp: 0.05716, 0.06215
listcomp: 0.11565, 0.17781
[0, 2, 4, 6, 8]
allTime = 0.17780527629411225

mapcall: 0.00002, 0.00002
mapcall: 0.00988, 0.00990
mapcall: 0.10601, 0.11591
mapcall: 0.21690, 0.33281
[0, 2, 4, 6, 8]
allTime = 0.3328064956447921

**map/comp = 1.872
```



##### 装饰器对比每次调用计时（Decorators versus per-call timing）

```python
>>> def listcomp(N): [x * 2 for x in range(N)]
>>> import timer                      # Chapter 21 techniques
>>> timer.total(1, listcomp, 1000000)
(0.1461295268088542, None)
>>> import timeit
>>> timeit.timeit(number=1, stmt=lambda: listcomp(1000000))
0.14964829430189397
```



#### 添加装饰器参数

为了使前面小节介绍的计时器装饰器更加可配置，我们对它适当编码，使它支持装饰器参数，以此来指定配置选项，这些选项可以根据每个装饰的函数而编码。例如，可以像下面这样添加标签：

```python
def timer(label=''):
    def decorator(func):
        def onCall(*args):         # Multilevel state retention:
            ...                    # args passed to function
            func(*args)            # func retained in enclosing scope
            print(label, ...       # label retained in enclosing scope
        return onCall
    return decorator               # Returns the actual decorator
      
@timer('==>')                      # Like listcomp = timer('==>')(listcomp)
def listcomp(N): ...               # listcomp is rebound to new onCall
                  
listcomp(...)                      # Really calls onCall
```

我们可以把这种结构用于定时器之中，来允许在装饰的时候传入一个标签和一个跟踪控制标志。下面是这么做的一个例子，编码在一个名为`timerdeco2.py`的模块文件中，以便它可以作为一个通用工具导入。它使用类作为第二个状态保留级别，而不是嵌套函数，但是最终结果是类似的：

```python
# timerdeco2.py

import time

def timer(label='', trace=True): # On decorator args: retain args
    class Timer:
        def __init__(self, func): # On @: retain decorated func
            self.func = func
            self.alltime = 0
        def __call__(self, *args, **kargs): # On calls: call original
            start = time.clock()
            result = self.func(*args, **kargs)
            elapsed = time.clock() - start
            self.alltime += elapsed
            if trace:
                format = '%s %s: %.5f, %.5f'
                values = (label, self.func.__name__, elapsed, self.alltime)
                print(format % values)
            return result
    return Timer
```

我们在这里做的主要是把最初的`Timer`类嵌入一个封闭的函数中，以便创建一个作用域以保持装饰器参数。外围的`timer`函数在装饰发生前调用，并且它只是返回`Timer`类作为实际的装饰器。在装饰时，创建了`Timer`的一个实例来记住装饰函数自身，而且访问了位于封闭的函数作用域中的装饰器参数。

不是把`self`测试代码嵌入这个文件，我们将在一个不同的文件中运行装饰器。下面是时间装饰器的一个客户，模块文件`testseqs.py`，再次将其应用于序列迭代器替代方案：

```python
# testseqs.py

import sys
from timerdeco2 import timer

force = list if sys.version_info[0] == 3 else (lambda X: X)

@timer(label='[CCC]==>')
def listcomp(N): # Like listcomp = timer(...)(listcomp)
    return [x * 2 for x in range(N)] # listcomp(...) triggers Timer.__call__

@timer(trace=True, label='[MMM]==>')
def mapcall(N):
    return force(map((lambda x: x * 2), range(N)))

for func in (listcomp, mapcall):
    result = func(5) # Time for this call, all calls, return value
    func(50000)
    func(500000)
    func(1000000)
    print(result)
    print('allTime = %s\n' % func.alltime) # Total time for all calls

print('**map/comp = %s' % round(mapcall.alltime / listcomp.alltime, 3))
```

输出如下：

```
c:\code> py −3 testseqs.py
[CCC]==> listcomp: 0.00001, 0.00001
[CCC]==> listcomp: 0.00504, 0.00505
[CCC]==> listcomp: 0.05839, 0.06344
[CCC]==> listcomp: 0.12001, 0.18344
[0, 2, 4, 6, 8]
allTime = 0.1834406801777564

[MMM]==> mapcall: 0.00003, 0.00003
[MMM]==> mapcall: 0.00961, 0.00964
[MMM]==> mapcall: 0.10929, 0.11892
[MMM]==> mapcall: 0.22143, 0.34035
[0, 2, 4, 6, 8]
allTime = 0.3403542519173618

**map/comp = 1.855
```

与通常一样，我们也可以交互地测试它，看看配置参数是如何应用的：

```python
>>> from timerdeco2 import timer
>>> @timer(trace=False) # No tracing, collect total time
... def listcomp(N):
...     return [x * 2 for x in range(N)]
...
>>> x = listcomp(5000)
>>> x = listcomp(5000)
>>> x = listcomp(5000)
>>> listcomp.alltime
0.0037191417530599152
>>> listcomp
<timerdeco2.timer.<locals>.Timer object at 0x02957518>

>>> @timer(trace=True, label='\t=>') # Turn on tracing, custom label
... def listcomp(N):
...     return [x * 2 for x in range(N)]
...
>>> x = listcomp(5000)
=> listcomp: 0.00106, 0.00106
>>> x = listcomp(5000)
=> listcomp: 0.00108, 0.00214
>>> x = listcomp(5000)
=> listcomp: 0.00107, 0.00321
>>> listcomp.alltime
0.003208920466562404
```



### 39.4 编写类装饰器

类装饰器应用于类，它们可以用于管理类自身，或者用来拦截实例创建调用以管理实例。

#### 单例类（Singleton Classes）

由于类装饰器可以拦截实例创建调用，所以它们可以用来管理一个类的所有实例，或者扩展这些实例的接口。为了说明这点，这里的第一个类装饰器示例做了前面一项工作——管理一个类的所有实例。这段代码实现了传统的 ***单例编码模式*** ，其中最多只有一个类的一个实例存在。其单例（singleton）函数定义并返回一个函数用于管理实例，并且`@`语法自动在这个函数中包装了一个主体类（subject class）：

```python
# singletons.py

# 3.X and 2.X: global table
instances = {}                         # 为了同时支持2.X和3.X，使用全局变量
def singleton(aClass):                 # On @ decoration
    def onCall(*args, **kwargs):       # On instance creation
        if aClass not in instances:    # One dict entry per class
            instances[aClass] = aClass(*args, **kwargs)
        return instances[aClass]
    return onCall
```

用它来装饰想要增强单例模型的类：

```python
# singletons.py

@singleton # Person = singleton(Person)
class Person:                                 # Rebinds Person to onCall
    def __init__(self, name, hours, rate):    # onCall remembers Person
        self.name = name
        self.hours = hours
        self.rate = rate
    def pay(self):
        return self.hours * self.rate

@singleton # Spam = singleton(Spam)
class Spam:                                   # Rebinds Spam to onCall
    def __init__(self, val):                  # onCall remembers Spam
        self.attr = val

bob = Person('Bob', 40, 10)                   # onCall('Bob', 40, 10)
print(bob.name, bob.pay())

sue = Person('Sue', 50, 20)                   # Same, single object
print(sue.name, sue.pay())

X = Spam(val=42)                              # One Person, one Spam
Y = Spam(99)
print(X.attr, Y.attr)
```

现在，当`Person`或`Spam`类稍后用来创建一个实例的时候，装饰器提供的包装逻辑层把实例构造（construction）调用指向了`onCall`，它确保每个类一个单个实例，而不管进行了多少次构建（construction）调用。这段代码的输出如下：

```
c:\code> python singletons.py
Bob 400
Bob 400
42 42
```



##### 替代编码方案（Coding alternatives）

使用`nonlocal`语句（Python 3.X可用）来改变封闭的作用域名称，我们在这里可以编写一个更为自包含的解决方案——后面的替代方案实现了同样的效果，它为每个类使用了一个 ***封闭作用域*** ，而不是为每个类使用一个全局表入口：

```python
# 3.X only: nonlocal
def singleton(aClass): # On @ decoration
    instance = None                              # 每次调用singleton都重新分配一个新的局部变量instance
    def onCall(*args, **kwargs):                 # On instance creation
        nonlocal instance                        # 3.X and later nonlocal
        if instance == None:
            instance = aClass(*args, **kwargs)   # One scope per class
        return instance
    return onCall                                # 被装饰的类实际上被重新绑定为onCall函数对象
```



在Python 3.X或2.6及其以上版本中，你也可以使用 **函数属性** 或者 **类** 来实现自包含的解决方案（self-contained solution）。

下例中，由于对象的命名空间可以扮演和封闭作用域相同的角色，所以可以利用 ***函数属性*** 实现每个`onCall`函数对应一个装饰操作：

```python
# 3.X and 2.X: func attrs (alternative codings)
def singleton(aClass):                                # On @ decoration
    def onCall(*args, **kwargs):                      # On instance creation
        if onCall.instance == None:
            onCall.instance = aClass(*args, **kwargs) # One function per class
        return onCall.instance
    onCall.instance = None
    return onCall

```

下例中，使用每个实例对应一个装饰操作，而不是封闭作用域、函数对象或者全局作用域：

```python
# 3.X and 2.X: classes (alternative codings)
class singleton:
    def __init__(self, aClass):                          # On @ decoration
        self.aClass = aClass
        self.instance = None
    def __call__(self, *args, **kwargs):                 # On instance creation
        if self.instance == None:
            self.instance = self.aClass(*args, **kwargs) # One instance per class
        return self.instance
```



#### 跟踪对象接口

类装饰器的另一个常用场景是用来增强每个产生的实例的接口。类装饰器基本上可以在实例上安装一个包装器（wrapper）或代理（proxy）逻辑层，来以某种方式管理对其接口的访问。

例如，在第31章中，`__getattr__`运算符重载方法作为包装嵌入的实例的整个对象接口的一种方法，以便实现委托编码模式。当获取未定义的属性名的时候，`__getattr__`会运行；我们可以使用这个钩子来拦截一个控制器类中的方法调用，并将它们传递给一个嵌入的对象。

为了便于参考，这里给出最初的非装饰器委托示例，它在两个内置类型对象上工作：

```python
class Wrapper:
    def __init__(self, object):
        self.wrapped = object                    # Save object
    def __getattr__(self, attrname):
        print('Trace:', attrname)                # Trace fetch
        return getattr(self.wrapped, attrname)   # Delegate fetch
    
>>> x = Wrapper([1,2,3])                         # Wrap a list
>>> x.append(4)                                  # Delegate to list method
Trace: append
>>> x.wrapped                                    # Print my member
[1, 2, 3, 4]
>>> x = Wrapper({"a": 1, "b": 2})                # Wrap a dictionary
>>> list(x.keys())                               # Delegate to dictionary method
Trace: keys                                      # Use list() in 3.X
['a', 'b']
```

在这段代码中，`Wrapper`类拦截了对任何包装的对象的属性的访问，打印出一条跟踪信息，并且使用内置函数`getattr`来终止对包装对象的请求。它特别跟踪包装的对象的类之外发出的属性访问。

##### 使用类装饰器跟踪接口

类装饰器为编写这种`__getattr__`技术来包装一个完整接口提供了一个替代的、方便的方法。例如，在Python 2.6和Python 3.X中，前面的类示例可能编写为一个类装饰器，来触发包装的实例创建，而不是把一个预产生的实例传递到包装器的构造函数中（在这里也用`**kargs`扩展了，以支持关键字参数，并且统计进行访问的次数）：

```python
# interfacetracer.py

def Tracer(aClass): # On @ decorator
    class Wrapper:
        def __init__(self, *args, **kargs):         # On instance creation
            self.fetches = 0
            self.wrapped = aClass(*args, **kargs)   # Use enclosing scope name
        def __getattr__(self, attrname):
            print('Trace: ' + attrname)             # Catches all but own attrs
            self.fetches += 1
            return getattr(self.wrapped, attrname)  # Delegate to wrapped obj
    return Wrapper

if __name__ == '__main__':
    @Tracer
    class Spam: # Spam = Tracer(Spam)
        def display(self):                          # Spam is rebound to Wrapper
            print('Spam!' * 8)

    @Tracer
    class Person: # Person = Tracer(Person)
        def __init__(self, name, hours, rate):      # Wrapper remembers Person
            self.name = name
            self.hours = hours
            self.rate = rate
        def pay(self):                              # Accesses outside class traced
            return self.hours * self.rate           # In-method accesses not traced

    food = Spam()                                   # Triggers Wrapper()
    food.display()                                  # Triggers __getattr__
    print([food.fetches])

    bob = Person('Bob', 40, 50)                     # bob is really a Wrapper
    print(bob.name)                                 # Wrapper embeds a Person
    print(bob.pay())

    print('')
    sue = Person('Sue', rate=100, hours=60)         # sue is a different Wrapper
    print(sue.name)                                 # with a different Person
    print(sue.pay())

    print(bob.name)                                 # bob has different state
    print(bob.pay())
    print([bob.fetches, sue.fetches])               # Wrapper attrs not traced
```

这里与我们前面在“编写函数装饰器”一节中遇到的跟踪器装饰器有很大不同。相反，通过拦截实例创建调用，这里的类装饰器允许我们跟踪整个对象接口，例如，对其任何属性的访问。

```python
c:\code> python interfacetracer.py
Trace: display                                 # food.display()
Spam!Spam!Spam!Spam!Spam!Spam!Spam!Spam!       
[1]                                            # print([food.fetches])
Trace: name                                    # print(bob.name)
Bob                                            
Trace: pay                                     # print(bob.pay())
2000

Trace: name                                    # print(sue.name)
Sue
Trace: pay                                     # print(sue.pay())
6000
Trace: name                                    # print(bob.name)
Bob
Trace: pay                                     # print(bob.pay())
2000
[4, 2]                                         # print([bob.fetches, sue.fetches])
```



##### 对内置类型应用类装饰器（Applying class decorators to built-in types）

注意，前面的代码装饰了一个用户定义的类。就像是在本书第31章最初的例子中一样，我们也可以使用装饰器来包装一个内置的类型，例如列表，只要我们的子类允许装饰器语法，或者手动地执行装饰——对于`@`所在的行，装饰器语法需要一条`class`语句。

在下面的代码中，由于装饰的间接作用，`x`实际是一个`Tracer`：

```python
>>> from interfacetracer import Tracer

>>> @Tracer
... class MyList(list): pass # MyList = Tracer(MyList)

>>> x = MyList([1, 2, 3]) # Triggers Wrapper()
>>> x.append(4) # Triggers __getattr__, append
Trace: append
>>> x.wrapped
[1, 2, 3, 4]

>>> WrapList = Tracer(list) # Or perform decoration manually
>>> x = WrapList([4, 5, 6]) # Else subclass statement required
>>> x.append(7)
Trace: append
>>> x.wrapped
[4, 5, 6, 7]
```

这种装饰器方法允许我们把实例创建移动到装饰器自身之中，而不是要求传入一个预先生成的对象。尽管这好像是一个细小的差别，它允许我们保留常规的实例创建语法并且通常实现装饰器的所有优点。我们只需要用装饰器语法来扩展类，而不是要求所有的实例创建调用都通过一个包装器来手动地指向对象：

```python
@Tracer # Decorator approach
class Person: ...
bob = Person('Bob', 40, 50)
sue = Person('Sue', rate=100, hours=60)

class Person: ... # Nondecorator approach
bob = Wrapper(Person('Bob', 40, 50))
sue = Wrapper(Person('Sue', rate=100, hours=60))
```

假设你将会产生类的多个实例，并希望将这些增强特性应用于类的每个实例，装饰器通常将会在代码大小和代码可维护性上双赢。



#### 类错误2：保持多个实例

使用正确的运算符重载协议，前一节例子中的装饰器函数看似可以不编写为一个函数，而编写为如下的一个类，但这会带来一个问题。

```python
class Tracer:
    def __init__(self, aClass):              # On @decorator
        self.aClass = aClass                 # Use instance attribute
    def __call__(self, *args):               # On instance creation
        self.wrapped = self.aClass(*args)    # ONE (LAST) INSTANCE PER CLASS!
        return self
    def __getattr__(self, attrname):
        print('Trace: ' + attrname)
        return getattr(self.wrapped, attrname)
    
@Tracer # Triggers __init__
class Spam: # Like: Spam = Tracer(Spam)
    def display(self):
        print('Spam!' * 8)

food = Spam()         # Triggers __call__
food.display()        # Triggers __getattr__
```

正如我们在前面见到的，这个替代方案能够像前面一样处理多个类，但是，它对于一个给定的类的多个实例并不是很有效：每个实例构建调用会触发`__call__`，这会覆盖前面的实例。直接效果是`Tracer`只保存了一个实例，即最后创建的一个实例。自行体验一下看看这是如何发生的，但是，这里给出该问题的一个示例：

```python
@Tracer
class Person: # Person = Tracer(Person)
    def __init__(self, name):   # Wrapper bound to Person
        self.name = name
        
bob = Person('Bob')             # bob is really a Wrapper
print(bob.name)                 # Wrapper embeds a Person
Sue = Person('Sue')
print(sue.name)                 # sue overwrites bob
print(bob.name)                 # OOPS: now bob's name is 'Sue'!
```

这段代码输出如下——由于这个跟踪器只有一个共享的实例，所以第二个实例覆盖了第一个实例：

```
Trace: name
Bob
Trace: name
Sue
Trace: name
Sue
```

**我们为每个类创建了一个装饰器实例，但是不是针对每个类实例，这样一来，只有最后一个实例保持住了。这导致无法对同一个类的不同实例进行有效的状态保持。其解决方案就像我们在前面针对装饰方法的类错误一样，在于放弃基于类的装饰器。**

前面的基于函数的`Tracer`版本确实可用于多个实例，因为每个实例构建调用都会创建一个新的`Wrapper`实例，而不是覆盖一个单个的共享的`Tracer`实例的状态。由于同样的原因，最初的非装饰器版本正确地处理多个实例。



#### 装饰器VS管理器函数

```python
class Spam: # Nondecorator version
    ... # Any class will do
food = Wrapper(Spam()) # Special creation syntax

@Tracer
class Spam: # Decorator version
    ... # Requires @ syntax at class
food = Spam() # Normal creation syntax
```



```python
instances = {}
def getInstance(aClass, *args, **kwargs):
    if aClass not in instances:
        instances[aClass] = aClass(*args, **kwargs)
    return instances[aClass]

bob = getInstance(Person, 'Bob', 40, 10)        # Versus: bob = Person('Bob', 40, 10)
```



```python
instances = {}
def getInstance(object):
    aClass = object.__class__
    if aClass not in instances:
        instances[aClass] = object
    return instances[aClass]

bob = getInstance(Person('Bob', 40, 10))      # Versus: bob = Person('Bob', 40, 10)
```



```python
def func(x, y): # Nondecorator version
    ... # def tracer(func, args): ... func(*args)
result = tracer(func, (1, 2)) # Special call 

@tracer
def func(x, y): # Decorator version
    ... # Rebinds name: func = tracer(func)
result = func(1, 2) # Normal call syntax
```



#### 为什么使用装饰器

***类装饰器* 有2个潜在的缺陷：**

- **类型修改：** 正如我们所见到的，当插入包装器的时候，一个装饰器函数或类不会保持其最初的类型——其名称重新绑定到一个包装器对象，在使用对象名称或测试对象类型的程序中，这可能会很重要。在单体的例子中，装饰器和管理函数的方法都为实例保持了最初的类类型；在跟踪器的代码中，没有一种方法这么做，因为需要有包装器。
- **额外调用：** 通过装饰添加一个包装层，在每次调用装饰对象的时候，会引发一次额外调用所需的额外性能成本——调用是相对耗费时间的操作，因此，装饰包装器可能会使程序变慢。在跟踪器代码中，两种方法都需要每个属性通过一个包装器层来指向；单体的示例通过保持最初的类类型而避免了额外调用。

类似的问题也适用于 ***函数装饰器*** ：装饰和管理器函数都会导致额外调用，并且当装饰的时候通常会发生类型变化（不装饰的时候就没有）。

**装饰器有3个主要优点：**

- **明确的语法：** 装饰器使得扩展明确而显然。它们的@比可能在源文件中任何地方出现的特殊代码要容易识别，例如，在单体和跟踪器实例中，装饰器行似乎比额外代码更容易被注意到。此外，装饰器允许函数和实例创建调用使用所有Python程序员所熟悉的常规语法。
- **代码可维护性：** 装饰器避免了在每个函数或类调用中重复扩展的代码。由于它们只出现一次，在类或者函数自身的定义中，它们排除了冗余性并简化了未来的代码维护。对于我们的单体和跟踪器示例，要使用管理器函数的方法，我们需要在每次调用的时候使用特殊的代码——最初以及未来必须做出的任何修改都需要额外的工作。
- **一致性：** 装饰器使得程序员忘记使用必需的包装逻辑的可能性大大减少。这主要得益于两个优点——由于装饰是显式的并且只出现一次，出现在装饰的对象自身中，与必须包含在每次调用中的特殊代码相比较，装饰器促进了更加一致和统一的API使用。例如，在单体示例中，可能更容易忘了通过特殊代码来执行所有类创建调用，而这将会破坏单体的一致性管理。

装饰器还促进了代码的封装以减少冗余性，并使得未来的维护代价最小化。尽管其他的编码结构化工具也能做到这些，但装饰器使得这对于扩展任务来说更自然。



### 39.5 直接管理函数和类

因为装饰器通过装饰器代码来运行新的函数和类，从而有效地工作，它们也可以用来管理函数和类对象自身，而不只是管理对它们随后的调用。

如下的简单实现定义了一个装饰器，它既应用于函数也应用于类，把对象添加到一个基于字典的注册中。由于它返回对象本身而不是一个包装器，所以它没有拦截随后的调用：

```python
# Registering decorated objects to an API
from __future__ import print_function # 2.X

registry = {}
def register(obj): # Both class and func decorator
    registry[obj.__name__] = obj # Add to registry
    return obj # Return obj itself, not a wrapper

@register
def spam(x):
    return(x ** 2) # spam = register(spam)

@register
def ham(x):
    return(x ** 3)

@register
class Eggs: # Eggs = register(Eggs)
    def __init__(self, x):
        self.data = x ** 4
    def __str__(self):
        return str(self.data)
    
print('Registry:')
for name in registry:
    print(name, '=>', registry[name], type(registry[name]))
    
print('\nManual calls:')
print(spam(2)) # Invoke objects manually
print(ham(2)) # Later calls not intercepted
X = Eggs(2)
print(X)

print('\nRegistry calls:')
for name in registry:
print(name, '=>', registry[name](2)) # Invoke from registry
```

当这段代码运行的时候，装饰的对象按照名称添加到注册中，但当随后调用它们的时候，它们仍然按照最初的编码工作，而没有指向一个包装器层。实际上，我们的对象可以手动运行，或从注册表内部运行：

```
c:\code> py −3 registry-deco.py
Registry:
spam => <function spam at 0x02969158> <class 'function'>
ham => <function ham at 0x02969400> <class 'function'>
Eggs => <class '__main__.Eggs'> <class 'type'>

Manual calls:
4
8
16

Registry calls:
spam => 4
ham => 8
Eggs => 16
```

例如，函数装饰器也可能用来处理函数属性，并且类装饰器可能动态地插入新的类属性，或者甚至新的方法。考虑如下的函数装饰器——它们把函数属性分配给记录信息，以便随后供一个API使用，但是，它们没有插入一个包含器层来拦截随后的调用：

```python
# Augmenting decorated objects directly
>>> def decorate(func):
        func.marked = True     # Assign function attribute for later use
        return func

>>> @decorate
    def spam(a, b):
        return a + b

>>> spam.marked
True

>>> def annotate(text):        # Same, but value is decorator argument
        def decorate(func):
            func.label = text
            return func
        return decorate
    
>>> @annotate('spam data')
    def spam(a, b):            # spam = annotate(...)(spam)
        return a + b

>>> spam(1, 2), spam.label
(3, 'spam data')
```

注意，再次强调，装饰器是一个可调用对象，它接收可调用对象作为其参数，并返回一个可调用对象。装饰器所返回的可调用对象可是被装饰的可调用对象本身，也可以是另一个全新的可调用对象。当使用装饰器来管理被装饰的可调用对象的调用时，装饰器通常返回另一个全新的可调用对象；当使用装饰器来直接管理被装饰的可调用对象时，装饰器通常返回被装饰的可调用对象本身。



### 39.6 实例：“私有”和“公有”属性

#### 实现Private属性

如下的类装饰器实现了一个用于类实例属性的`Private`声明，也就是说，属性存储在一个实例上，或者从其一个类继承而来。不接受从装饰的类的外部对这样的属性的获取和修改访问，但是，仍然允许类自身在其方法中自由地访问那些名称。

这里的版本扩展了这一概念以验证属性获取，并且它使用委托而不是继承来实现该模型。实际上，在某种意义上，这只是我们前面遇到的属性跟踪器类装饰器的一个扩展。

尽管这个例子利用了类装饰器的新语法糖来编写私有属性，但它的属性拦截最终仍然是基于我们在前面各章介绍的`__getattr__`和`__setattr__`运算符重载方法。当检测到访问一个私有属性的时候，这个版本使用`raise`语句引发一个异常，还有一条出错消息；异常可能在一个`try`中捕获，或者允许终止脚本。

代码如下所示，在文件的底部还有一个self测试。它在Python 3.X和Python 2.6以上版本都能够工作，因为它使用了Python 3.X的`print`和`raise`语法，尽管它在Python 2.X下只是捕获运算符重载方法属性：

```python
"""
File access1.py (3.X + 2.X)

Privacy for attributes fetched from class instances.
See self-test code at end of file for a usage example.

Decorator same as: Doubler = Private('data', 'size')(Doubler).
Private returns onDecorator, onDecorator returns onInstance,
and each onInstance instance embeds a Doubler instance.
"""

traceMe = False
def trace(*args):
    if traceMe: print('[' + ' '.join(map(str, args)) + ']')
        
def Private(*privates):                                 # privates in enclosing scope
    def onDecorator(aClass):                            # aClass in enclosing scope
        class onInstance:                               # wrapped in instance attribute
            def __init__(self, *args, **kargs):
                self.wrapped = aClass(*args, **kargs)
            def __getattr__(self, attr):                # My attrs don't call getattr
                trace('get:', attr)                     # Others assumed in wrapped
                if attr in privates:
                    raise TypeError('private attribute fetch: ' + attr)
                else:
                    return getattr(self.wrapped, attr)
            def __setattr__(self, attr, value):         # Outside accesses
                trace('set:', attr, value)              # Others run normally
                if attr == 'wrapped':                   # Allow my attrs
                    self.__dict__[attr] = value         # Avoid looping
                elif attr in privates:
                    raise TypeError('private attribute change: ' + attr)
                else:
                    setattr(self.wrapped, attr, value)  # Wrapped obj attrs
        return onInstance                               # Or use __dict__
    return onDecorator

if __name__ == '__main__':
    traceMe = True
    
    @Private('data', 'size') # Doubler = Private(...)(Doubler)
    class Doubler:
        def __init__(self, label, start):
            self.label = label                          # Accesses inside the subject class
            self.data = start                           # Not intercepted: run normally
        def size(self):
            return len(self.data)                       # Methods run with no checking
        def double(self):                               # Because privacy not inherited
            for i in range(self.size()):
                self.data[i] = self.data[i] * 2
        def display(self):
            print('%s => %s' % (self.label, self.data))

    X = Doubler('X is', [1, 2, 3])
    Y = Doubler('Y is', [-10, −20, −30])

    # The following all succeed
    print(X.label)                                      # Accesses outside subject class
    X.display(); X.double(); X.display()                # Intercepted: validated, delegated
    print(Y.label)
    Y.display(); Y.double()
    Y.label = 'Spam'
    Y.display()
    
    # The following all fail properly
    """
    print(X.size())                   # prints "TypeError: private attribute fetch: size"
    print(X.data)
    X.data = [1, 1, 1]
    X.size = lambda S: 0
    print(Y.data)
    print(Y.size())
    """
```

当`traceMe`为`True`的时候，模块文件的self测试代码产生如下的输出。注意，装饰器是如何捕获和验证在包装的类之外运行的属性获取和赋值的，但是，却没有捕获类自身内部的属性访问：

```
c:\code> py −3 access1.py
[set: wrapped <__main__.Doubler object at 0x00000000029769B0>]
[set: wrapped <__main__.Doubler object at 0x00000000029769E8>]
[get: label]
X is
[get: display]
X is => [1, 2, 3]
[get: double]
[get: display]
X is => [2, 4, 6]
[get: label]
Y is
[get: display]
Y is => [-10, −20, −30]
[get: double]
[set: label Spam]
[get: display]
Spam => [−20, −40, −60]
```



#### 实现细节1

##### 继承 VS 委托 (Inheritance versus delegation)

第30章中给出的粗糙的私有示例使用继承来混入`__setattr__`捕获访问。然而，继承使得这很困难，因为从类的内部或外部的访问之间的区分不是很直接的（内部访问应该允许常规运行，并且外部的访问应该限制）。要解决这个问题，第30章的示例需要继承类，以使用`__dict__`赋值来设置属性，这最多是一个不完整的解决方案。

这里的版本使用的委托（在另一个对象中嵌入一个对象），而不是继承。这种模式更好地适合于我们的任务，因为它使得区分主体对象的内部访问和外部访问容易了很多。对主体对象的来自外部的属性访问，由包装器层的重载方法拦截，并且如果合法的话，委托给类。类自身内部的访问（例如，通过其方法代码内的`self`）没有拦截并且允许不经检查而常规运行，因为这里没有继承私有的属性。

##### 装饰器参数

这里使用的类装饰器接受任意多个参数，以命名私有属性。然而，真正发生的情况是，参数传递给了`Private`函数，并且`Private`返回了应用于主体类的装饰器函数。也就是说，在装饰器发生之前使用这些参数；`Private`返回装饰器，装饰器反过来把私有的列表作为一个封闭作用域应用来“记住”。

##### 状态保持和封闭作用域 (State retention and enclosing scopes)

说到封闭的作用域，在这段代码中，实际上用到了3个层级的状态保持：

- `Private`的参数在装饰发生前使用，并且作为一个封闭作用域引用保持，以用于`onDecorator`和`onInstance`中。
- `onDecorator`的类参数在装饰时使用，并且作为一个封闭作用域引用保持，以便在实例构建时使用。
- 包装的实例对象保存为`onInstance`中的一个实例属性，以便随后从类外部访问属性的时候使用。

##### 使用`__dcit__`和`__slots__`以及其他虚拟变量名

这段代码中`__setattr__`依赖于一个实例对象的`__dict__`属性命名空间字典，以设置`onInstance`自己的包装属性。正如我们在上一章所了解到的，不能直接赋值一个属性而避免循环。然而，它使用了setattr内置函数而不是`__dict__`来设置包装对象自身之中的属性。此外，`getattr`用来获取包装对象中的属性，因为它们可能存储在对象自身中或者由对象继承。

因此，这段代码将对大多数类有效，包括那些基于slots，特性 (properties) ，甚至`__getattr__`等具有“虚拟”类级别 (class-level) 的属性。

你可能还记得，在第32章中介绍过，带有`__slots__`的新式类不能把属性存储到一个`__dict__`中。然而，由于我们在这里只是在`onInstance`层级依赖于一个`__dict__`，而不是在包装的实例中，并且因为`setattr`和`getattr`应用于基于`__dict__`和`__slots__`的属性，所以我们的装饰器应用于使用任何一种存储方案的类。同理，装饰器也适用于新式的特性和类似工具。不管装饰器代理对象本身的属性如何，被委托的变量名将会在被包装的实例中重新查找。



#### Public声明的泛化 (Generalizing for Public Declarations, Too)

既然有了一个`Private`实现，泛化其代码以考虑`Public`声明就很简单了——它们基本上是`Private`声明的反过程，因此，我们只需要取消内部测试。本节列出的实例允许一个类使用装饰器来定义一组`Private`或`Public`的实例属性（存储在一个实例上的属性，或者从其类继承的属性），使用如下的语法：

- `Private`声明类实例的那些不能获取或赋值的属性，而从类的方法的代码内部获取或赋值除外。也就是说，任何声明为`Private`的名称都不能从类的外部访问，而任何没有声明为`Private`的名称都可以自由地从类的外部获取或赋值。
- `Public`声明了一个类的实例属性，它可以从类的外部以及在类的方法内部获取和访问。也就是说，声明为`Public`的任何名称，可以从任何地方自由地访问，而没有声明为`Public`的任何名称，不能从类的外部访问。

`Private`和`Public`声明规定为互斥的：当使用了`Private`，所有未声明的名称都被认为是`Public`的；并且当使用了`Public`，所有未声明的名称都被认为是`Private`。它们基本上相反，尽管未声明的、不是由类方法创建的名称行为略有不同——它们可以赋值并且由此从类的外部在`Private`之下创建（所有未声明的名称都是可以访问的），但不是在`Public`下创建的（所有未声明的名称都是不可访问的）。

注意，这个方案在顶层添加了额外的第4层状态保持，超越了前面描述的3个层次：`lambda`所使用的测试函数保存在一
个额外的封闭作用域中。这个示例编写为可以在Python 3.X和2.6及其以上版本中运行，尽管它在Python 3.X下运行的时候带有一个缺陷（在文件的文档字符串之后简短地说明，并且在代码之后详细说明）：

```python
"""
File access2.py (3.X + 2.X)

Class decorator with Private and Public attribute declarations.

Controls external access to attributes stored on an instance, or
Inherited by it from its classes. Private declares attribute names
that cannot be fetched or assigned outside the decorated class,
and Public declares all the names that can.

Caveat: this works in 3.X for explicitly named attributes only: __X__
operator overloading methods implicitly run for built-in operations
do not trigger either __getattr__ or __getattribute__ in new-style
classes. Add __X__ methods here to intercept and delegate built-ins.
"""

traceMe = False
def trace(*args):
    if traceMe: print('[' + ' '.join(map(str, args)) + ']')
        
def accessControl(failIf):
    def onDecorator(aClass):
        class onInstance:
            def __init__(self, *args, **kargs):
                self.__wrapped = aClass(*args, **kargs)
            def __getattr__(self, attr):
                trace('get:', attr)
                if failIf(attr):
                    raise TypeError('private attribute fetch: ' + attr)
                else:
                    return getattr(self.__wrapped, attr)
            def __setattr__(self, attr, value):
                trace('set:', attr, value)
                if attr == '_onInstance__wrapped':      # 因为使用了伪私有变量__wrapped，Python自动将其扩展为_onInstance__wrapped
                    self.__dict__[attr] = value
                elif failIf(attr):
                    raise TypeError('private attribute change: ' + attr)
                else:
                    setattr(self.__wrapped, attr, value)
        return onInstance
    return onDecorator

def Private(*attributes):
    return accessControl(failIf=(lambda attr: attr in attributes))

def Public(*attributes):
    return accessControl(failIf=(lambda attr: attr not in attributes))
```

这里在交互提示模式下快速地看看这些类装饰器的使用。正如所介绍的那样，非`Private`或`Public`名称可以从主体类之外访问和修改，但是`Private`或非`Public`的名称不可以：

```python
>>> from access2 import Private, Public

>>> @Private('age')                            # Person = Private('age')(Person)
    class Person:                              # Person = onInstance with state
        def __init__(self, name, age):
            self.name = name
            self.age = age                     # Inside accesses run normally

>>> X = Person('Bob', 40)
>>> X.name                                     # Outside accesses validated
'Bob'
>>> X.name = 'Sue'
>>> X.name
'Sue'
>>> X.age
TypeError: private attribute fetch: age
>>> X.age = 'Tom'
TypeError: private attribute change: age
        
>>> @Public('name')
    class Person:
        def __init__(self, name, age):
            self.name = name
            self.age = age
            
>>> X = Person('bob', 40)                      # X is an onInstance
>>> X.name                                     # onInstance embeds Person
'bob'
>>> X.name = 'Sue'
>>> X.name
'Sue'
>>> X.age
TypeError: private attribute fetch: age
>>> X.age = 'Tom'
TypeError: private attribute change: age
```



#### 实现细节2

##### 使用`__X`伪私有变量名 (Using `__X` pseudoprivate names)

除了泛化，这个版本还使用了Python的`__X`伪私有名称压缩功能（我们在第31章遇到过），来把包装的属性局部化为控制类，通过自动将其作为类名的前缀就可以做到。这避免了前面的版本与一个真实的、包装类可能使用的包装属性冲突的风险，并且它也是一个有用的通用工具。然而，它不是很“私有”，因为压缩的名称可以在类之外自由地使用。注意，在`__setattr__`中，我们也必须使用完整扩展的名称字符串`'_onInstance__wrapped'`，因为这是Python对其的修改。

##### 打破私有性 (Breaking privacy)

尽管这个例子确实实现了对一个实例及其类的属性的访问控制，它可能以各种方式破坏了这些控制——例如，通过检查包装属性的显式扩展版本（`bob.pay`可能无效，因为完全压缩的`bob._onInstance__wrapped.pay`可能会有效）。如果你必须显式地这么做，这些控制可能对于常规使用来说足够了。当然，私有控制通常在任何语言中都会遭到破坏，如果你足够努力地尝试的话（#define private public在某些C++实现中也可能有效）。尽管访问控制可以减少意外修改，但这样的情况大多取决于使用任何语言的程序员。不管何时，源代码可能会被修改，访问控制总是管道流中的一小部分。

##### 装饰器的权衡 (Decorator tradeoffs)

不用装饰器，我们也可以实现同样的结果，通过使用管理器函数或者手动编写装饰器的名称重绑定；然而，装饰器语法使得代码更加一致而明确。这一方法以及任何其他基于包装的方法的主要潜在缺点是，属性访问导致额外调用，并且装饰的类的实例并不真的是最初的装饰类的实例——例如，如果你用`X.__class__`或`isinstance(X, C)`测试它们的类型，将会发现它们是包装类的实例。除非你计划在对象类型上进行内省，否则类型问题可能是不相关的。



#### 开放问题 (Open Issues)

Most notably, this tool turns in mixed performance on operator overloading methods if they are used by client classes.

As coded, the proxy class is a classic class when run under 2.X, but a new-style classwhen run by 3.X. As such, the code supports any client class in 2.X, but in 3.X fails tovalidate or delegate operator overloading methods dispatched implicitly by built-in operations, unless they are redefined in the proxy. Clients that do not use operator overloading are fully supported, but others may require additional code in 3.X.

Importantly, this is not a new-style class issue here, it’s a Python version issue—the same code runs differently and fails in 3.X only. Because the nature of the wrapped object’s class is irrelevant to the proxy, we are concerned only with the proxy’s own code, which works under 2.X but not 3.X.



##### 缺陷：运算符重载方法无法在Python 3.X下委托 (Caveat: Implicitly run operator overloading methods fail to delegate under 3.X)

就像使用`__getattr__`的所有的基于委托的类，这个装饰器只对常规命名的属性能够跨版本工作。像`__str__`和`__add__`这样在新式类下不同工作的运算符方法，在Python 3.0下运行的时候，如果定义了嵌入的对象，将无法有效到达。

正如我们在上一章所了解到的， 传统类通常在运行时在实例中查找运算符重载名称，但新式类不这么做——它们完全略过实例，在类中查找这样的方法。因此，在Python 2.X的新式类和Python 3.X的所有类中，`__X__`运算符重载方法显式地针对内置操作运行，不会触发`__getattr__`和`__getattribute__`。这样的属性获取将会和我们的`onInstance.__getattr__`一起忽略，因此，它们无法验证或委托。

我们的装饰器类没有编写为新式类（通过派生自`object`），因此，如果在Python 2.X下运行，它将会捕获运算符重载方法。由于在Python 3.X下所有的类自动都是新式类，如果它们在嵌入的对象上编码，这样的方法将会失效。Python 3.X中最简单的解决方案是，在`onInstance`中重新冗余地定义所有那些可能在包装的对象中用到的运算符重载方法。例如，可以手动添加额外的方法，可以通过工具来自动完成部分任务（例如，使用类装饰器或者下一章将要介绍的元类），或者通过在超类中定义。

同样的代码在Python 3.X下运行的时候，显式地调用`__str__`和`__add__`将会忽略装饰器的`__getattr__`，并且在装饰器类之中或其上查找定义；`print`最终查找到从类类型继承的默认显示（从技术上讲，是从Python 3.X中隐藏的`object`超类），并且`+`产生一个错误，因为没有默认继承：

```python
C:\code> c:\python33\python
>>> from access2 import Private
>>> @Private('age')
class Person:
    def __init__(self):
        self.age = 42
    def __str__(self):
        return 'Person: ' + str(self.age)
    def __add__(self, yrs):
        self.age += yrs
>>> X = Person()           # Name validations still work
>>> X.age                  # But 3.X fails to delegate built-ins!
TypeError: private attribute fetch: age
>>> print(X)
<access2.accessControl.<locals>.onDecorator.<locals>.onInstance object at ...etc>
>>> X + 10
TypeError: unsupported operand type(s) for +: 'onInstance' and 'int'
>>> print(X)
<access2.accessControl.<locals>.onDecorator.<locals>.onInstance object at ...etc>
Strangely, this occurs only for dispatch from built-in operations; explicit direct calls
```

使用替代的`__getattribute__`方法在这里帮不上忙——尽管它定义为捕获每次属性引用（而不只是未定义的名称），它也不会由内置操作运行。我们在本书第38章介绍的Python的特性功能，在这里也帮不上忙。

##### Approaches to redefining operator overloading methods for 3.X

正如前面所提到的，Python 3.X中最直接的解决方案是：在类似装饰器的基于委托的类中，冗余地重新定义可能在嵌入对象中出现的运算符重载名称。这种方法并不理想，因为它产生了一些代码冗余，特别是与Python 2.X的解决方案相比较尤其如此。然而，这不会有太大的编码工作，在某种程度上可以使用工具或超类来自动完成，足以使装饰器在Python 3.X下工作，并且也允许运算符重载名称声明为`Private`或`Public`（假设每个运算符重载方法内部都运行failIf测试）。

###### Inline definition.

For instance, the following is an inline redefinition approach—add method redefinitions to the proxy for every operator overloading method a wrapped object may define itself, to catch and delegate. We’re adding just four operation interceptors to illustrate, but others are similar (new code is in bold font here):

```python
def accessControl(failIf):
    def onDecorator(aClass):
        class onInstance:
            def __init__(self, *args, **kargs):
                self.__wrapped = aClass(*args, **kargs)
                
            # Intercept and delegate built-in operations specifically
            def __str__(self):
                return str(self.__wrapped)
            def __add__(self, other):
                return self.__wrapped + other # Or getattr(x, '__add__')(y)
            def __getitem__(self, index):
                return self.__wrapped[index] # If needed
            def __call__(self, *args, **kargs):
                return self.__wrapped(*args, **kargs) # If needed
            # plus any others needed
            
            # Intercept and delegate by-name attribute access generically
            def __getattr__(self, attr): ...
            def __setattr__(self, attr, value): ...
        return onInstance
    return onDecorator
```



###### Mix-in superclasses.

```python
class BuiltinsMixin:
    def __add__(self, other):
        return self.__class__.__getattr__(self, '__add__')(other)
    def __str__(self):
        return self.__class__.__getattr__(self, '__str__')()
    def __getitem__(self, index):
        return self.__class__.__getattr__(self, '__getitem__')(index)
    def __call__(self, *args, **kargs):
        return self.__class__.__getattr__(self, '__call__')(*args, **kargs)
    # plus any others needed

def accessControl(failIf):
    def onDecorator(aClass):
        class onInstance(BuiltinsMixin):
            #...rest unchanged...
            def __getattr__(self, attr): ...
            def __setattr__(self, attr, value): ...
    
class BuiltinsMixin:
    def __add__(self, other):
        return self._wrapped + other # Assume a _wrapped
    def __str__(self): # Bypass __getattr__
        return str(self._wrapped)
    def __getitem__(self, index):
        return self._wrapped[index]
    def __call__(self, *args, **kargs):
        return self._wrapped(*args, **kargs)
    # plus any others needed

def accessControl(failIf):
    def onDecorator(aClass):
        class onInstance(BuiltinsMixin):
            #...and use self._wrapped instead of self.__wrapped...
            def __getattr__(self, attr): ...
            def __setattr__(self, attr, value): ...
```



###### Coding variations: Routers, descriptors, automation.

Naturally, both of the prior section’s mixin superclasses might be improved with additional code changes we’ll largely pass on here, except for two variations worth noting briefly. First, compare the following mutation of the first mix-in—which uses a simpler coding structure but will incur an extra call per built-in operation, making it slower (though perhaps not significantly so in a proxy context):

```python
class BuiltinsMixin:
    def reroute(self, attr, *args, **kargs):
        return self.__class__.__getattr__(self, attr)(*args, **kargs)
    def __add__(self, other):
        return self.reroute('__add__', other)
    def __str__(self):
        return self.reroute('__str__')
    def __getitem__(self, index):
        return self.reroute('__getitem__', index)
    def __call__(self, *args, **kargs):
        return self.reroute('__call__', *args, **kargs)
    # plus any others needed

```

Naturally, both of the prior section’s mixin superclasses might be improved with additional code changes we’ll largely pass on here, except for two variations worth noting briefly. First, compare the following mutation of the first mix-in—which uses a simpler coding structure but will incur an extra call per built-in operation, making it slower (though perhaps not significantly so in aproxy context):

```python
class BuiltinsMixin:
    class ProxyDesc(object): # object for 2.X
        def __init__(self, attrname):
            self.attrname = attrname
        def __get__(self, instance, owner):
            return getattr(instance._wrapped, self.attrname) # Assume a _wrapped
        
    builtins = ['add', 'str', 'getitem', 'call'] # Plus any others

    for attr in builtins:
        exec('__%s__ = ProxyDesc("__%s__")' % (attr, attr))
```



##### Should operator methods be validated?

Adding support for operator overloading methods is required of interface proxies in general, to delegate calls correctly. In our specific privacy application, though, it also raises some additional design choices. In particular, privacy of operator overloading methods differs per implementation:

- Because they invoke `__getattr__`, the rerouter mix-ins require either that all `__X__` names accessed be listed in Public decorations, or that Private be used instead when operator overloading is present in clients. In classes that use overloading heavily, Public may be impractical.
- Because they bypass `__getattr__` entirely, as coded here both the inline scheme and `self._wrapped` mix-ins do not have these constraints, but they preclude builtin operations from being made private, and cause built-in operation dispatch towork asymmetrically from both explicit `__X__` calls by-name and 2.X’s default classic classes.
- Python 2.X classic classes have the first bullet’s constraints, simply because all`__X__` names are routed through `__getattr__` automatically.
- Operator overloading names and protocols differ between 2.X and 3.X, makingtruly cross-version decoration less than trivial (e.g., Public decorators may need tolist names from both lines).

We’ll leave final policy here a TBD, but some interface proxies might prefer to allow `__X__` operator names to always pass unchecked when delegated. In the general case, though, a substantial amount of extra code is required to accommodate 3.X’s new-style classes as delegation proxies—in principle, every operator overloading method that is no longer dispatched as a normal instance attribute automatically will need to be defined redundantly in a general tool class like this privacy decorator. This is why this extension is omitted in our code: there are potentially more than 50 such methods! Because all its classes are new-style, delegation-based code is more difficult—though not necessarily impossible—in Python 3.X.



##### 实现替代：`__getattribute__`插入，调用堆栈检查 (Implementation alternatives: `__getattribute__` inserts, call stack inspection)

如下是对类装饰器代码的潜在修改：

```python
# Method insertion: rest of access2.py code as before
def accessControl(failIf):
    def onDecorator(aClass):
        def getattributes(self, attr):
            trace('get:', attr)
            if failIf(attr):
                raise TypeError('private attribute fetch: ' + attr)
            else:
                return object.__getattribute__(self, attr)
            
        def setattributes(self, attr, value):
            trace('set:', attr)
            if failIf(attr):
                raise TypeError('private attribute change: ' + attr)
            else:
                return object.__setattr__(self, attr, value)
            
        aClass.__getattribute__ = getattributes
        aClass.__setattr__ = setattributes # Insert accessors
        return aClass # Return original class
    return onDecorator
```





#### Python 不是关于控制

既然我已经用如此大的篇幅添加了对Python代码的`Private`和`Public`属性声明，必须再次提醒你，像这样为类添加访问控制一点都不Pythonic。实际上，大多数Python程序员可能发现这一示例太大或者完全无关，除非仅充当装饰器实践的一个演示 (demonstration)。大多数大型的Python程序根本没有任何这样的控制而获得成功。如果你确实想要控制属性访问以杜绝编码错误，或者恰好近乎是专家级的C++或Java程序员，那么使用Python的运算符重载和内省工具，大多数事情是可以完成的。



### 39.7 实例：验证函数参数

作为装饰器工具的最后一个示例，本节开发了一个函数装饰器，它自动地测试传递给一个函数或方法的参数是否在有效的数值范围内。它设计用来在任何开发或产品阶段使用，并且它可以用作类似任务的一个模板（例如，参数类型测试，如果必须这么做的话）。

#### 目标

针对我们现在或将来要编写的任何函数或方法的参数，开发一个通用的工具来自动为我们执行范围测试。装饰器方法使得这明确而方便：

```python
class Person:
    @rangetest(percent=(0.0, 1.0)) # Use decorator to validate
    def giveRaise(self, percent):
        self.pay = int(self.pay * (1 + percent))
```

在装饰器中隔离验证逻辑，这简化了客户类和未来的维护。

注意，我们这里的目标和前面编写的 **属性验证** 不同。这里，我们想要验证传入的 **函数参数的值** ，而不是设置的属性的值。Python的装饰器和内省工具允许我们很容易地编写这一新的任务。

#### 针对位置参数的一个基本范围测试装饰器

为了简化，我们将从编写一个只对位置参数有效的装饰器开始，并且假设它们在每次调用中总是出现在相同的位置。

在名为`rangetest1.py`的文件中编写如下代码：

```python
# rangetest1.py

def rangetest(*argchecks):        # Validate positional arg ranges
    def onDecorator(func):
        if not __debug__:         # True if "python -O main.py args..."
            return func           # No-op: call original directly
        else:                     # Else wrapper while debugging
            def onCall(*args):
                for (ix, low, high) in argchecks:
                    if args[ix] < low or args[ix] > high:
                        errmsg = 'Argument %s not in %s..%s' % (ix, low, high)
                        raise TypeError(errmsg)
                return func(*args)
            return onCall
    return onDecorator
```

这段代码我们使用装饰器参数，嵌套作用域以进行状态保持，等等。我们还使用了嵌套的`def`语句以确保这对简单函数和方法都有效。

还要注意到，这段代码使用了`__debug__`内置变量——Python将其设置为`True`，除非它将以`-O`优化命令行标志运行（例如，`python -O main.py`）。当`__debug__`为`False`的时候，装饰器返回未修改的最初函数，以避免额外调用及其相关的性能损失。

```python
# File rangetest1_test.py

from __future__ import print_function # 2.X
from rangetest1 import rangetest
print(__debug__) # False if "python -O main.py"

@rangetest((1, 0, 120)) # persinfo = rangetest(...)(persinfo)
def persinfo(name, age): # age must be in 0..120
    print('%s is %s years old' % (name, age))

@rangetest([0, 1, 12], [1, 1, 31], [2, 0, 2009])
def birthday(M, D, Y):
    print('birthday = {0}/{1}/{2}'.format(M, D, Y))

class Person:
    def __init__(self, name, job, pay):
        self.job = job
        self.pay = pay

    @rangetest([1, 0.0, 1.0]) # giveRaise = rangetest(...)(giveRaise)
    def giveRaise(self, percent): # Arg 0 is the self instance here
        self.pay = int(self.pay * (1 + percent))

# Comment lines raise TypeError unless "python -O" used on shell command line
persinfo('Bob Smith', 45)                     # Really runs onCall(...) with state
#persinfo('Bob Smith', 200)                   # Or person if -O cmd line argument

birthday(5, 31, 1963)
#birthday(5, 32, 1963)

sue = Person('Sue Jones', 'dev', 100000)
sue.giveRaise(.10)                            # Really runs onCall(self, .10)
print(sue.pay)                                # Or giveRaise(self, .10) if -O
#sue.giveRaise(1.10)
#print(sue.pay)
```

运行的时候，这段代码中的有效调用产生如下的输出：

```
C:\code> python rangetest1_test.py
True
Bob Smith is 45 years old
birthday = 5/31/1963
110000
```

取消任何无效调用的注释会导致`TypeError`被装饰器引发。以下是最后两行代码被允许执行时的输出：

```
C:\code> python rangetest1_test.py
True
Bob Smith is 45 years old
birthday = 5/31/1963
110000
TypeError: Argument 1 not in 0.0..1.0
```

在系统命令行，使用`-O`标志来运行Python，将会关闭范围测试，也会避免包装层的性能负担，最终直接调用最初未装饰的函数。假设这只是一个调试工具，可以使用这个标志来优化程序以供产品阶段使用：

```
C:\code> python -O rangetest1_test.py
False
Bob Smith is 45 years old
birthday = 5/31/1963
110000
231000
```



#### 针对关键字和默认泛化

通过把包装函数的期待参数与调用时实际传入的参数匹配，它支持对按照位置或关键字名称传入的参数的验证，并且对于调用忽略的默认参数，它会跳过测试。

```python
"""
File rangetest.py: function decorator that performs range-test
validation for arguments passed to any function or method.

Arguments are specified by keyword to the decorator. In the actual
call, arguments may be passed by position or keyword, and defaults
may be omitted. See rangetest_test.py for example use cases.
"""
trace = True

def rangetest(**argchecks):       # Validate ranges for both+defaults
    def onDecorator(func):        # onCall remembers func and argchecks
        if not __debug__:         # True if "python -O main.py args..."
            return func           # Wrap if debugging; else use original
        else:
            code = func.__code__
            allargs = code.co_varnames[:code.co_argcount]
            funcname = func.__name__
            
            def onCall(*pargs, **kargs):
                # All pargs match first N expected args by position
                # The rest must be in kargs or be omitted defaults
                expected = list(allargs)
                positionals = expected[:len(pargs)]
                for (argname, (low, high)) in argchecks.items():
                    # For all args to be checked
                    if argname in kargs:
                        # Was passed by name
                        if kargs[argname] < low or kargs[argname] > high:
                            errmsg = '{0} argument "{1}" not in {2}..{3}'
                            errmsg = errmsg.format(funcname, argname, low, high)
                            raise TypeError(errmsg)
                    elif argname in positionals:
                        # Was passed by position
                        position = positionals.index(argname)
                        if pargs[position] < low or pargs[position] > high:
                            errmsg = '{0} argument "{1}" not in {2}..{3}'
                            errmsg = errmsg.format(funcname, argname, low, high)
                            raise TypeError(errmsg)
                    else:
                        # Assume not passed: default
                        if trace:
                            print('Argument "{0}" defaulted'.format(argname))
                return func(*pargs, **kargs) # OK: run original call
            return onCall
    return onDecorator
```

如下的测试脚本展示了如何使用装饰器：

```python
"""
File rangetest_test.py (3.X + 2.X)
Comment lines raise TypeError unless "python -O" used on shell command line
"""
from __future__ import print_function # 2.X
from rangetest import rangetest

# Test functions, positional and keyword

@rangetest(age=(0, 120)) # persinfo = rangetest(...)(persinfo)
def persinfo(name, age):
    print('%s is %s years old' % (name, age))
@rangetest(M=(1, 12), D=(1, 31), Y=(0, 2013))
def birthday(M, D, Y):
    print('birthday = {0}/{1}/{2}'.format(M, D, Y))

persinfo('Bob', 40)
persinfo(age=40, name='Bob')
birthday(5, D=1, Y=1963)
#persinfo('Bob', 150)
#persinfo(age=150, name='Bob')
#birthday(5, D=40, Y=1963)

# Test methods, positional and keyword

class Person:
    def __init__(self, name, job, pay):
        self.job = job
        self.pay = pay
    # giveRaise = rangetest(...)(giveRaise)
    @rangetest(percent=(0.0, 1.0)) # percent passed by name or position
    def giveRaise(self, percent):
        self.pay = int(self.pay * (1 + percent))

bob = Person('Bob Smith', 'dev', 100000)
sue = Person('Sue Jones', 'dev', 100000)
bob.giveRaise(.10)
sue.giveRaise(percent=.20)
print(bob.pay, sue.pay)
#bob.giveRaise(1.10)
#bob.giveRaise(percent=1.20)

# Test omitted defaults: skipped

@rangetest(a=(1, 10), b=(1, 10), c=(1, 10), d=(1, 10))
def omitargs(a, b=7, c=8, d=9):
    print(a, b, c, d)

omitargs(1, 2, 3, 4)
omitargs(1, 2, 3)
omitargs(1, 2, 3, d=4)
omitargs(1, d=4)
omitargs(d=4, a=1)
omitargs(1, b=2, d=4)
omitargs(d=8, c=7, a=1)
#omitargs(1, 2, 3, 11) # Bad d
#omitargs(1, 2, 11) # Bad c
#omitargs(1, 2, 3, d=11) # Bad d
#omitargs(11, d=4) # Bad a
#omitargs(d=4, a=11) # Bad a
#omitargs(1, b=11, d=4) # Bad b
#omitargs(d=8, c=7, a=11) # Bad a
```

这段脚本运行的时候，超出范围的参数会像前面一样引发异常，但参数可以按照名称或位置传递，并且忽略的默认参数不会验证：

```
C:\code> python rangetest_test.py
Bob is 40 years old
Bob is 40 years old
birthday = 5/1/1963
110000 120000
1 2 3 4
Argument "d" defaulted
1 2 3 9
1 2 3 4
Argument "c" defaulted
Argument "b" defaulted
1 7 8 4
Argument "c" defaulted
Argument "b" defaulted
1 7 8 4
Argument "c" defaulted
1 2 8 4
Argument "b" defaulted
1 7 7 8
```

对于验证错误，当方法测试行之一注释掉的时候，我们像前面一样得到一个异常（除非`-0`命令行参数传递给Python）：

```
TypeError: giveRaise argument "percent" not in 0.0..1.0
```



#### 实现细节

##### 函数内省 (Function introspection)

已经证实了内省API可以在函数对象以及其拥有我们所需的工具的相关代码对象上实现。期待的参数名集合只是附加给一个函数的代码对象的前`N`个变量名：

```python
# In Python 3.X (and 2.6+ for compatibility)
>>> def func(a, b, c, e=True, f=None): # Args: three required, two defaults
x = 1 # Plus two more local variables
y = 2
>>> code = func.__code__ # Code object of function object
>>> code.co_nlocals
7
>>> code.co_varnames # All local variable names
('a', 'b', 'c', 'e', 'f', 'x', 'y')
>>> code.co_varnames[:code.co_argcount] # <== First N locals are expected args
('a', 'b', 'c', 'e', 'f')
```



##### 参数假设

给定期待的参数名的这个集合，该解决方案依赖于Python对于参数传递顺序所施加的两条限制（在Python 2.X和Python 3.X中都仍然成立）：

- 在调用时，所有的位置参数出现在所有关键字参数之前。
- 在`def`中，所有的非默认参数出现在所有的默认参数之前。

##### 匹配算法





#### 开放问题
##### 无效调用 (Invalid calls)

##### 任意参数 (Arbitrary arguments)

##### 装饰器嵌套 (Decorator nesting)



#### 装饰器参数 VS 函数注解

Python 3.X中提供的函数注解功能，可能为我们在指定范围测试的示例中所使用的装饰器参数给出了一种替代方法。正如我们在第19章中学到的，注解允许我们把表达式和参数及返回值关联起来，通过在自己的def头部行中编写它们。Python把注解收集到字典中并且将其附加给注解的函数。

我们可以在示例中的标题行编写范围限制，而不是在装饰器参数中编写。我们将仍然需要一个函数装饰器来包装函数以拦截随后的调用，但我们基本上换掉了装饰器参数语法：

```python
@rangetest(a=(1, 5), c=(0.0, 1.0))
def func(a, b, c): # func = rangetest(...)(func)
    print(a + b + c)
```

注解语法如下：

```python
@rangetest
def func(a:(1, 5), b, c:(0.0, 1.0)):
    print(a + b + c)
```

装饰器参数编码模式就是我们前面给出的完整解决方案，注解替代方案需要一个较少层级的嵌套，因为它不需要保持装饰器参数：

```python
# Using decorator arguments (3.X + 2.X)
def rangetest(**argchecks):             # 装饰器参数版本的信息保持在封闭作用域的一个参数中
    def onDecorator(func):
        def onCall(*pargs, **kargs):
            print(argchecks)
            for check in argchecks:
                pass # Add validation code here
            return func(*pargs, **kargs)
        return onCall
    return onDecorator

@rangetest(a=(1, 5), c=(0.0, 1.0))
def func(a, b, c): # func = rangetest(...)(func)
    print(a + b + c)
func(1, 2, c=3) # Runs onCall, argchecks in scope


# Using function annotations (3.X only)
def rangetest(func):
    def onCall(*pargs, **kargs):
        argchecks = func.__annotations__    # 注解版本的信息保持在函数自身的一个属性中
        print(argchecks)
        for check in argchecks:
            pass # Add validation code here
        return func(*pargs, **kargs)
    return onCall

@rangetest
def func(a:(1, 5), b, c:(0.0, 1.0)): # func = rangetest(func)
    print(a + b + c)

func(1, 2, c=3)
```

运行的时候，两种方案都会访问同样的验证测试信息，但以不同的形式——装饰器参数版本的信息保持在封闭作用域的一个参数中；注解版本的信息保持在函数自身的一个属性中：

```
C:\code> py −3 decoargs-vs-annotation.py
{'a': (1, 5), 'c': (0.0, 1.0)}
6
{'a': (1, 5), 'c': (0.0, 1.0)}
6
```





#### 其他应用程序：类型测试

在处理装饰器参数时，我们所使用的编码模式可以应用于其他环境。例如，在开发时检查参数数据类型，这是一种直接的扩展：

```python
def typetest(**argchecks):
    def onDecorator(func):
        ...
        def onCall(*pargs, **kargs):
            positionals = list(allargs)[:len(pargs)]
            for (argname, type) in argchecks.items():
                if argname in kargs:
                    if not isinstance(kargs[argname], type):
                        ...
                        raise TypeError(errmsg)
                elif argname in positionals:
                    position = positionals.index(argname)
                    if not isinstance(pargs[position], type):
                        ...
                        raise TypeError(errmsg)
                else:
                    # Assume not passed: default
            return func(*pargs, **kargs)
        return onCall
    return onDecorator

@typetest(a=int, c=float)
def func(a, b, c, d): # func = typetest(...)(func)
    ...
    
func(1, 2, 3.0, 4) # OK
func('spam', 2, 99, 4) # Triggers exception correctly
```

正如前面的小节所述，对于这样的一个装饰器使用函数注解而不是装饰器参数，将会使其看起来更像是其他语言中的类型声明：

```python
@typetest
def func(a: int, b, c: float, d): # func = typetest(func)
    ... # Gasp!...
```

这一特定角色通常是工作代码中的一个糟糕思路，并且一点都不 *Pythonic*（实际上，它往往是有经验的C++程序员初次尝试使用Python的一种现象）。







---



## 第40章 元类 (Metaclasses)

函数装饰器和类装饰器允许我们拦截并扩展函数调用以及类实例创建调用。以类似的思路，元类允许我们拦截并扩展类创建——它们提供了一个API以插入在一条`class`语句结束时运行的额外逻辑，尽管是以与装饰器不同的方式。同样，它们提供了一种通用的协议来管理程序中的类对象。

另一方面，元类为各种没有它而难以实现或不可能实现的编码模式打开了大门，并且对于那些追求编写灵活的API或编程工具供其他人使用的程序员来说，它特别有用。



### 40.1 如果你不知道为什么要使用元类，那就不要使用它

> 如果你犹豫是否需要元类，那你不需要它们（真正需要元类的人，能够确定地知道需要它们，并且不需要说明为什么需要）。

换句话说，元类主要是针对那些构建API和工具供他人使用的程序员。在很多情况下（如果不是大多数的话），它们可能不是应用程序工作的最佳选择。

然而，元类有着各种各样广泛的潜在角色，并且知道它们何时有用是很重要的。例如，它们可能用来扩展具有跟踪、对象持久、异常日志等功能的类。它们也可以用来在运行时根据配置文件来构建类的一部分，对一个类的每个方法广泛地应用函数装饰器，验证其他的接口的一致性，等等。元类甚至可以用来实现替代的编程模式，例如面向方面编程 (aspect-oriented programming)、数据库的对象/关系映射（ORM），等等。

#### 提高魔力层次

- **内省属性：** 像`__class__`和`__dict__`这样的特殊属性允许我们查看Python对象的内部实现方面，以便更广泛地处理它们，列出对象的所有属性、显示一个类名，等等。
- **运算符重载方法：** 像`__str__`和`__add__`这样特殊命名的方法，在类中编写来拦截并提供应用于类实例的内置操作的行为，例如，打印、表达式运算符等等。它们自动运行作为对内置操作的响应，并且允许类符合期望的接口。
- **属性拦截方法：** 一类特殊的运算符重载方法提供了一种方法在实例上广泛地拦截属性访问：`__getattr__`、`__setattr__`和`__getattribute__`允许包装的类插入自动运行的代码，这些代码可以验证属性请求并且将它们委托给嵌入的对象。它们允许一个对象的任意数目的属性——要么是选取的属性，要么是所有的属性——在访问的时候计算。
- **类特性：** 内置函数`property`允许我们把代码和特殊的类属性关联起来，当获取、赋值或删除该属性的时候就自动运行代码。尽管不像前面一段所介绍的工具那样通用，特性考虑到了访问特定属性时候的自动代码调用。
- **类属性描述符：** 其实，特性只是定义根据访问自动运行函数的属性描述符的一种简洁方式。描述符允许我们在单独的类中编写`__get__`、`__set__`和`__delete__`处理程序方法，当分配给该类的一个实例的属性被访问的时候自动运行它们。它们提供了一种通用的方式，来插入当访问一个特定的属性时自动运行的代码，并且在一个属性的常规查找之后触发它们。
- **函数和类装饰器：** 装饰器的特殊的`@`可调用语法，允许我们添加当调用一个函数或创建一个类实例的时候自动运行的逻辑。这个包装器逻辑可以跟踪或计时调用，验证参数，管理类的所有实例，用诸如属性获取验证的额外行为来扩展实例，等等。装饰器语法插入名称重新绑定逻辑，在函数或类定义语句的末尾自动运行该逻辑——装饰的函数和类名重新绑定到拦截了随后调用的可调用对象。
- **元类：** 元类是这些技术的延续——它们允许我们在一条`class`语句的末尾，插入当创建一个类对象的时候自动运行的逻辑。这个逻辑不会把类名重新绑定到一个装饰器可调用对象，而是把类自身的创建指向特定的逻辑。



##### 语言的钩子

换句话说，元类最终只是定义自动运行代码的另外一种方式。通过元类以及前面列出的其他工具，Python为我们提供了在各种环境中插入逻辑的方法——在运算符计算时、属性访问时、函数调用时、类实例创建时，现在是在类对象创建时。It’s a language with hooks galore—a feature open to abuse like any other, but one that also offers the flexibility that some programmers desire, and that some programs may require.

正如我们已经看到的，这些高级Python工具中的很多都有交叉的角色。例如，属性往往可以用特性、描述符或属性拦截方法来管理。正如我们在本章中见到的，类装饰器和元类往往可以交换使用：

- 尽管类装饰器常常用来管理实例，它们也可以用来管理类；
- 类似的，尽管元类设计用来扩展类构建，它们也常常插入代码来管理实例。

In fact, the main functional difference between these two tools is simply their place in the timing of class creation. As we saw in the prior chapter, class decorators run after the decorated class has already been created. Thus, they are often used to add logic to be run at instance creation time. When they do provide behavior for a class, it is typically through changes or proxies, instead of a more direct relationship.

As we’ll see here, metaclasses, by contrast, run during class creation to make and return the new client class. Therefore, they are often used for managing or augmenting classes themselves, and can even provide methods to process the classes that are created from them, via a direct instance relationship.

For example, metaclasses can be used to add decoration to all methods of classes automatically, register all classes in use to an API, add user-interface logic to classes automatically, create or extend classes from simplified specifications in text files, and so on. Because they can control how classes are made—and by proxy the behavior their instances acquire—metaclass applicability is potentially very wide.

As we’ll also see here, though, these two tools are more similar than different in many common roles. Since tool choices are sometimes partly subjective, knowledge of the alternatives can help you pick the right tool for a given task. To understand the options better, let’s see how metaclasses stack up.



#### 管理器函数的缺点

就像装饰器一样，元类常常是可选的。我们通常可以通过管理器函数 (manager functions) 传递类对象来实现同样的效果，这和我们通过管理器代码传递函数和实例来实现装饰器的目的很相似。然而，就像装饰器一样，元类：

- 提供一种更为正式和明确的结构。
- 有助于确保应用程序员不会忘记根据一个API需求来扩展他们的类。
- 通过把类定制逻辑工厂化到一个单独的位置（元类）中，避免代码冗余及其相关的维护成本。

从维护的角度，把选择逻辑隔离到一个单独的地方，这可能会更好。我们可以通过把类指向一个管理器函数，从而把一些这些额外工作的一部分封装起来——这样的一个管理器函数将根据需求扩展类，并且处理运行时测试和配置的所有工作：

```python
def extra(self, arg): ...
    
def extras(Class): # Manager function: too manual
    if required():
        Class.extra = extra

class Client1: ...
extras(Client1)

class Client2: ...
extras(Client2)

class Client3: ...
extras(Client3)

X = Client1()
X.extra()
```

这段代码通过紧随类创建之后一个管理器函数来运行类。尽管像这样的一个管理器函数在这里可以实现我们的目标，但它们仍然给类的编写者增加了相当重的负担，所以编写者必须理解需求并且将它们附加到代码中。

**元类** 提供一种简单的方式在主体类中增强这种扩展。只需要在客户类中声明一个元类，在创建新类的时候，Python在`class`语句的末尾自动调用元类，客户类将自动获取元类所提供的扩展。因此，使用元类可以根据需要扩展、注册或管理类。

```python
def extra(self, arg): ...
    
class Extras(type):
    def __init__(Class, classname, superclasses, attributedict):
        if required():
            Class.extra = extra

class Client1(metaclass=Extras): ... # Metaclass declaration only (3.X form)
class Client2(metaclass=Extras): ... # Client class is instance of meta
class Client3(metaclass=Extras): ...

X = Client1() # X is instance of Client1
X.extra()
```



#### 元类VS类装饰器：第1回合

类装饰器有时候在功能上与元类有重合。尽管类装饰器通常用来管理或扩展实例，但它们也可以扩展类，而独立于任何创建的实例。由于类装饰器通过自动把一个类名绑定到一个函数的结果，那么，没有理由在任何实例创建之前不能用它来扩展类。也就是说，在创建的时候，类装饰器可以对类应用额外的逻辑，而不只是对实例应用：

```python
def extra(self, arg): ...
    
def extras(Class):
    if required():
        Class.extra = extra
    return Class

@extras
class Client1: ... # Client1 = extras(Client1)

@extras
class Client2: ... # Rebinds class independent of instances

@extras
class Client3: ...

X = Client1() # Makes instance of augmented class
X.extra() # X is instance of original Client1
```

这里，装饰器基本上会把前面示例的手动名称重新绑定自动化。就像是使用元类，由于装饰器返回最初的类，实例由此创建，而不是由包装器对象创建。实际上，根本没有拦截实例创建。

由于类装饰器和元类都可以管理类，所以在这个特定例子中，装饰器对应到元类的`__init__`方法，但是，元类还有其他的定制钩子。

**尽管装饰器可以管理实例和类，但元类是设计来管理类的，用元类来管理实例则不是很容易。**




### 40.2 元类模型

#### 类是`type`的实例

- 在Python 3.X 中，用户定义的类对象是名为`type`的对象的实例，`type`本身是一个类。
- 在Python 2.X 中，新式类继承自`object`，它是`type`的一个子类；传统类是`type`的一个实例，并且并不创建自一个类。

```python
C:\code> py −3 # In 3.X:
>>> type([]), type(type([])) # List instance is created from list class
(<class 'list'>, <class 'type'>) # List class is created from type class
>>> type(list), type(type) # Same, but with type names
(<class 'type'>, <class 'type'>) # Type of type is type: top of hierarchy
```

Python 3.X中，“类型”的概念与“类”的概念合并了，类就是类型，类型也是类。即：

- 类对象（类型）由派生自`type`的类定义。
- 类对象是`type`类的实例。
- 一个实例对象的类型是产生它类。

实际上，类现在有了连接到`type`的一个`__class__`，就像是一个实例有了链接到创建它的类的`__class__`：

```python
C:\code> py −3
>>> class C: pass # 3.X class object (new-style)
>>> X = C() # Class instance object
>>> type(X) # Instance is instance of class
<class '__main__.C'>
>>> X.__class__ # Instance's class
<class '__main__.C'>
>>> type(C) # Class is instance of type
<class 'type'>
>>> C.__class__ # Class's class is type
<class 'type'>
```



#### 元类是`type`的子类

由于类实际上是`type`类的实例，从`type`的定制的子类创建类允许我们实现各种定制的类。在Python 3.X和Python 2.6以上版本的新式类中：

- `type`是产生用户定义类对象的一个类。
- 元类是`type`类的一个子类。
- 类对象是`type`类的一个实例或一个子类。
- 实例对象产生自一个类。

换句话说，为了控制创建类以及扩展其行为的方式，我们所需要做的只是指定一个用户定义的类创建自一个用户定义的元类，而不是常规的`type`类。

#### class语句协议

从技术上讲，Python遵从一个标准的协议来处理`class`语句：在一条`class`语句的末尾，并在运行了一个命名空间词典中的所有嵌套代码之后，它调用`type`对象来创建`class`对象：

```python
class = type(classname, superclasses, attributedict)
```

`type`对象定义了一个`__call__`运算符重载方法，当调用`type`对象的时候，该方法运行两个其他的方法：

```python
type.__new__(typeclass, classname, superclasses, attributedict)
type.__init__(class, classname, superclasses, attributedict)
```

`__new__`方法创建并返回了新的`class`对象，并且随后`__init__`方法初始化了新创建的对象。这是`type`的元类子类通常用来定制类的钩子。

例如，给定一个如下所示的类定义：

```python
class Eggs: ... # Inherited names here
    
class Spam(Eggs): # Inherits from Eggs
    data = 1 # Class data attribute
    def meth(self, arg): # Class method attribute
        return self.data + arg
```

Python将会从内部运行嵌套的代码块来创建该类的两个属性（`data`和`meth`），然后在`class`语句的末尾调用`type`对象，产生`class`对象：

```python
Spam = type('Spam', (Eggs,), {'data': 1, 'meth': meth, '__module__': '__main__'})
```

实际上，你可以用这种方式调用`type`来动态地创建类对象。例如，在Python 3.X和2.X中，尽管这里有一个虚构的方法函数和空的超类元组，Python都会自动创建对象：

```python
>>> x = type('Spam', (), {'data': 1, 'meth': (lambda x, y: x.data + y)})
>>> i = x()
>>> x, i
(<class '__main__.Spam'>, <__main__.Spam object at 0x029E7780>)
>>> i.data, i.meth(2)
(1, 3)
```

这个产生的类与运行一个`class`语句得到的类是完全一样的：

```python
>>> x.__bases__
(<class 'object'>,)
>>> [(a, v) for (a, v) in x.__dict__.items() if not a.startswith('__')]
[('data', 1), ('meth', <function <lambda> at 0x0297A158>)]
```

因为这个`type`调用是在`class`语句末尾自动被执行的，所以它是扩展类或以其他方式处理类的理想钩子。诀窍在于用一个自定义子类替换默认`type`，这个子类将拦截这个调用。



### 40.3 声明元类

要告诉Python用一个定制的元类来创建一个类，直接声明一个元类来拦截常规的类创建调用。

#### 在3.X中声明元类

在Python 3.X中，在类头部 (header) 中把想要的元类作为一个关键字参数列出来：

```python
class Spam(metaclass=Meta): # 3.X version (only)
```

继承的超类也可以列在类头部中，例如，下面的新类`Spam`继承自`Eggs`，但同时也是元类`Meta`的实例并且由`Meta`创建：

```python
class Spam(Eggs, metaclass=Meta): # Normal supers OK: must list first
```

在这个形式中，超类必须列在元类前面。

#### Metaclass Dispatch in Both 3.X and 2.X

当以这些方式声明的时候，创建类对象的调用在`class`语句的底部运行，修改为调用元类而不是默认的`type`：

```python
class = Meta(classname, superclasses, attributedict)
```

由于元类是`type`的一个子类，所以`type`类的`__call__`把创建和初始化新的类对象的调用委托给元类，如果它定义了这些方法的定制版本：

```python
Meta.__new__(Meta, classname, superclasses, attributedict)
Meta.__init__(class, classname, superclasses, attributedict)
```

为了演示，这里再次给出前一节的例子，用Python 3.X的元类声明扩展：

```python
class Spam(Eggs, metaclass=Meta): # Inherits from Eggs, instance of Meta
    data = 1 # Class data attribute
    def meth(self, arg): # Class method attribute
        return self.data + arg
```

在这条`class`语句的末尾，Python内部运行如下的代码来创建`class`对象：

```python
Spam = Meta('Spam', (Eggs,), {'data': 1, 'meth': meth, '__module__': '__main__'})
```

如果元类定义了`__new__`或`__init__`的自己版本，在此处的调用期间，它们将依次由继承的`type`类的`__call__`方法调用，以创建并初始化新类。



### 40.4 编写元类

#### 基本元类

只带有一个`__new__`方法的`type`的子类可能就是最简单的元类了。例如，下例中的元类`Meta`只有一个`__new__`方法，它执行所需的任何定制并且调用超类`type`的`__new__`方法来创建并运行新的类对象：

```python
class Meta(type):
    def __new__(meta, classname, supers, classdict):
        # Run by inherited type.__call__
        return type.__new__(meta, classname, supers, classdict)
```

下面这个更具实用性的实例，将打印添加到元类和文件以便追踪：

```python
# metaclass1.py

class MetaOne(type):
    def __new__(meta, classname, supers, classdict):
        print('In MetaOne.new:', meta, classname, supers, classdict, sep='\n...')
        return type.__new__(meta, classname, supers, classdict)
    
class Eggs:
    pass

print('making class')
class Spam(Eggs, metaclass=MetaOne): # Inherits from Eggs, instance of MetaOne
    data = 1 # Class data attribute
    def meth(self, arg): # Class method attribute
        return self.data + arg
    
print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

在这里，`Spam`继承自`Eggs`并且是`MetaOne`的一个实例，但是`X`是`Spam`的一个实例并且继承自它。在我们真正创建一个实例之前，元类用来处理类，并且类用来处理实例：

```python
c:\code> py −3 metaclass1.py
making class
In MetaOne.new:
...<class '__main__.MetaOne'>
...Spam
...(<class '__main__.Eggs'>,)
...{'data': 1, 'meth': <function Spam.meth at 0x02A191E0>, '__module__': '__main__'}
making instance
data: 1 3
```

以上输出省略了命名空间中部分不相关的内置`__X__`变量名。



#### 定制构建和初始化

元类也可以接入`__init__`协议，由`type`对象的`__call__`调用：通常，`__new__`创建并返回了类对象，`__init__`初始化了已经创建的类。元类也可以用做在创建时管理类的钩子：

```python
# metaclass2.py

class MetaTwo(type):
    def __new__(meta, classname, supers, classdict):
        print('In MetaTwo.new: ', classname, supers, classdict, sep='\n...')
        return type.__new__(meta, classname, supers, classdict)
    def __init__(Class, classname, supers, classdict):
        print('In MetaTwo.init:', classname, supers, classdict, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))
        
class Eggs:
    pass

print('making class')
class Spam(Eggs, metaclass=MetaTwo): # Inherits from Eggs, instance of MetaTwo
    data = 1 # Class data attribute
    def meth(self, arg): # Class method attribute
        return self.data + arg

print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

在这个例子中，类初始化方法在类构建方法之后运行，但是，两者都在`class`语句最后运行，并且在创建任何实例之前运行：

```python
c:\code> py −3 metaclass2.py
making class
In MetaTwo.new:
...Spam
...(<class '__main__.Eggs'>,)
...{'data': 1, 'meth': <function Spam.meth at 0x02967268>, '__module__': '__main__'}
In MetaTwo.init:
...Spam
...(<class '__main__.Eggs'>,)
...{'data': 1, 'meth': <function Spam.meth at 0x02967268>, '__module__': '__main__'}
...init class object: ['__qualname__', 'data', '__module__', 'meth', '__doc__']
making instance
data: 1 3
```



#### 其他元类编程技巧

尽管重新定义`type`超类的`__new__`和`__init__`方法是元类向类对象创建过程插入逻辑的最常见方法，其他的方案也是可能的。

##### 使用简单的工厂函数

例如，元类根本不是真的需要类。正如我们所学习的，`class`语句发布了一条简单的调用，在其处理的最后创建了一个类。因此，实际上任何可调用对象都可以用作一个元类，只要它接收传递的参数并且返回与目标类兼容的一个对象。实际上，一个简单的对象工厂函数就像一个类一样工作：

```python
# metaclass3.py
# A simple function can serve as a metaclass too

def MetaFunc(classname, supers, classdict):
    print('In MetaFunc: ', classname, supers, classdict, sep='\n...')
    return type(classname, supers, classdict)

class Eggs:
    pass

print('making class')
class Spam(Eggs, metaclass=MetaFunc): # Run simple function at end
    data = 1 # Function returns class
    def meth(self, arg):
        return self.data + arg

print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

运行的时候，在声明`class`语句的末尾调用该函数，并且它返回期待的新的类对象。该函数直接捕获`type`对象的`__call__`通常会默认拦截的调用：

```python
c:\code> py −3 metaclass3.py
making class
In MetaFunc:
...Spam
...(<class '__main__.Eggs'>,)
...{'data': 1, 'meth': <function Spam.meth at 0x029471E0>, '__module__': '__main__'}
making instance
data: 1 3
```



##### 使用普通类来重载类创建调用

因为普通类实例可以通过运算符重载来响应调用操作，所以普通类实例也可以充当元类的角色：

```python
# metaclass4.py
# A normal class instance can serve as a metaclass too
class MetaObj:
    def __call__(self, classname, supers, classdict):
        print('In MetaObj.call: ', classname, supers, classdict, sep='\n...')
        Class = self.__New__(classname, supers, classdict)
        self.__Init__(Class, classname, supers, classdict)
        return Class
    def __New__(self, classname, supers, classdict):
        print('In MetaObj.new: ', classname, supers, classdict, sep='\n...')
        return type(classname, supers, classdict)
    def __Init__(self, Class, classname, supers, classdict):
        print('In MetaObj.init:', classname, supers, classdict, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))

class Eggs:
    pass

print('making class')
class Spam(Eggs, metaclass=MetaObj()): # MetaObj is normal class instance
    data = 1 # Called at end of statement
    def meth(self, arg):
        return self.data + arg

print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

当运行时，这三个方法通过普通实例的`__call__`方法被调度：

```python
c:\code> py −3 metaclass4.py
making class
In MetaObj.call:
...Spam
...(<class '__main__.Eggs'>,)
...{'data': 1, 'meth': <function Spam.meth at 0x029492F0>, '__module__': '__main__'}
In MetaObj.new:
...Spam
...(<class '__main__.Eggs'>,)
...{'data': 1, 'meth': <function Spam.meth at 0x029492F0>, '__module__': '__main__'}
In MetaObj.init:
...Spam
...(<class '__main__.Eggs'>,)
...{'data': 1, 'meth': <function Spam.meth at 0x029492F0>, '__module__': '__main__'}
...init class object: ['__module__', '__doc__', 'data', '__qualname__', 'meth']
making instance
data: 1 3
```

实际上，我们可以使用普通超类继承来获得调用拦截器：

```python
# metaclass4-super.py
# Instances inherit from classes and their supers normally
class SuperMetaObj:
    def __call__(self, classname, supers, classdict):
        print('In SuperMetaObj.call: ', classname, supers, classdict, sep='\n...')
        Class = self.__New__(classname, supers, classdict)
        self.__Init__(Class, classname, supers, classdict)
        return Class
    
class SubMetaObj(SuperMetaObj):
    def __New__(self, classname, supers, classdict):
        print('In SubMetaObj.new: ', classname, supers, classdict, sep='\n...')
        return type(classname, supers, classdict)
    def __Init__(self, Class, classname, supers, classdict):
        print('In SubMetaObj.init:', classname, supers, classdict, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))
        
class Spam(Eggs, metaclass=SubMetaObj()): # Invoke Sub instance via Super.__call__
    ...rest of file unchanged...
```

```python
c:\code> py −3 metaclass4-super.py
making class
In SuperMetaObj.call:
...as before...
In SubMetaObj.new:
...as before...
In SubMetaObj.init:
...as before...
making instance
data: 1 3
```



##### 用元类重载类创建调用

The redefinitions of both `__new__` and `__call__` must be careful to call back to their defaults in type if they mean to make a class in the end, and `__call__` must invoke type to kick off the other two here:

```python
# metaclass5.py
# Classes can catch calls too (but built-ins look in metas, not supers!)
class SuperMeta(type):
    def __call__(meta, classname, supers, classdict):
        print('In SuperMeta.call: ', classname, supers, classdict, sep='\n...')
        return type.__call__(meta, classname, supers, classdict)
    def __init__(Class, classname, supers, classdict):
        print('In SuperMeta init:', classname, supers, classdict, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))
        print('making metaclass')
        
class SubMeta(type, metaclass=SuperMeta):
    def __new__(meta, classname, supers, classdict):
        print('In SubMeta.new: ', classname, supers, classdict, sep='\n...')
        return type.__new__(meta, classname, supers, classdict)
    def __init__(Class, classname, supers, classdict):
        print('In SubMeta init:', classname, supers, classdict, sep='\n...')
        print('...init class object:', list(Class.__dict__.keys()))

class Eggs:
     pass
    
print('making class')
class Spam(Eggs, metaclass=SubMeta): # Invoke SubMeta, via SuperMeta.__call__
    data = 1
    def meth(self, arg):
        return self.data + arg
    
print('making instance')
X = Spam()
print('data:', X.data, X.meth(2))
```

当这段代码运行的时候，所有3个重新定义的方法都依次运行。这基本上就是`type`对象默认做的事情：

```python
c:\code> py −3 metaclass5.py
making metaclass
In SuperMeta init:
...SubMeta
...(<class 'type'>,)
...{'__init__': <function SubMeta.__init__ at 0x028F92F0>, ...}
...init class object: ['__doc__', '__module__', '__new__', '__init__, ...]
making class
In SuperMeta.call:
...Spam
...(<class '__main__.Eggs'>,)
...{'data': 1, 'meth': <function Spam.meth at 0x028F9378>, '__module__': '__main__'}
In SubMeta.new:
...Spam
...(<class '__main__.Eggs'>,)
...{'data': 1, 'meth': <function Spam.meth at 0x028F9378>, '__module__': '__main__'}
In SubMeta init:
...Spam
...(<class '__main__.Eggs'>,)
...{'data': 1, 'meth': <function Spam.meth at 0x028F9378>, '__module__': '__main__'}
...init class object: ['__qualname__', '__module__', '__doc__', 'data', 'meth']
making instance
data: 1 3
```

This example is complicated by the fact that it overrides a method invoked by a builtin operation—in this case, the call run automatically to create a class. Metaclasses are used to create class objects, but only generate instances of themselves when called in a metaclass role. Because of this, name lookup with metaclasses may be somewhat different than what we are accustomed to. The `__call__` method, for example, is looked up by built-ins in the class (a.k.a. type) of an object; for metaclasses, this means the metaclass of a metaclass!

As we’ll see ahead, metaclasses also inherit names from other metaclasses normally, but as for normal classes, this seems to apply to explicit name fetches only, not to the implicit lookup of names for built-in operations such as calls. The latter appears to look in the metaclass’s class, available in its `__class__` link—which is either the default type or a metaclass. This is the same built-ins routing issue we’ve seen so often in this book for normal class instances. The metaclass in `SubMeta` is required to set this link, though this also kicks off a metaclass construction step for the metaclass itself.

Trace the invocations in the output. `SuperMeta`’s `__call__` method is not run for the call to SuperMeta when making `SubMeta` (this goes to type instead), but is run for the `SubMeta` call when making `Spam`. Inheriting normally from `SuperMeta` does not suffice to catch `SubMeta` calls, and for reasons we’ll see later is actually the wrong thing to do for operator overloading methods: `SuperMeta`’s `__call__` is then acquired by `Spam`, causing `Spam` instance creation calls to fail before any instance is ever created. Subtle but true!

Here’s an illustration of the issue in simpler terms—a normal superclass is skipped for built-ins, but not for explicit fetches and calls, the latter relying on normal attribute name inheritance:

```python
class SuperMeta(type):
    def __call__(meta, classname, supers, classdict): # By name, not built-in
        print('In SuperMeta.call:', classname)
        return type.__call__(meta, classname, supers, classdict)

class SubMeta(SuperMeta): # Created by type default
    def __init__(Class, classname, supers, classdict): # Overrides type.__init__
        print('In SubMeta init:', classname)

print(SubMeta.__class__)
print([n.__name__ for n in SubMeta.__mro__])
print()
print(SubMeta.__call__) # Not a data descriptor if found by name
print()
SubMeta.__call__(SubMeta, 'xxx', (), {}) # Explicit calls work: class inheritance
print()
SubMeta('yyy', (), {}) # But implicit built-in calls do not: type
```

```
c:\code> py −3 metaclass5b.py
<class 'type'>
['SubMeta', 'SuperMeta', 'type', 'object']
<function SuperMeta.__call__ at 0x029B9158>
In SuperMeta.call: xxx
In SubMeta init: xxx
In SubMeta init: yyy
```

Of course, this specific example is a special case: catching a built-in run on a metaclass, a likely rare usage related to `__call__` here. But it underscores a core asymmetry and apparent inconsistency: *normal attribute inheritance is not fully used for built-in dispatch* —for both instances and classes.

To truly understand this example’s subtleties, though, we need to get more formal about what metaclasses mean for Python name resolution in general.



### 40.5 继承和实例

由于元类以类似于继承超类的方式来制定，因此它们乍看上去有点容易令人混淆。一些关键点有助于概括和澄清这一模型：

- **继承自`type`类的元类** 
  - 尽管它们有一种特殊的角色元类，但元类是用`class`语句编写的，并且遵从Python中有用的OOP模型。例如，就像`type`的子类一样，它们可以重新定义`type`对象的方法，需要的时候重载或定制它们。元类通常重新定义`type`类的`__new__`和`__init__`，以定制类创建和初始化，但是，如果它们希望直接捕获类末尾的创建调用的话，它们也可以重新定义`__call__`。尽管元类不常见，它们甚至是返回任意对象而不是`type`子类的简单函数。
- **元类声明被子类继承** 
  - 在用户定义的类中，`metaclass=M`声明由该类的子类继承，因此，对于在超类链中继承了这一声明的每个类的构建，该元类都将运行。
- **元类属性没有被类实例继承** 
  - 元类声明指定了一个实例关系，它和继承不同。由于类是元类的实例，所以元类中定义的行为应用于类，而不是类随后的实例。实例从它们的类和超类获取行为，但是，不是从任何元类获取行为。从技术上讲，实例属性查找通常只是搜索实例及其所有类的`__dict__`字典；元类不包含在实例查找中。
- **元类属性被类获取 (Metaclass attributes are acquired by classes) ** 
  - By contrast, classes do acquire methods of their metaclasses by virtue of the instancerelationship. This is a source of class behavior that processes classes themselves.Technically, classes acquire metaclass attributes through the class’s `__class__` link just as normal instances acquire names from their class, but inheritance via `__dict__` search is attempted first: when the same name is available to a class in both a metaclass and a superclass, the superclass (inheritance) version is used instead of that on a metaclass (instance). The class’s `__class__`, however, is not followed for its own instances: metaclass attributes are made available to their instance classes, but not to instances of those instance classes (and see the earlier reference to Dr. Seuss...).

为了说明，考虑如下的例子：

```python
# File metainstance.py
class MetaOne(type):
    def __new__(meta, classname, supers, classdict): # Redefine type method
        print('In MetaOne.new:', classname)
        return type.__new__(meta, classname, supers, classdict)
    def toast(self):
        return 'toast'

class Super(metaclass=MetaOne): # Metaclass inherited by subs too
    def spam(self): # MetaOne run twice for two classes
        return 'spam'
    
class Sub(Super): # Superclass: inheritance versus instance
    def eggs(self): # Classes inherit from superclasses
        return 'eggs' # But not from metaclasses
```

当这段代码作为脚本或模块运行时，元类处理客户类`Super`和`Sub`的创建 (construction) ；并且实例只继承类属性，但没有继承元类的属性：

```python
>>> from metainstance import * # Runs class statements: metaclass run twice
In MetaOne.new: Super
In MetaOne.new: Sub
>>> X = Sub() # Normal instance of user-defined class
>>> X.eggs() # Inherited from Sub
'eggs'
>>> X.spam() # Inherited from Super
'spam'
>>> X.toast() # Not inherited from metaclass
AttributeError: 'Sub' object has no attribute 'toast'
```

相比之下，类`Super`和`Sub`都从它们的超类继承了变量名，并从它们的元类获取到了变量名：

```python
>>> Sub.eggs(X) # Own method
'eggs'
>>> Sub.spam(X) # Inherited from Super
'spam'
>>> Sub.toast() # Acquired from metaclass
'toast'
>>> Sub.toast(X) # Not a normal class method
TypeError: toast() takes 1 positional argument but 2 were given
```

因为变量名解析到一个元类方法，而不是一个类方法，所以，在上例中，当我们传入一个实例时调用失败了。从元类取得的方法是绑定到主体类对象的；然而，如果通过类来取得普通类的方法，则取得的方法是未绑定的，如果通过实例来取得普通类的方法，则是绑定的：

```python
>>> Sub.toast
<bound method MetaOne.toast of <class 'metainstance.Sub'>>
>>> Sub.spam
<function Super.spam at 0x0298A2F0>
>>> X.spam
<bound method Sub.spam of <metainstance.Sub object at 0x02987438>>
```



#### 元类VS超类
In even simpler terms, watch what happens in the following: as an instance of the A metaclass type, class B acquires A’s attribute, but this attribute is not made available for inheritance by B’s own instances—the acquisition of names by metaclass instances is distinct from the normal inheritance used for class instances:

```python
>>> class A(type): attr = 1
>>> class B(metaclass=A): pass # B is meta instance and acquires meta attr
>>> I = B() # I inherits from class but not meta!
>>> B.attr
1
>>> I.attr
AttributeError: 'B' object has no attribute 'attr'
>>> 'attr' in B.__dict__, 'attr' in A.__dict__
(False, True)
```

By contrast, if A morphs from metaclass to superclass, then names inherited from an A superclass become available to later instances of B, and are located by searching namespace dictionaries in classes in the tree—that is, by checking the `__dict__` of objects in the method resolution order (MRO), much like the mapattrs example we coded back in Chapter 32:

```python
>>> class A: attr = 1
>>> class B(A): pass # I inherits from class and supers
>>> I = B()
>>> B.attr
1
>>> I.attr
1
>>> 'attr' in B.__dict__, 'attr' in A.__dict__
(False, True)
```

This is why metaclasses often do their work by manipulating a new class’s namespace dictionary, if they wish to influence the behavior of later instance objects—instances will see names in a class, but not its metaclass. Watch what happens, though, if the same name is available in both attribute sources—the inheritance name is used instead of instance acquisition:

```python
>>> class M(type): attr = 1
>>> class A: attr = 2
>>> class B(A, metaclass=M): pass # Supers have precedence over metas
>>> I = B()
>>> B.attr, I.attr
(2, 2)
>>> 'attr' in B.__dict__, 'attr' in A.__dict__, 'attr' in M.__dict__
(False, True, True)
```

This is true regardless of the relative height of the inheritance and instance sources— Python checks the `__dict__` of each class on the MRO (inheritance), before falling back on metaclass acquisition (instance):

```python
>>> class M(type): attr = 1
>>> class A: attr = 2
>>> class B(A): pass
>>> class C(B, metaclass=M): pass # Super two levels above meta: still wins
>>> I = C()
>>> I.attr, C.attr
(2, 2)
>>> [x.__name__ for x in C.__mro__] # See Chapter 32 for all things MRO
['C', 'B', 'A', 'object']
```

In fact, classes acquire metaclass attributes through their `__class__` link, in the same way that normal instances inherit from classes through their `__class__`, which makes sense, given that classes are also instances of metaclasses. The chief distinction is that instance inheritance does not follow a class’s `__class__`, but instead restricts its scope to the `__dict__` of each class in a tree per the MRO—following `__bases__` at each class only, and using only the instance’s `__class__` link once:

```python
>>> I.__class__ # Followed by inheritance: instance's class
<class '__main__.C'>
>>> C.__bases__ # Followed by inheritance: class's supers
(<class '__main__.B'>,)
>>> C.__class__ # Followed by instance acquisition: metaclass
<class '__main__.M'>
>>> C.__class__.attr # Another way to get to metaclass attributes
1
```




#### 关于继承的来龙去脉

第五版，暂略



### 40.6 元类方法

Just as important as the inheritance of names, methods in metaclasses process their instance classes—not the normal instance objects we’ve known as “self,” but classes themselves. This makes them similar in spirit and form to the class methods we studied in Chapter 32, though they again are available in the metaclasses instance realm only, not to normal instance inheritance. The failure at the end of the following, for example, stems from the explicit name inheritance rules of the prior section:

```python
>>> class A(type):
        def x(cls): print('ax', cls) # A metaclass (instances=classes)
        def y(cls): print('ay', cls) # y is overridden by instance B
>>> class B(metaclass=A):
        def y(self): print('by', self) # A normal class (normal instances)
        def z(self): print('bz', self) # Namespace dict holds y and z
    
>>> B.x # x acquired from metaclass
<bound method A.x of <class '__main__.B'>>
>>> B.y # y and z defined in class itself
<function B.y at 0x0295F1E0>
>>> B.z
<function B.z at 0x0295F378>
>>> B.x() # Metaclass method call: gets cls
ax <class '__main__.B'>
>>> I = B() # Instance method calls: get inst
>>> I.y()
by <__main__.B object at 0x02963BE0>
>>> I.z()
bz <__main__.B object at 0x02963BE0>
>>> I.x() # Instance doesn't see meta names
AttributeError: 'B' object has no attribute 'x'
```



#### Metaclass Methods Versus Class Methods

Though they differ in inheritance visibility, much like class methods, metaclass methods are designed to manage class-level data. In fact, their roles can overlap—much as metaclasses do in general with class decorators—but metaclass methods are not accessible except through the class, and do not require an explicit classmethod class-level data declaration in order to be bound with the class. In other words, metaclass methods can be thought of as implicit class methods, with limited visibility:

```python
>>> class A(type):
        def a(cls): # Metaclass method: gets class
            cls.x = cls.y + cls.z
            
>>> class B(metaclass=A):
        y, z = 11, 22
        @classmethod # Class method: gets class
        def b(cls):
            return cls.x
        
>>> B.a() # Call metaclass method; visible to class only
>>> B.x # Creates class data on B, accessible to normal instances
33
>>> I = B()
>>> I.x, I.y, I.z
(33, 11, 22)
>>> I.b() # Class method: sends class, not instance; visible to instance
33
>>> I.a() # Metaclass methods: accessible through class only
AttributeError: 'B' object has no attribute 'a'
```



#### Operator Overloading in Metaclass Methods

Just like normal classes, metaclasses may also employ operator overloading to make built-in operations applicable to their instance classes. The `__getitem__` indexing method in the following metaclass, for example, is a metaclass method designed to process classes themselves—the classes that are instances of the metaclass, not those classes’ own later instances. In fact, per the inheritance algorithms sketched earlier, normal class instances don’t inherit names acquired via the metaclass instance relationship at all, though they can access names present on their own classes:

```python
>>> class A(type):
        def __getitem__(cls, i):        # Meta method for processing classes:
            return cls.data[i]          # Built-ins skip class, use meta
                                        # Explicit names search class + meta
>>> class B(metaclass=A):               # Data descriptors in meta used first
        data = 'spam'
        
>>> B[0]                  # Metaclass instance names: visible to class only
's'
>>> B.__getitem__
<bound method A.__getitem__ of <class '__main__.B'>>
>>> I = B()
>>> I.data, B.data        # Normal inheritance names: visible to instance and class
('spam', 'spam')
>>> I[0]
TypeError: 'B' object does not support indexing
```

It’s possible to define a `__getattr__` on a metaclass too, but it can be used to process its instance classes only, not their normal instances—as usual, it’s not even acquired by a class’s instances:

```python
>>> class A(type):
        def __getattr__(cls, name): # Acquired by class B getitem
            return getattr(cls.data, name) # But not run same by built-ins
>>> class B(metaclass=A):
        data = 'spam'
>>> B.upper()
'SPAM'
>>> B.upper
<built-in method upper of str object at 0x029E7420>
>>> B.__getattr__
<bound method A.__getattr__ of <class '__main__.B'>>
>>> I = B()
>>> I.upper
AttributeError: 'B' object has no attribute 'upper'
>>> I.__getattr__
AttributeError: 'B' object has no attribute '__getattr__'
```

Moving the `__getattr__` to a metaclass doesn’t help with its built-in interception shortcomings, though. In the following continuation, explicit attributes are routed to the metaclass’s `__getattr__`, but built-ins are not, despite that fact the indexing is routed to a metaclass’s `__getitem__` in the first example of the section—strongly suggesting that new-style `__getattr__` is a special case of a special case, and further recommending code simplicity that avoids dependence on such boundary cases:

```python
>>> B.data = [1, 2, 3]
>>> B.append(4) # Explicit normal names routed to meta's getattr
>>> B.data
[1, 2, 3, 4]
>>> B.__getitem__(0) # Explicit special names routed to meta's gettarr
1
>>> B[0] # But built-ins skip meta's gettatr too?!
TypeError: 'A' object does not support indexing
```

As you can probably tell, metaclasses are interesting to explore, but it’s easy to lose track of their big picture. In the interest of space, we’ll omit additional fine points here. For the purposes of this chapter, it’s more important to show why you’d care to use such a tool in the first place. Let’s move on to some larger examples to sample the roles of metaclasses in action. As we’ll find, like so many tools in Python, metaclasses are first and foremost about easing maintenance work by eliminating redundancy.



### 40.7 实例：向类添加方法

#### 手动扩展

```python
# Extend manually - adding new methods to classes
class Client1:
    def __init__(self, value):
        self.value = value
    def spam(self):
        return self.value * 2

class Client2:
    value = 'ni?'
    def eggsfunc(obj):
        return obj.value * 4
    def hamfunc(obj, value):
        return value + 'ham'

Client1.eggs = eggsfunc
Client1.ham = hamfunc

Client2.eggs = eggsfunc
Client2.ham = hamfunc

X = Client1('Ni!')
print(X.spam())
print(X.eggs())
print(X.ham('bacon'))

Y = Client2()
print(Y.eggs())
print(Y.ham('bacon'))
```

```
c:\code> py −3 extend-manual.py
Ni!Ni!
Ni!Ni!Ni!Ni!
baconham
ni?ni?ni?ni?
baconham
```



#### 基于元类的扩展

如果我们在元类中编写扩展，那么声明了元类的每个类都将统一且正确地扩展，并自动地接收未来做出的任何修改。如下的代码展示了这一点：

```python
# Extend with a metaclass - supports future changes better
def eggsfunc(obj):
    return obj.value * 4

def hamfunc(obj, value):
    return value + 'ham'

class Extender(type):
    def __new__(meta, classname, supers, classdict):
        classdict['eggs'] = eggsfunc
        classdict['ham'] = hamfunc
        return type.__new__(meta, classname, supers, classdict)

class Client1(metaclass=Extender):
    def __init__(self, value):
    self.value = value
    def spam(self):
    return self.value * 2

class Client2(metaclass=Extender):
    value = 'ni?'

X = Client1('Ni!')
print(X.spam())
print(X.eggs())
print(X.ham('bacon'))

Y = Client2()
print(Y.eggs())
print(Y.ham('bacon'))
```

```
c:\code> py −3 extend-meta.py
Ni!Ni!
Ni!Ni!Ni!Ni!
baconham
ni?ni?ni?ni?
baconham
```



#### 元类VS类装饰器：第2回合

请记住，上一章的类装饰器常常和本章的元类在功能上有重合。这源自于如下的事实：

- 在class语句的末尾，类装饰器把类名重新绑定到一个函数的结果。
- 元类通过在一条class语句的末尾把类对象创建过程路由到一个对象来工作。

##### 基于装饰器的扩展

前面小节的元类示例，像创建的一个类添加方法，也可以用一个类装饰器来编写。在这种模式下，装饰器大致与元类的`__init__`方法对应，因为在调用装饰器的时候，类对象已经创建了。如下的输出与前面的元类代码的输出相同：

```python
# Extend with a decorator: same as providing __init__ in a metaclass
def eggsfunc(obj):
    return obj.value * 4
def hamfunc(obj, value):
    return value + 'ham'

def Extender(aClass):
    aClass.eggs = eggsfunc # Manages class, not instance
    aClass.ham = hamfunc # Equiv to metaclass __init__
    return aClass

@Extender
class Client1: # Client1 = Extender(Client1)
    def __init__(self, value): # Rebound at end of class stmt
        self.value = value
    def spam(self):
        return self.value * 2

@Extender
class Client2:
    value = 'ni?'

X = Client1('Ni!') # X is a Client1 instance
print(X.spam())
print(X.eggs())
print(X.ham('bacon'))

Y = Client2()
print(Y.eggs())
print(Y.ham('bacon'))
```



##### 管理实例而不是类

正如我们已经见到的，类装饰器常常可以和元类一样充当类管理角色。元类往往和装饰器一样充当实例管理的角色，但是管理实例需要一些额外工作。

例如，前一章中的类装饰器示例，无论何时，获取一个类实例的任意常规命名的属性的时候，它用来打印一条跟踪消息：

```python
# Class decorator to trace external instance attribute fetches
def Tracer(aClass): # On @ decorator
    class Wrapper:
        def __init__(self, *args, **kargs): # On instance creation
            self.wrapped = aClass(*args, **kargs) # Use enclosing scope name
        def __getattr__(self, attrname):
            print('Trace:', attrname) # Catches all but .wrapped
            return getattr(self.wrapped, attrname) # Delegate to wrapped object
    return Wrapper

@Tracer
class Person: # Person = Tracer(Person)
    def __init__(self, name, hours, rate): # Wrapper remembers Person
        self.name = name
        self.hours = hours
        self.rate = rate # In-method fetch not traced
    def pay(self):
        return self.hours * self.rate
    
bob = Person('Bob', 40, 50) # bob is really a Wrapper
print(bob.name) # Wrapper embeds a Person
print(bob.pay()) # Triggers __getattr__
```

```
c:\code> py −3 manage-inst-deco.py
Trace: name
Bob
Trace: pay
2000
```

尽管用一个元类也可能实现同样的效果，但它似乎概念上不太直接明了。 **元类明确地设计来管理类对象创建，并且它们有一个为此目的而设计的接口。** 如下的元类和前面的装饰器具有同样的效果和输出：

```python
# Manage instances like the prior example, but with a metaclass
def Tracer(classname, supers, classdict): # On class creation call
    aClass = type(classname, supers, classdict) # Make client class
    class Wrapper:
        def __init__(self, *args, **kargs): # On instance creation
            self.wrapped = aClass(*args, **kargs)
        def __getattr__(self, attrname):
            print('Trace:', attrname) # Catches all but .wrapped
            return getattr(self.wrapped, attrname) # Delegate to wrapped object
    return Wrapper

class Person(metaclass=Tracer): # Make Person with Tracer
    def __init__(self, name, hours, rate): # Wrapper remembers Person
        self.name = name
        self.hours = hours
        self.rate = rate # In-method fetch not traced
    def pay(self):
        return self.hours * self.rate

bob = Person('Bob', 40, 50) # bob is really a Wrapper
print(bob.name) # Wrapper embeds a Person
print(bob.pay()) # Triggers __getattr__
```

**通常，元类更适合于类对象管理，因为它们就设计为如此。**



##### 元类和类装饰器等价吗？ ()

The preceding section illustrated that metaclasses incur an extra step to create the class when used in instance management roles, and hence can’t quite subsume decorators in all use cases. But what about the inverse—are decorators a replacement for metaclasses?

Just in case this chapter has not yet managed to make your head explode, consider the following metaclass coding alternative too—a class decorator that returns a metaclass instance:

```python
# A decorator can call a metaclass, though not vice versa without type()
>>> class Metaclass(type):
        def __new__(meta, clsname, supers, attrdict):
            print('In M.__new__:')
            print([clsname, supers, list(attrdict.keys())])
            return type.__new__(meta, clsname, supers, attrdict)
>>> def decorator(cls):
        return Metaclass(cls.__name__, cls.__bases__, dict(cls.__dict__))

>>> class A:
        x = 1

>>> @decorator
    class B(A):
        y = 2
        def m(self): return self.x + self.y

In M.__new__:
['B', (<class '__main__.A'>,), ['__qualname__', '__doc__', 'm', 'y', '__module__']]
>>> B.x, B.y
(1, 2)
>>> I = B()
>>> I.x, I.y, I.m()
(1, 2, 3)
```

This nearly proves the equivalence of the two tools, but really just in terms of dispatch at class construction time. Again, decorators essentially serve the same role as metaclass `__init__` methods. Because this decorator returns a metaclass instance, metaclasses— or at least their type superclass—are still assumed here. Moreover, this winds up triggering an additional metaclass call after the class is created, and isn’t an ideal scheme in real code—you might as well move this metaclass to the first creation step:

```python
>>> class B(A, metaclass=Metaclass): ... # Same effect, but makes just one class
```

Still, there is some tool redundancy here, and decorator and metaclass roles often overlap in practice. And although decorators don’t directly support the notion of class-level methods in metaclasses discussed earlier, methods and state in proxy objects created by decorators can achieve similar effects, though for space we’ll leave this last observation in the suggested explorations column.

The inverse may not seem applicable—a metaclass can’t generally defer to a nonmetaclass decorator, because the class doesn’t yet exist until the metaclass call completes —although a metaclass can take the form of a simple callable that invokes type to create the class directly and passes it on to the decorator. In other words, the crucial hook in the model is the type call issued for class construction. Given that, metaclasses and class decorators are often functionally equivalent, with varying dispatch protocol models:

```python
>>> def Metaclass(clsname, supers, attrdict):
        return decorator(type(clsname, supers, attrdict))
>>> def decorator(cls): ...
>>> class B(A, metaclass=Metaclass): ... # Metas can call decos and vice versa
```

In fact, metaclasses need not necessarily return a type instance either—any object compatible with the class coder’s expectations will do—and this further blurs the decorator/metaclass distinction:

```python
>>> def func(name, supers, attrs):
        return 'spam'

>>> class C(metaclass=func): # A class whose metaclass makes it a string!
        attr = 'huh?'

>>> C, C.upper()
('spam', 'SPAM')
>>> def func(cls):
        return 'spam'

>>> @func
    class C: # A class whose decorator makes it a string!
        attr = 'huh?'

>>> C, C.upper()
('spam', 'SPAM')
```

Odd metaclass and decorator tricks like these aside, timing often determines roles in practice, as stated earlier:

- Because decorators run after a class is created, they incur an extra runtime step in class creation roles.
- Because metaclasses must create classes, they incur an extra coding step in instance management roles. 

In other words, neither completely subsumes the other. Strictly speaking, metaclasses might be a functional superset, as they can call decorators during class creation; but metaclasses can also be substantially heavier to understand and code, and many roles intersect completely. In practice, the need to take over class creation entirely is probably much less important than tapping into the process in general.

Rather than follow this rabbit hole further, though, let’s move on to explore metaclass roles that may be a bit more typical and practical. The next section concludes this chapter with one more common use case—applying operations to a class’s methods automatically at class creation time.





### 40.8 实例：对方法应用装饰器

可以将装饰器和元类结合起来使用，作为互补的工具。在本小节中，我们将展示一个示例，它就是这样的组合——对一个类的所有方法应用一个函数装饰器。

#### 用装饰器手动跟踪

```python
# File decotools.py: assorted decorator tools
import time
def tracer(func): # Use function, not class with __call__
    calls = 0 # Else self is decorator instance only
    def onCall(*args, **kwargs):
        nonlocal calls
        calls += 1
        print('call %s to %s' % (calls, func.__name__))
        return func(*args, **kwargs)
    return onCall

def timer(label='', trace=True): # On decorator args: retain args
    def onDecorator(func): # On @: retain decorated func
        def onCall(*args, **kargs): # On calls: call original
            start = time.clock() # State is scopes + func attr
            result = func(*args, **kargs)
            elapsed = time.clock() - start
            onCall.alltime += elapsed
            if trace:
                format = '%s%s: %.5f, %.5f'
                values = (label, func.__name__, elapsed, onCall.alltime)
                print(format % values)
            return result
        onCall.alltime = 0
        return onCall
    return onDecorator
```

要手动使用这些装饰器，我们直接从模块导入它们，并且在想要跟踪或计时的每个方法前编写`@`装饰语法：

```python
from decotools import tracer

class Person:
    @tracer
    def __init__(self, name, pay):
        self.name = name
        self.pay = pay
    @tracer
    def giveRaise(self, percent):        # giveRaise = tracer(giverRaise)
        self.pay *= (1.0 + percent)      # onCall remembers giveRaise
    @tracer
    def lastName(self):                  # lastName = tracer(lastName)
        return self.name.split()[-1]

bob = Person('Bob Smith', 50000)
sue = Person('Sue Jones', 100000)
print(bob.name, sue.name)
sue.giveRaise(.10)                       # Runs onCall(sue, .10)
print('%.2f' % sue.pay)
print(bob.lastName(), sue.lastName())    # Runs onCall(bob), remembers lastName
```

```
c:\code> py −3 decoall-manual.py
call 1 to __init__
call 2 to __init__
Bob Smith Sue Jones
call 1 to giveRaise
110000.00
call 1 to lastName
call 2 to lastName
Smith Jones
```



#### 用元类和装饰器跟踪

前一小节的手动装饰方法是有效的，但是它需要我们在想要跟踪的每个方法前面添加装饰语法，并且在不再想要跟踪的使用后删除该语法。如果想要跟踪一个类的每个方法，在较大的程序中，这会变得很繁琐。

有了元类，我们可以对一个类的所有方法自动地应用跟踪装饰器——因为它们在构建一个类的时候运行，它们是把装饰包装器添加到一个类方法中的自然地方。通过扫描类的属性字典并测试函数对象，我们可以通过装饰器自动运行方法，并且把最初的名称重新绑定到结果。其效果与装饰器的自动方法名重新绑定是相同的，但是，我们可以更全面地应用它：

```python
# Metaclass that adds tracing decorator to every method of a client class
from types import FunctionType
from decotools import tracer

class MetaTrace(type):
    def __new__(meta, classname, supers, classdict):
        for attr, attrval in classdict.items():
            if type(attrval) is FunctionType: # Method?
                classdict[attr] = tracer(attrval) # Decorate it
        return type.__new__(meta, classname, supers, classdict) # Make class
    
class Person(metaclass=MetaTrace):
    def __init__(self, name, pay):
        self.name = name
        self.pay = pay
    def giveRaise(self, percent):
        self.pay *= (1.0 + percent)
    def lastName(self):
        return self.name.split()[-1]

bob = Person('Bob Smith', 50000)
sue = Person('Sue Jones', 100000)
print(bob.name, sue.name)
sue.giveRaise(.10)
print('%.2f' % sue.pay)
print(bob.lastName(), sue.lastName())
```

当这段代码运行的时候，结果与前面是相同的——方法的调用将首先指向跟踪装饰器以跟踪，然后传递到最初的方法：

```
c:\code> py −3 decoall-meta.py
call 1 to __init__
call 2 to __init__
Bob Smith Sue Jones
call 1 to giveRaise
110000.00
call 1 to lastName
call 2 to lastName
Smith Jones
```

我们这里看到的就是装饰器和元类组合工作的结果——在类创建的时候，元类自动把函数装饰器应用于每个方法，并且函数装饰器自动拦截方法调用，以便在此输出中打印出跟踪消息。这一组合能够有效，得益于两种工具的通用性。

#### 把任何装饰器用于方法

```python
# Metaclass factory: apply any decorator to all methods of a class
from types import FunctionType
from decotools import tracer, timer

def decorateAll(decorator):
    class MetaDecorate(type):
        def __new__(meta, classname, supers, classdict):
            for attr, attrval in classdict.items():
                if type(attrval) is FunctionType:
                    classdict[attr] = decorator(attrval)
            return type.__new__(meta, classname, supers, classdict)
    return MetaDecorate   # 返回元类

class Person(metaclass=decorateAll(tracer)): # Apply a decorator to all
    def __init__(self, name, pay):
        self.name = name
        self.pay = pay
    def giveRaise(self, percent):
        self.pay *= (1.0 + percent)
    def lastName(self):
        return self.name.split()[-1]

bob = Person('Bob Smith', 50000)
sue = Person('Sue Jones', 100000)
print(bob.name, sue.name)
sue.giveRaise(.10)
print('%.2f' % sue.pay)
print(bob.lastName(), sue.lastName())
```

当这段代码运行的时候，输出再次与前面的示例相同——最终我们仍然在一个客户类中用跟踪器函数装饰器装饰了每个方法，但是，我们以一种更为通用的方式做到了这点：

```
c:\code> py −3 decoall-meta-any.py
call 1 to __init__
call 2 to __init__
Bob Smith Sue Jones
call 1 to giveRaise
110000.00
call 1 to lastName
call 2 to lastName
Smith Jones
```

现在，要对方法应用一种不同的装饰器，我们只要在类标题行替换装饰器名称。例如，要使用前面介绍的计时器函数装饰器，定义类的时候，我们可以使用如下示例标题行的最后两行中的任何一个——第一个接收了定时器的默认参数，第二个指定了标签文本：

```python
class Person(metaclass=decorateAll(tracer)):                # Apply tracer
class Person(metaclass=decorateAll(timer())):               # Apply timer, defaults
class Person(metaclass=decorateAll(timer(label='**'))):     # Decorator arguments
```



#### 元类VS类装饰器：第3回合

如下的版本，用一个类装饰器替换了前面的示例中的元类。它定义并使用一个类装饰器，该装饰器把一个函数装饰器应用于一个类的所有方法。

```python
# Class decorator factory: apply any decorator to all methods of a class
from types import FunctionType
from decotools import tracer, timer

def decorateAll(decorator):
    def DecoDecorate(aClass):
        for attr, attrval in aClass.__dict__.items():
            if type(attrval) is FunctionType:
                setattr(aClass, attr, decorator(attrval)) # Not __dict__
        return aClass
    return DecoDecorate

@decorateAll(tracer)                        # Use a class decorator
class Person:                               # Applies func decorator to methods
    def __init__(self, name, pay):          # Person = decorateAll(..)(Person)
        self.name = name                    # Person = DecoDecorate(Person)
        self.pay = pay
    def giveRaise(self, percent):
        self.pay *= (1.0 + percent)
    def lastName(self):
        return self.name.split()[-1]

bob = Person('Bob Smith', 50000)
sue = Person('Sue Jones', 100000)
print(bob.name, sue.name)
sue.giveRaise(.10)
print('%.2f' % sue.pay)
print(bob.lastName(), sue.lastName())
```

当这段代码运行的时候，类装饰器把跟踪器函数装饰器应用于每个方法，并且在调用时产生一条跟踪消息（输出和本示例前面的元类版本相同）：

```
c:\code> py −3 decoall-deco-any.py
call 1 to __init__
call 2 to __init__
Bob Smith Sue Jones
call 1 to giveRaise
110000.00
call 1 to lastName
call 2 to lastName
Smith Jones
```

同样地，使用类装饰器也可以像下面这样处理参数：

```python
@decorateAll(tracer) # Decorate all with tracer
@decorateAll(timer()) # Decorate all with timer, defaults
@decorateAll(timer(label='@@')) # Same but pass a decorator argument
```

正如你所看到的，元类和类装饰器不仅常常可以交换，而且通常是互补的。它们都对于定制和管理类和实例对象，提供了高级但强大的方法，因为这二者最终都允许我们在类创建过程中插入代码。尽管某些高级应用可能用一种方式或另一种方式编码更好，但在很多情况下，我们选择或组合这两种工具的方法，很大程度上取决于你。









---



