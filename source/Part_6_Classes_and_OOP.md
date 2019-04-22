# 第6部分 类和OOP

## 第26章 OOP：The Big Picture

在Python中，使用`class`语句创建类。

在Python中，OOP完全是可选的！

### 26.1 为何使用类

类是Python的程序组成单元，就像函数和模块一样。相比函数和模块，类有三个重要的特点，使其在建立新对象时更为有用：

- **多重实例**：类基本上就是产生对象的工厂。每次调用一个类，就会产生一个有独立命名空间的新对象。每个由类产生的新对象都能读取类的属性，并获得自己的命名空间来存储数据，这些数据对于每个对象来说都不同。
- **通过继承进行定制**：类也支持OOP的继承的概念。我们可以在类的外部重新定义其属性从而扩充这个类。更通用的是，类可以建立命名空间的层次结构，而这种层次结构可以定义该结构中类创建的对象所使用的变量名。
- **运算符重载**：通过提供特定的协议方法，类可以定义对象来响应在内置类型上的几种运算。Python提供了一些可以由类使用的钩子，从而能够中断并实现任何的内置类型运算。

### 26.2 概览OOP

#### 属性继承搜索

作为动态类型脚本语言，Python把其他语言中让OOP隐藏的语法杂质和复杂性都去掉了。实际上，Python中大多数OOP的故事，都可简化成这个表达式：

```
object.attribute
```

当我们对`class`语句产生的对象使用这种方式时，这个表达式会在Python中搜索对象连接的树，来寻找`attribute`首次出现的对象。换句话说，取出属性只是简单地搜索“树”而已。我们称这种搜索程序为 ***继承*** ，因为树中位置较低的对象继承了树中位置较高的对象拥有的属性。

在Python中，我们通过代码建立连接对象树，而每次使用`object.attribute`表达式时，Python确实会在运行期间去“爬树”，来搜索属性。

下图是搜索树的一个例子：

![Figure 26-1](_static/images/Part_6_Classes_and_OOP.assets/1555603030106.png)

上图中，类树的底端有2个实例`I1`和`I2`，在它们之上有个类`C1`，而顶端有2个超类`C2`和`C3`。所有这些对象都是命名空间（变量的封装），而继承就是由下而上搜索此对象树，来寻找属性名称所出现的最低的地方。

**注意：在Python对象模型中，类和通过类产生的实例是两种不同的对象类型。**

- **类** ：类是实例工厂。类的属性提供了行为（数据和函数），所有从类产生的实例都继承该类的属性。
- **实例** ：代表程序域（program's domain）中具体的项。实例属性记录数据，而每个特定对象的数据都不同。

通常，我们把树中位置较高的类称为 ***超类***  （也叫做 *基类* ） ，位置较低的类称为 ***子类*** （也叫做 *派生类* ） 。

超类提供了所有子类共享的行为。但是因为搜索是由下而上的，子类可能可能会在树中较低位置重新定义超类的变量名，从而覆盖超类定义的行为。

以上图中的类树为例：

1. 当取出`I2.w`时，会触发对类树的搜索，依次搜索连接的对象：`I2`，`C1`，`C2`，`C3`。
2. 找到首个`w`属性就会停止搜索。但如果没有找到，就会发生一个错误。
3. 此例中，搜索到`C3`才会找到`w`。也就是说，`I2.w`会解析为`C3.w`。就OOP术语而言，`I2`继承了`C3`的属性`w`。

#### 类和实例

虽然在Python模型中，类和实例是两种不同的对象类型，但放在这些树中时，不同类型的主要用途都是用来作为另一种类型的命名空间（变量的封装）。

类和实例的主要差异在于，类是一种产生实例的工厂。

从操作的角度来说，类通常都有函数，而实例有其他的基本数据项，类的函数中使用了实例中的这些数据。在OOP中，实例就像是带有“数据”的记录，而类是处理这些记录的“程序”。

#### 类方法调用

再次考虑之前类树的例子。当我们调用方法（也就是附属于类的函数属性）时会发生什么？

