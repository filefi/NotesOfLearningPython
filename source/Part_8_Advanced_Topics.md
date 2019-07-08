第8部分 高级话题

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

属性获取拦截表现为两种形式，可用两个不同的方法来编写：

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

这样的代码结构可以用来实现我们在第31章介绍的委托设计模式。由于所有的属性通常
都指向我们的拦截方法，所以我们可以验证它们并将其传递到嵌入的、管理的对象中。特性和描述符没有这样的类似功能，做不到对每个可能的包装对象中每个可能的属性编
写访问器。

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







### 38.5 实例：属性验证

















---



## 第39章 装饰器







---



## 第40章 元类







---



## 第41章 所有美好的事物