- **如果这个`I2.w`引用是一个函数调用，其实际含义是“调用`C3.w`函数以处理`I2`”。也就是说，Python将会自动将`I2.w()`调用映射为`C3.w(I2)`调用，传入该实例作为继承的函数的第一个参数。**
- **也就是说，Python把隐含的实例传进方法的第一个参数，习惯性将其称为`self`。**



#### 编写类树

- 每个`class`语句会生成一个新的类对象。
- 每次类调用时，就会生成一个新的实例对象。
- 实例自动连接至创建了这些实例的类。
- 类连接至其超类的方式是，将超类列在类头部的括号内。其从左至右的顺序会决定树中的次序。

例如，要创建前例中的树，我们可以运行以下代码（这里省略了`class`语句中的内容）：

```python
class C2:                   # Make class objects (ovals)
    pass 
    
class C3: 
    pass
    
class C1(C2, C3):           # Make and link class C1 to superclasses C2, C3 (in this order)
    def setname(self, who): # Assign name: C1.setname
        self.name = who     # Self is either I1 or I2
        
I1 = C1()                   # Make instance objects (rectangles)
I2 = C1()                   # Linked to their classes
I1.setname('bob')           # Sets I1.name to 'bob'
I2.setname('sue')           # Sets I2.name to 'sue'
print(I1.name)              # Prints 'bob'
```

从技术角度来讲，这个例子使用了所谓的 ***多重继承*** 。也就是说，在类树中，类有一个以上的超类。在Python中，如果`class`语句中的小括号内有一个以上的超类（像这里的`C1`）,它们由左至右的次序会决定超类搜索的顺序。

附加在实例上的属性只属于那些实例，但附加在类上的属性则由所有子类及其实例共享。

- Attributes are usually attached to classes by assignments made at the top level in
    class statement blocks, and not nested inside function def statements there.
- Attributes are usually attached to instances by assignments to the special argument
    passed to functions coded inside classes, called self. 通过传参并赋值给类中的函数的特殊参数（也就是`self`），属性通常被附加给实例。

类通过函数（在`class`语句内由`def`语句编写而成）为实例提供行为。当`def`出现在类的内部时，通常称为 ***方法*** 。方法会自动接收第一个特殊参数（通常称为`self`），这个参数提供了被处理的实例的参照值。

> 注：如果你是用过C++或Java，就知道Python的`self`相当于`this`，但是Python中的`self`必须是明确写出的，这样使属性的读取更为明显。

因为类是多个实例的工厂，每当需要取出或设置正有某个方法调用所处理的特定实例的属性时，类的方法通常都会经历这种自动传入的`self`参数。

#### 运算符重载
略


#### OOP是为了代码重用
##### 多态(polymorphism) 和 类

***多态*** 是指运算的意义取决于运算对象。多态可用于隐藏（封装）接口差异性。

##### 自定义编程

一旦习惯了使用OOP方式进行程序设计，你就会发现，当要写新程序时，大部分任务就是把已实现的超类混合起来。





---



## 第27章 类代码编写基础
### 27.1 类产生多个实例对象

***类对象*** 提供默认行为，是 **产生多个实例对象** 的工厂。

***实例对象*** 是程序处理的实际对象：各自有独立的命名空间，但是继承创建该实例的类中的变量名。

*类对象* 来自于语句，而 *实例对象* 来自于类的调用。每次调用一个类，就会得到这个类的新的实例。


#### 类对象提供默认行为
执行`class`语句，就会得到类对象。以下是Python类主要特性的要点：
- **`class`语句创建类对象并将其赋值给变量名。** 就像`def`语句一样，`class`语句是可执行的，一般是在其所在的文件第一次被导入时执行。
- **`class`语句内的赋值语句会创建类的属性。** 就像模块文件一样，`class`语句内顶层的赋值语句（但不在`def`语句中）会产生类对象的属性。`class`语句的作用域会变成类对象的属性的命名空间，就像模块的全局作用域一样。
- **类属性提供对象的状态和行为。** 类对象的属性记录状态信息和行为，可由这个类所创建的所有实例共享。位于类中的函数`def`语句会生成 **方法** ，方法将会处理实例。



#### 实例对象是具体的元素

当调用类对象时，我们得到了实例对象。以下是关于类的实例的关键点：

- **像函数那样调用类对象会创建新的实例对象。**
- **每个实例对象继承类的属性并获得了自己的命名空间。**
- **在方法内`self`属性做赋值运算会产生每个实例自己的属性。** 在类方法内，第一个参数（按惯例称为`self`）会引用正处理的实例对象。对`self`的属性做赋值运算，会创建或修改实例内的数据，而不是类的数据。

#### 第一个例子

首先，定义一个名为`FirstClass`的类：

```python
>>> class FirstClass:              # Define a class object
        def setdata(self, value):  # Define class's methods
            self.data = value      # self is the instance
        def display(self):
            print(self.data)       # self.data: per instance
```

在类的代码块中顶层的任何变量名赋值（包括`def`语句），都会变成类的属性。所以上例中的函数对象`setdata`和`display`都成为了类`FirstClass`的属性。当函数是类的属性时，通常称为 ***方法*** 。

在方法调用时，第一个参数自动接收隐含的实例对象，即调用的主体。

调用类时会创建实例对象，实例对象会连接至创建它们的类，其实质就是实例对象可读取类属性的命名空间。以下代码，建立实例`x`和`y`：

```python
>>> x = FirstClass() # Make two instances
>>> y = FirstClass() # Each is a new namespace
```

如下图所示，建立实例`x`和`y`后，实际有3个对象，即2个实例对象和1个类对象；同时也有了三个连接的命名空间：

![Figure 27-1](_static/images/Part_6_Classes_and_OOP.assets/1555818374206.png)

类和实例是类树中通过继承搜索的相连的命名空间。这里的`data`属性会在实例内找到，但是`setdata`和`display`则是在它们之上的类中找到的。

**继承** 是在属性点号运算时发生的。对实例对象和类对象内的属性名称进行点号运算，Python会通过继承搜索从类取得变量名（除非该变量名位于实例对象内）。例如，实例对象`x`和`y`本身没有`setdata`属性，为了寻找这个属性，Python会进行继承搜索，取得类对象`FirstClass`的属性`setdata`：

```python
>>> x.setdata("King Arthur")   # Call methods: self is x
>>> y.setdata(3.14159)         # Runs: FirstClass.setdata(y, 3.14159)
```

在`FirstClass`的`setdata`函数中，传入的值会赋值给`self.data`。在方法中的第一个参数（按惯例称`self`）会自动引用正在处理的实例（如`x`和`y`），所以赋值语句会把值存储在实例的命名空间，而不是类的命名空间（这是图27-1中变量名`data`的创建的方式）。

因为类会产生多个实例，方法必须经过`self`参数才能获取正在处理的实例。所以，当调用类的`display`方法来打印`self.data`时，实例`x`和`y`的值都不同：

```python
>>> x.display()      # self.data differs in each instance
King Arthur
>>> y.display()      # Runs: FirstClass.display(y)
3.14159
```

我们可以在类的内部或外部修改实例属性。在类内时，通过方法内对`self`进行赋值运算；而在类外时，则可以通过对实例对象进行赋值运算：

```python
>>> x.data = "New value"   # Can get/set attributes
>>> x.display()            # Outside the class too
New value
```

通过在类方法函数外对变量名进行赋值运算，我们甚至可以在实例命名空间内创建全新的属性：

```python
>>> x.anothername = "spam"      # Can set new attributes here too!
```




### 27.2 类通过继承进行定制

在Python中，实例从类中继承，而类继承与超类。以下是属性继承机制的关键点：

- **超类列在了类开头的括号中。** 要继承另一个类的属性，把该类列在`class`语句开头的括号中就可以了。含有继承的类称为子类，而子类所继承的类就是其超类。
- **类从其超类中继承属性。** 就像实例继承其类中所定义的属性名一样，类也会继承其超类中定义的所有属性名称。当读取的属性不存在子类中时，Python会自动在搜索类树中搜索这个属性。
- **实例会继承所有可读取类的属性。** 每个实例会从创建它的类中获取变量名，此外，还有该类的超类。搜索变量名时，Python会检查实例，然后是它的类，最后是所有超类。
- **每个`object.attribute`都会开启新的独立搜索。** Python会对每个属性读取表达式进行对类树的独立搜索。这包括在`class`语句外对实例和类的引用，以及在类方法函数内对`self`实例参数属性的引用。方法中的每个`self.attr`表达式都会开启对`self`及其上层的类的`attr`属性的搜索。
- **逻辑的修改是通过创建子类，而不是修改超类。** 在树中层次较低的子类中重新定义超类的变量名，子类就可取代并定制所继承的行为。


#### 第二个例子

下例建立在上一个例子的基础之上，定义一个继承类`FirstClass`所有变量名的新类`SecondClass`，并提供一个其自己的变量名`display`，我们把这种在树中较低处发生的重新定义的、取代属性的动作称为 ***重载*** 。

```python
>>> class SecondClass(FirstClass):   # Inherits setdata
...     def display(self):           # Changes display
...         print('Current value = "%s"' % self.data)
...
```

结果就是`SecondClass`改变了方法`display`的行为，但依然会继承`FirstClass`的`setdata`方法：

```python
>>> z = SecondClass()
>>> z.setdata(42)          # Finds setdata in FirstClass
>>> z.display()            # Finds overridden method in SecondClass
Current value = "42"
```

下图描绘了上面操作涉及的命名空间：

![Figure 27-2](_static/images/Part_6_Classes_and_OOP.assets/1555841428600.png)

#### 类是模块内的属性
当`class`语句执行时，这只是赋值给对象的变量，而对象可以用到任何普通表达式引用。

类是模块内的属性，如果`FirstClass`是写在模块内的，就可以将其导入：
```python
from modulename import FirstClass # Copy name into my scope
class SecondClass(FirstClass): # Use class name directly
    def display(self): ...
```
等价于：
```python
import modulename # Access the whole module
class SecondClass(modulename.FirstClass): # Qualify to reference
    def display(self): ...
```


### 27.3 类可以截获Python运算符
简而言之，运算符重载就是用类编码的对象截获（intercept）和响应对内置类型起作用的运算：加法、分片、打印和点号运算等。

以下是运算符重载的主要概念：
- **以双下划线命名的方法（如`__X__`）是特殊钩子。** Python运算符重载的实现是提供特殊命名的方法来拦截运算。Python语言替每种运算和特殊命名的方法之间，定义了固定不变的映射关系。
- **当实例出现在内置运算时，这类方法会自动调用。** 例如，如果实例对象继承了`__add__`方法，当对象出现在`+`表达式内时，该方法就会调用。该方法的返回值会变成相应表达式的结果。
- **类可覆盖多数内置类型运算。** 有几十种特殊运算符重载的方法名称，几乎可截获并实现内置类型的所有运算。它不仅包含了表达式，而且像打印和对象建立这类基本运算也包括在内。
- **运算符覆盖方法没有默认值，而且也不需要。** 如果类没有定义或继承运算符重载方法，就是说相应的运算在类实例中并不支持。例如，如果没有`__add__`，表达式`+`就会引发异常。
- **New-style classes have some defaults, but not for common operations. 新式类有一些默认值，但不是针对通用运算符。** In Python 3.X, and so-called “new style” classes in 2.X that we’ll define later, a root class named object does provide defaults for some __X__ methods, but not for many, and not for most commonly used operations.在Python 3.X中，以及在Python 2.X中也叫做新式类，一个叫做`object`的根类确实为一些`__X__`方法提供了默认值，但不并不多，尤其是不针对大多数通用的运算。
- **运算符允许类与Python的对象模型相集成。** 重载类型运算时，以类实现的用户定义对象的行为就会像内置对象一样，因此，提供了一致性，以及与预期接口的兼容性。



#### 第三个例子

在这个例子中，我们要定义`SecondClass`的子类，实现三个特殊名称的属性，让Python自动进行调用：

- 当新的实例被创建时，构造函数`__init__`会被调用。其中，`__init__`方法的参数`self`是`ThirdClass`类的新对象。
- 当一个`ThirdClass`的实例出现在表达式`+`中时，`__add__`会被调用。
- 当一个对象被打印（技术上来说，当它被内置函数`str`转换为它的打印字符串或它的Python内部等价），`__str__`会被调用。

```python
class ThirdClass(SecondClass): # Inherit from SecondClass
    def __init__(self, value): # On "ThirdClass(value)"
        self.data = value
    def __add__(self, other): # On "self + other"
        return ThirdClass(self.data + other)
    def __str__(self): # On "print(self)", "str()"
        return '[ThirdClass: %s]' % self.data
    def mul(self, other): # In-place change: named
        self.data *= other
```

```python
>>> a = ThirdClass('abc') # __init__ called
>>> a.display() # Inherited method called
Current value = "abc"
>>> print(a) # __str__: returns display string
[ThirdClass: abc]
>>> b = a + 'xyz' # __add__: makes a new instance
>>> b.display() # b has all ThirdClass methods
Current value = "abcxyz"
>>> print(b) # __str__: returns display string
[ThirdClass: abcxyz]
>>> a.mul(3) # mul: changes instance in place
>>> print(a)
[ThirdClass: abcabcabc]
```

#### 为什么要使用运算符重载

选择使用或不使用运算符重载，取决于你有多想让对象的用法和外观看起来更像内置类型。

几乎每个实际的类都会出现的一个重载方法是：`__init__`构造函数。它让类立即在其新建的实例内添加属性。


### 27.4 世界上最简单的Python类

实际上，我们建立的类中可以完全没有属性，而仅仅是空的命名空间对象：

```python
>>> class rec: pass     # Empty namespace object
```

建立这个类后，可以完全在最初的`class`语句外，通过赋值变量名给这个类增加属性：

```python
>>> rec.name = 'Bob'    # Just objects with attributes
>>> rec.age = 40
```

通过赋值语句创建这些属性后，就可以用一般的语法将它们取出。这样使用类时，类差不多就像是C语言中的`struct`：

```python
>>> print(rec.name)    # Like a C struct
Bob
```

创建`rec`类的实例`x`和`y`：

```python
>>> x = rec()    # Instances inherit class names
>>> y = rec()
```

虽然`x`和`y`本身没有它们自己的属性，是完全空的命名空间对象，但因为继承了`rec`，所以`x`和`y`会获取到`rec`类的属性：
```python
>>> x.name, y.name   # name is stored on the class only
('Bob', 'Bob')
```

如果把一个属性赋值给一个实例，就会在该对象内创建或修改该属性，而不会因属性的引用而启动继承搜索，因为属性赋值运算只会影响属性赋值所在的对象：

```python
>>> x.name = 'Sue'            # But assignment changes x only
>>> rec.name, x.name, y.name  # y依然继承类rec的属性name
('Bob', 'Sue', 'Bob')
```



事实上，命名空间对象的属性通常都是以字典的形式实现的，而类继承树只是连接至其他字典的字典而已。例如，`__dict__`属性是针对大多数基于类的对象的命名空间字典。一些类也可能在`__slots__`中定义了属性。

```python
>>> rec.__dict__
mappingproxy({'__module__': '__main__', '__dict__': <attribute '__dict__' of 'rec' objects>, '__weakref__': <attribute '__weakref__' of 'rec' objects>, '__doc__': None})
>>> rec.__dict__.keys()
dict_keys(['__module__', '__dict__', '__weakref__', '__doc__'])
```
```python
>>> x.__dict__.keys()
dict_keys(['name'])
>>> x.__dict__
{'name': 'Sue'}
```
```python
>>> y.__dict__
{}
>>> y.__dict__.keys()
dict_keys([])
```

每个实例都连接至其类以便于继承，可以通过`__class__`查看：

```python
>>> x.__class__
<class '__main__.rec'>
>>> y.__class__
<class '__main__.rec'>
```

每个类也有一个`__base__`属性，它是其超类的元组：

```python
>>> rec.__base__
<class 'object'>
```



即使是方法（通常在类中通过`def`创建），也可以完全独立地在任意类对象的外部创建，然后赋值给类作为其属性：

```python
>>> def uppername(obj):
...     return obj.name.upper() # Still needs a self argument (obj)
...
>>> rec.method = uppername # Now it's a class's method!
>>> x.method() # Run method to process x
'SUE'
>>> y.method() # Same, but pass y to self
'BOB'
>>> rec.method(x) # Can call through instance or class
'SUE'
```

当然这个函数也可以直接调用，只要我们手动传入一个带有`name`属性的对象：

```python
>>> uppername(x) # Call as a simple function
'SUE'
```



#### 类VS字典

使用元组来记录实体的属性：

```python
>>> rec = ('Bob', 40.5, ['dev', 'mgr']) # Tuple-based record
>>> print(rec[0])
Bob
```

使用字典来记录实体的属性：
```python
>>> rec = {}
>>> rec['name'] = 'Bob' # Dictionary-based record
>>> rec['age'] = 40.5 # Or {...}, dict(n=v), etc.
>>> rec['jobs'] = ['dev', 'mgr']
>>>
>>> print(rec['name'])
Bob
```

使用类来记录实体的属性：
```python
>>> class rec: pass
>>> rec.name = 'Bob' # Class-based record
>>> rec.age = 40.5
>>> rec.jobs = ['dev', 'mgr']
```

使用类的实例对象来记录实体的属性：
```python
>>> class rec: pass
>>> pers1 = rec() # Instance-based records
>>> pers1.name = 'Bob'
>>> pers1.jobs = ['dev', 'mgr']
>>> pers1.age = 40.5
>>>
>>> pers2 = rec()
>>> pers2.name = 'Sue'
>>> pers2.jobs = ['dev', 'cto']
>>>
>>> pers1.name, pers2.name
('Bob', 'Sue')
```

编写一个完整的类，来记录实体的属性，并提供对属性操作的行为：
```python
>>> class Person:
        def __init__(self, name, jobs, age=None): # class = data + logic
            self.name = name
            self.jobs = jobs
            self.age = age
        def info(self):
            return (self.name, self.jobs)

>>> rec1 = Person('Bob', ['dev', 'mgr'], 40.5) # Construction calls
>>> rec2 = Person('Sue', ['dev', 'cto'])
>>>
>>> rec1.jobs, rec2.info() # Attributes + methods
(['dev', 'mgr'], ('Sue', ['dev', 'cto']))
```

尽管像字典这样的类型使用起来十分灵活，但是类允许我们已内置类型和简单函数不能直接支持的方式为对象添加行为。尽管我们也可以把函数存储到字典中，但使用类来处理隐含的实例更加自然。



---




## 第28章 更多实例
在本章中，我们将编写两个类：
- `Person`：创建并处理关于人员的信息的一个类。
- `Manager`：一个定制的`Person`，修改了继承的行为。

在这个过程中，我们将创建2个类的实例，并测试它们的功能。最后，我们将把实例存储到一个`shelve`的面向对象数据库中，使它们持久化。通过这种方式，我们可以把这些代码用作模板，从而发展为完全用Python编写的一个完备的个人数据库。
### 28.1 步骤1：创建实例

在Python中，模块名使用小写字母开头，而类名使用一个大写字母开头，这通常是惯例。

按照惯例，我们将新的模块文件命名为`person.py`，将其中的类命名为`Person`，如下所示：

```python
# File person.py (start)
class Person: # Start a class
```

### 28.2 步骤2：添加行为方法

通过 ***实例对象属性*** 记录人员的基本信息。

实例对象属性通常通过给类方法函数中的`self`属性赋值来创建。

每次创建一个实例时，Python自动运行构造函数`__init__`。构造函数`__init__`通过其第一个参数`self`给实例对象属性赋第一个值以创建该对象实例属性。

```python
# Add record field initialization
class Person:
    def __init__(self, name, job, pay): # Constructor takes three arguments
        self.name = name # Fill out fields when created
        self.job = job # self is the new instance object
        self.pay = pay
```




### 28.3 步骤3：运算符重载


### 28.4 步骤4：通过子类定制行为


### 28.5 步骤5：自定义构造函数


### 28.6 步骤6：使用内省工具


### 28.7 步骤7（最后一步）：把对象存储到数据库中


### 28.8 未来的方向



---


## 第29章 类编码细节
### 29.1 `class`语句


### 29.2 方法


### 29.3 继承


### 29.4 命名空间：结论


### 29.5 回顾文档字符串


### 29.6 类VS模块



---


## 第30章 运算符重载



---


## 第31章 类的设计
