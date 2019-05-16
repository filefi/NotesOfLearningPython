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

#### 编写构造函数

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

虽然构造函数`__init__`的名字很怪异，但它仍然是一个常规的函数，并且支持所有的函数特性。例如，我们可以为它的参数提供默认值：
```python
# Add defaults for constructor arguments
class Person:
    def __init__(self, name, job=None, pay=0): # Normal function args
        self.name = name
        self.job = job
        self.pay = pay
```

#### 在进行中测试

用Python编程其实就是一种 ***增量原型*** ，编写一些代码，测试它，编写更多代码，再次测试，以此类推。

#### 以两种方式使用代码

使用语句块`if __name__ == '__main__':`控制测试代码的运行：

```python
# Allow this file to be imported as well as run/tested
class Person:
    def __init__(self, name, job=None, pay=0):
        self.name = name
        self.job = job
        self.pay = pay
        
if __name__ == '__main__': # When run for testing only
    # self-test code
    bob = Person('Bob Smith')
    sue = Person('Sue Jones', job='dev', pay=100000)
    print(bob.name, bob.pay)
    print(sue.name, sue.pay)
```

因为`__name__`等于`__main__`，所以把文件作为顶层脚本运行时，就能够运行语句块`if __name__ == '__main__':`内的测试代码：
```python
C:\code> person.py
Bob Smith 0
Sue Jones 100000
```

而把它作为类库导入的时候，则不会运行测试代码：

```python
C:\code> python
Python 3.3.0 (v3.3.0:bd8afb90ebf2, Sep 29 2012, 10:57:17) ...
>>> import person
>>>
```



### 28.2 步骤2：添加行为方法

#### 编写方法

把操作实例对象属性的代码移入类方法中，从而实现 ***封装*** 。

```python
# Add methods to encapsulate operations for maintainability
class Person:
    def __init__(self, name, job=None, pay=0):
        self.name = name
        self.job = job
        self.pay = pay
    def lastName(self): # Behavior methods
        return self.name.split()[-1] # self is implied subject
    def giveRaise(self, percent):
        self.pay = int(self.pay * (1 + percent)) # Must change here only

if __name__ == '__main__':
    bob = Person('Bob Smith')
    sue = Person('Sue Jones', job='dev', pay=100000)
    print(bob.name, bob.pay)
    print(sue.name, sue.pay)
    print(bob.lastName(), sue.lastName()) # Use the new methods
    sue.giveRaise(.10) # instead of hardcoding
    print(sue.pay)
```

***方法*** 只是附加给类并旨在处理那些类的实例的常规函数。实例是方法调用的主体，并且会自动传递给方法的`self`参数。

### 28.3 步骤3：运算符重载

如果类中定义了`__str__`和`__repr__`，或者从一个超类继承了这两个方法，则每次一个实例转换为其打印字符串的时候，`__str__`和`__repr__`就会自动运行，其直接的效果就是，打印一个对象会显示对象的`__str__`和`__repr__`方法所返回的内容。

这里我们重载运算符`__repr__`，以使打印类的实例时，会列出属性：

```python
# Add __repr__ overload method for printing objects
class Person:
    def __init__(self, name, job=None, pay=0):
        self.name = name
        self.job = job
        self.pay = pay
    def lastName(self):
        return self.name.split()[-1]
    def giveRaise(self, percent):
        self.pay = int(self.pay * (1 + percent))
    def __repr__(self): # Added method
        return '[Person: %s, %s]' % (self.name, self.pay) # String to print

if __name__ == '__main__':
    bob = Person('Bob Smith')
    sue = Person('Sue Jones', job='dev', pay=100000)
    print(bob)
    print(sue)
    print(bob.lastName(), sue.lastName())
    sue.giveRaise(.10)
    print(sue)
```



### 28.4 步骤4：通过子类定制行为

#### 编写子类

定义一个`Person`的子类`Manager`，并重写超类中的`giveRaise`方法。

#### 扩展方法：不好的方式

以复制粘贴超类`Person`中的代码的方式扩展子类的方法是非常不好的。因为如果一旦改变了涨工资的方式，将必须修改超类和子类两处代码。

```python
class Manager(Person):
    def giveRaise(self, percent, bonus=.10):
        self.pay = int(self.pay * (1 + percent + bonus)) # Bad: cut and paste
```

#### 扩展方法：好的方式

使用扩展参数来直接调用其超类中最初的版本是扩展方法的好方式：

```python
class Manager(Person):
    def giveRaise(self, percent, bonus=.10):
        Person.giveRaise(self, percent + bonus) # Good: augment original
```

这段代码利用了这样一个事实： **类方法总是可以在一个实例中调用（Python自动地把实例发送给方法的`self`参数），或者通过类来调用（必须手动地传递实例给类方法）。** 也就是说：

```python
instance.method(args...)
```

等价于：

```python
class.method(instance, args...)
```

通过类直接调用，有效地扰乱了继承，并且把调用沿着类树向上传递以运行一个特定的版本。

以下是实现本步骤的整个模块文件：

```python
# Add customization of one behavior in a subclass
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

class Manager(Person):
    def giveRaise(self, percent, bonus=.10): # Redefine at this level
        Person.giveRaise(self, percent + bonus) # Call Person's version

if __name__ == '__main__':
    bob = Person('Bob Smith')
    sue = Person('Sue Jones', job='dev', pay=100000)
    print(bob)
    print(sue)
    print(bob.lastName(), sue.lastName())
    sue.giveRaise(.10)
    print(sue)
    tom = Manager('Tom Jones', 'mgr', 50000) # Make a Manager: __init__
    tom.giveRaise(.10) # Runs custom version
    print(tom.lastName()) # Runs inherited method
    print(tom) # Runs inherited __repr__
```

> **关于`super`**
>
> To extend inherited methods, the examples in this chapter simply call the original through the superclass name: Person.giveRaise(...). This is the traditional and simplest scheme in Python, and the one used in most of this book.
>
> Java programmers may especially be interested to know that Python also has a super built-in function that allows calling back to a superclass’s methods more generically— but it’s cumbersome to use in 2.X; differs in form between 2.X and 3.X; relies on unusual semantics in 3.X; works unevenly with Python’s operator overloading; and does not always mesh well with traditionally coded multiple inheritance, where a single superclass call won’t suffice.
>
> In its defense, the super call has a valid use case too—cooperative same-named method dispatch in multiple inheritance trees—but it relies on the “MRO” ordering of classes, which many find esoteric and artificial; unrealistically assumes universal deployment to be used reliably; does not fully support method replacement and varying argument lists; and to many observers seems an obscure solution to a use case that is rare in real Python code.
>
> Because of these downsides, this book prefers to call superclasses by explicit name instead of super, recommends the same policy for newcomers, and defers presenting super until Chapter 32. It’s usually best judged after you learn the simpler, and generally more traditional and “Pythonic” ways of achieving the same goals, especially if you’re new to OOP. Topics like MROs and cooperative multiple inheritance dispatch seem a lot to ask of beginners—and others.
>
> And to any Java programmers in the audience: I suggest resisting the temptation to use Python’s super until you’ve had a chance to study its subtle implications. Once you step up to multiple inheritance, it’s not what you think it is, and more than you probably expect. The class it invokes may not be the superclass at all, and can even vary per context. Or to paraphrase a movie line: Python’s super is like a box of chocolates—you never know what you’re going to get!

#### 多态实战

为了使得这个从继承获取的行为更为惊人，我们在文件的末尾添加了如下代码：

```python
if __name__ == '__main__':
    ...
    print('--All three--')
    for obj in (bob, sue, tom): # Process objects generically
        obj.giveRaise(.10) # Run this object's giveRaise
        print(obj) # Run the common __repr__
```

这里利用了Python中的 ***多态*** 。

#### 继承、定制和扩展

实际上，类比我们的例子所展示的更加灵活。通常，类可以 **继承** 、**定制** 或 **扩展** 超类中已有的代码。

例如，我们可以为`Manager`添加`Person`中所没有的方法`someThingElse`：

```python
class Person:
    def lastName(self): ...
    def giveRaise(self): ...
    def __repr__(self): ...
        
class Manager(Person): # Inherit
    def giveRaise(self, ...): ... # Customize
    def someThingElse(self, ...): ... # Extend
        
tom = Manager()
tom.lastName() # Inherited verbatim
tom.giveRaise() # Customized version
tom.someThingElse() # Extension here
print(tom) # Inherited overload method
```



#### OOP：大思路

在OOP中，我们通过 **定制** 来编程，而不是复制和修改已有的代码。

我们可以通过编写新的子类来裁剪或扩展之前已经做过的工作，而不是每次从头开始。

### 28.5 步骤5：自定义构造函数

在上一个例子中，我们必须为`Manager`对象提供一个`job`参数`mgr`，这有点多余。我们重新编写新的`Manager`构造函数，并把创建对象的调用修改为自动传入`mgr`作为超类构造函数的`job`参数。

```python
# File person.py
# Add customization of constructor in a subclass
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
    
class Manager(Person):
    def __init__(self, name, pay): # Redefine constructor
        Person.__init__(self, name, 'mgr', pay) # Run original with 'mgr'
    def giveRaise(self, percent, bonus=.10):
        Person.giveRaise(self, percent + bonus)

if __name__ == '__main__':
    bob = Person('Bob Smith')
    sue = Person('Sue Jones', job='dev', pay=100000)
    print(bob)
    print(sue)
    print(bob.lastName(), sue.lastName())
    sue.giveRaise(.10)
    print(sue)
    tom = Manager('Tom Jones', 50000) # Job name not needed:
    tom.giveRaise(.10) # Implied/set by class
    print(tom.lastName())
    print(tom)
```



#### OOP比我们认为的要简单

Python中OOP的机制：

- 实例创建：填充实例属性。
- 行为方法：在类方法中封装逻辑。
- 运算符重载：为打印这样的内置操作提供行为。
- 定制行为：重新定义子类中的方法以使其特殊化。
- 定制构造函数：为超类步骤添加初始化逻辑。

#### 组合类的其他方式

一种常见的编码模式是把对象彼此嵌套以组成 ***复合对象*** 。

下面使用这种组合的思想来编写`Manager`扩展代码，将它嵌入一个`Person`中，而不是继承`Person`。这里使用重载`__getattr__`运算符的方法来做到这点：

```python
# File person-composite.py
# Embedding-based Manager alternative
class Person:
    ...same...
class Manager:
    def __init__(self, name, pay):
        self.person = Person(name, 'mgr', pay) # Embed a Person object
    def giveRaise(self, percent, bonus=.10):
        self.person.giveRaise(percent + bonus) # Intercept and delegate
    def __getattr__(self, attr):
        return getattr(self.person, attr) # Delegate all other attrs
    def __repr__(self):
        return str(self.person) # Must overload again (in 3.X)

if __name__ == '__main__':
    ...same...
```

此外，像下面这样的一个假设的`Department`可能聚合其他的对象，以便将它们当作一个集合对待。将这段代码添加到`person.py`文件的底部：

```python
# File person-department.py
# Aggregate embedded objects into a composite
class Person:
    ...same...
    
class Manager(Person):
    ...same...
    
class Department:
    def __init__(self, *args):
        self.members = list(args)
    def addMember(self, person):
        self.members.append(person)
    def giveRaises(self, percent):
        for person in self.members:
            person.giveRaise(percent)
    def showAll(self):
        for person in self.members:
            print(person)
            
if __name__ == '__main__':
    bob = Person('Bob Smith')
    sue = Person('Sue Jones', job='dev', pay=100000)
    tom = Manager('Tom Jones', 50000)
    development = Department(bob, sue) # Embed objects in a composite
    development.addMember(tom)
    development.giveRaises(.10) # Runs embedded objects' giveRaise
    development.showAll() # Runs embedded objects' __repr__
```



> **在Python 3.X中捕获内置属性**
>
> 高级特性，暂略

### 28.6 步骤6：使用内省工具

我们的代码仍然存在两个问题：

- 当打印`tom`时，`Manager`会把它标记为`Person`。
- 我们还无法通过`Manager`的构造函数验证`tom`工作名已经正确地设置为`mgr`，因为我们为`Person`编写的`__str__`没有打印出这个字段。

#### 特殊类属性

***内省工具*** 是特殊的属性和函数，允许我们访问对象实现的一些内部机制，并允许我们编写以通用方式处理类的代码。通常只有为程序员开发工具的人才会用到这些高级工具。

有两个钩子可以解决我们的问题：

- 内置的`instance.__class__`属性提供了一个从实例到创建它的类的链接。类则有`__name__`和`__bases__`序列，提供了超类的访问。我们使用这些来打印创建一个实例的类的名字，而不是通过硬编码来做到。
- 内置的`object.__dict__`属性提供了一个字典，带有一个“键/值”对，以便每个属性都附加到一个命名控件对象（包括模块、类和实例）。由于它是字典，因此我们可以获取键的列表、按键来索引、迭代键，等等，从而更广泛地处理所有的属性。

下面是这些工具在Python交互模式中实际使用的情形：

```python
>>> from person import Person
>>> bob = Person('Bob Smith')
>>> bob # Show bob's __repr__ (not __str__)
[Person: Bob Smith, 0]
>>> print(bob) # Ditto: print => __str__ or __repr__
[Person: Bob Smith, 0]
>>>
>>> bob.__class__ # Show bob's class and its name
<class 'person.Person'>
>>> bob.__class__.__name__
'Person'
>>>
>>> list(bob.__dict__.keys()) # Attributes are really dict keys
['pay', 'job', 'name'] # Use list to force list in 3.X
>>>
>>> for key in bob.__dict__:
        print(key, '=>', bob.__dict__[key]) # Index manually
pay => 0
job => None
name => Bob Smith
>>> for key in bob.__dict__:
        print(key, '=>', getattr(bob, key)) # obj.attr, but attr is a var
pay => 0
job => None
name => Bob Smith
```

> 如果一个实例的类定义了`__slots__`，而实例可能没有存储在`__dict__`字典中，但实例的一些属性也是可以访问的。这是新式类（以及Python 3.X中所有类）的一项可选的和相对不太明确的功能，即，把属性存储在数组中。

#### 一种通用显示工具

在下面的模块`classtools.py`中，类`AttrDisplay`将对任何实例有效，不管实例的属性集合是什么。这使得类`AttrDisplay`变成了一个公用的工具。通过继承，它可以混合到想到使用它显示格式的任何类中。

```python
# File classtools.py (new)
"Assorted class utilities and tools"
class AttrDisplay:
    """
    Provides an inheritable display overload method that shows
    instances with their class names and a name=value pair for
    each attribute stored on the instance itself (but not attrs
    inherited from its classes). Can be mixed into any class,
    and will work on any instance.
    """
    def gatherAttrs(self):
        attrs = []
        for key in sorted(self.__dict__):
        attrs.append('%s=%s' % (key, getattr(self, key)))
        return ', '.join(attrs)
    def __repr__(self):
        return '[%s: %s]' % (self.__class__.__name__, self.gatherAttrs())

if __name__ == '__main__':
    class TopTest(AttrDisplay):
        count = 0
        def __init__(self):
            self.attr1 = TopTest.count
            self.attr2 = TopTest.count+1
            TopTest.count += 2

    class SubTest(TopTest):
        pass

X, Y = TopTest(), SubTest() # Make two instances
print(X) # Show all instance attrs
print(Y) # Show lowest class name
```

直接运行模块`classtools.py`，这个模块的自测试代码会创建2个实例并打印它们。这里定义的`__str__`显示了实例的类，及其所有的属性名和值，并按照属性名排序：

```python
C:\code> classtools.py
[TopTest: attr1=0, attr2=1]
[SubTest: attr1=2, attr2=3]
```

#### 实例VS类属性

可以用`__class__`连接爬升到实例的类，然后使用`__dict__`去获取类属性，然后迭代类的`__bases__`属性爬升到更高的超类。有必要的话还可以重复此过程。

```python
>>> list(bob.__dict__.keys()) # 3.X keys is a view, not a list
['name', 'job', 'pay']
>>> dir(bob) # 3.X includes class type methods
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__',
'__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__',
...more omitted: 31 attrs...
'__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__',
'giveRaise', 'job', 'lastName', 'name', 'pay']
```

#### 工具类的命名考虑

为了减少名称冲突的机会，Python程序员常常对于不想做其他用途的方法添加一个 **单下划线** 的前缀，在这个例子中就是`_gatherAttrs`。

另一种更好的，但不太常用的方法是，在方法名前面使用 **两个下划线** 的前缀，在这个例子中就是`__gatherAttrs`。Python自动扩展这样的名称，以包含类的名称，从而使它们变得真正唯一。这一功能叫做 **伪私有类属性** 。

#### 类的最终形式

新的打印重载方法将会由`Person`的实例继承，`Manager`的实例也会继承`Person`的`__str__`。

```python
# File classtools.py (new)
...as listed earlier...
# File person.py (final)
"""
Record and process information about people.
Run this file directly to test its classes.
"""
from classtools import AttrDisplay # Use generic display tool

class Person(AttrDisplay): # Mix in a repr at this level
    """
    Create and process person records
    """
    def __init__(self, name, job=None, pay=0):
        self.name = name
        self.job = job
        self.pay = pay

    def lastName(self): # Assumes last is last
        return self.name.split()[-1]
    
    def giveRaise(self, percent): # Percent must be 0..1
        self.pay = int(self.pay * (1 + percent))
        
class Manager(Person):
    """
    A customized Person with special requirements
    """
    def __init__(self, name, pay):
        Person.__init__(self, name, 'mgr', pay) # Job name is implied
    def giveRaise(self, percent, bonus=.10):
        Person.giveRaise(self, percent + bonus)
        
if __name__ == '__main__':
    bob = Person('Bob Smith')
    sue = Person('Sue Jones', job='dev', pay=100000)
    print(bob)
    print(sue)
    print(bob.lastName(), sue.lastName())
    sue.giveRaise(.10)
    print(sue)
    tom = Manager('Tom Jones', 50000)
    tom.giveRaise(.10)
    print(tom.lastName())
    print(tom)
```

由于`AttrDisplay`直接从`self`实例中提取了类名，所以每个对象都显示其最近的（最低的）类的名称：`tom`现在显示为`Manager`，而不是`Person`。并且我们最终验证了其`job`也已经由`Manager`的构造函数正确地填充为`mgr`：

```python
C:\code> person.py
[Person: job=None, name=Bob Smith, pay=0]
[Person: job=dev, name=Sue Jones, pay=100000]
Smith Jones
[Person: job=dev, name=Sue Jones, pay=110000]
Jones
[Manager: job=mgr, name=Tom Jones, pay=60000]
```



### 28.7 步骤7（最后一步）：把对象存储到数据库中

在最后一步，让我们把对象持久化。

#### Pickle和Shelve

对象持久化通过3个标准的库模块来实现，这3个模块在Python中都可用：

- `pickle`：任意Python对象的字节串之间的序列化。
- `dbm`（在Python 2.X中叫做`anydbm`）：实现一个可通过键访问的文件系统，以存储字符串。
- `shelve`：使用另两个模块按照键把Python对象存储到一个文件中。

##### `pickle`模块

`pickle`模块是一种非常通用的对象序列化和反序列化工具：能够将内存中几乎任何的Python对象转换为字节串。这个自己串可以在随后用来在内存中重新构建最初的对象。

##### `shelve`模块

`shelve`使用`pickle`把一个对象转换为`pickle`化的字符串，并将其存储在一个`dbm`文件中的键之下；之后载入的时候，`shelve`通过键获取`pickle`化的字符串，并用`pickle`在内存中重新创建最初的对象。

对于脚本来说，一个`shelve`的`pickle`化的对象看上去就像是字典，它们之间唯一的区别就是：必须先 **打开** `shelve`并且在修改之后，必须 **关闭** 它。`shelve`自动吧字典操作映射到存储在文件中的对象。

最终的效果就是，一个`shelve`提供了一个简单的数据库来按照键存储和获取本地的Python对象，并由此使它们跨程序运行而保持持久化。

#### 在shelve数据库中存储对象

```python
# File makedb.py: store Person objects on a shelve database
from person import Person, Manager # Load our classes
bob = Person('Bob Smith') # Re-create objects to be stored
sue = Person('Sue Jones', job='dev', pay=100000)
tom = Manager('Tom Jones', 50000)

import shelve
db = shelve.open('persondb') # Filename where objects are stored
for obj in (bob, sue, tom): # Use object's name attr as key
    db[obj.name] = obj # Store object on shelve by key
db.close() # Close after making changes
```

#### 交互地探索`shelve`

此时，当前的目录下会有一个或多个文件名以`persondb`开头的文件。我们的数据库存储在3个文件中：

```python
>>> import glob
>>> glob.glob('persondb*')
['persondb.bak', 'persondb.dat', 'persondb.dir']
```

如果愿意，可以查看`shelve`的文件，但是它们是二进制文件，并且大多数内容对于`shelve`模块以外的环境没有太多意义。
```python
>>> print(open('persondb.dir').read())
'Sue Jones', (512, 92)
'Tom Jones', (1024, 91)
'Bob Smith', (0, 80)
>>> print(open('persondb.dat','rb').read())
b'\x80\x03cperson\nPerson\nq\x00)\x81q\x01}q\x02(X\x03\x00\x00\x00jobq\x03NX\x03\x00
...more omitted...
```

由于`shelve`是包含了对象的对象，所以我们可以用常规的Python语法和开发模式来处理它：
```python
>>> import shelve
>>> db = shelve.open('persondb')        # Reopen the shelve
>>> len(db)                             # Three 'records' stored
3
>>> list(db.keys())                     # keys is the index
['Sue Jones', 'Tom Jones', 'Bob Smith'] # list() to make a list in 3.X
>>> bob = db['Bob Smith']               # Fetch bob by key
>>> bob                                 # Runs __repr__ from AttrDisplay
[Person: job=None, name=Bob Smith, pay=0]
>>> bob.lastName()                      # Runs lastName from Person
'Smith'
>>> for key in db:                      # Iterate, fetch, print
print(key, '=>', db[key])
Sue Jones => [Person: job=dev, name=Sue Jones, pay=100000]
Tom Jones => [Manager: job=mgr, name=Tom Jones, pay=50000]
Bob Smith => [Person: job=None, name=Bob Smith, pay=0]
>>> for key in sorted(db):
print(key, '=>', db[key])               # Iterate by sorted keys
Bob Smith => [Person: job=None, name=Bob Smith, pay=0]
Sue Jones => [Person: job=dev, name=Sue Jones, pay=100000]
Tom Jones => [Manager: job=mgr, name=Tom Jones, pay=50000]
```

注意，为了载入和使用存储的对象，我们不必先导入`Person`或`Manager`类。因为Python对一个类实例进行`pickle`操作会记录其`self`实例属性，以及创建实例的类的名字和位置。当随后从`shelve`中获取`bob`并对其`unpickle`的时候，Python将自动地重新导入该类并且将`bob`连接到它。

**只有在创建新势力的时候，才必须提前导入类。在处理已有实例的时候，则不必提前导入类。**

#### 更新`shelve`中的对象

如下的文件`updatedb.py`打印出数据库，并且每次把我们所存储的对象之一增加一次。

```python
# File updatedb.py: update Person object on database
import shelve
db = shelve.open('persondb')      # Reopen shelve with same filename

for key in sorted(db):            # Iterate to display database objects
    print(key, '\t=>', db[key])       # Prints with custom format

sue = db['Sue Jones']             # Index by key to fetch
sue.giveRaise(.10)                # Update in memory using class's method
db['Sue Jones'] = sue             # Assign to key to update in shelve
db.close()                        # Close after making changes
```

运行这段脚本几次以看到对象的变化：
```python
C:\code> updatedb.py
Bob Smith => [Person: job=None, name=Bob Smith, pay=0]
Sue Jones => [Person: job=dev, name=Sue Jones, pay=100000]
Tom Jones => [Manager: job=mgr, name=Tom Jones, pay=50000]
C:\code> updatedb.py
Bob Smith => [Person: job=None, name=Bob Smith, pay=0]
Sue Jones => [Person: job=dev, name=Sue Jones, pay=110000]
Tom Jones => [Manager: job=mgr, name=Tom Jones, pay=50000]
C:\code> updatedb.py
Bob Smith => [Person: job=None, name=Bob Smith, pay=0]
Sue Jones => [Person: job=dev, name=Sue Jones, pay=121000]
Tom Jones => [Manager: job=mgr, name=Tom Jones, pay=50000]
C:\code> updatedb.py
Bob Smith => [Person: job=None, name=Bob Smith, pay=0]
Sue Jones => [Person: job=dev, name=Sue Jones, pay=133100]
Tom Jones => [Manager: job=mgr, name=Tom Jones, pay=50000]
```

再一次，我们可以在交互模式中验证脚本的作用：
```python
C:\code> python
>>> import shelve
>>> db = shelve.open('persondb') # Reopen database
>>> rec = db['Sue Jones'] # Fetch object by key
>>> rec
[Person: job=dev, name=Sue Jones, pay=146410]
>>> rec.lastName()
'Jones'
>>> rec.pay
146410
```


### 28.8 未来的方向
略



---


## 第29章 类编码细节
### 29.1 `class`语句

和C++或Java不同，Python的`class`语句并不是声明。

就像`def`一样，`class`语句是对象的创建者并且是一个隐含的赋值运算：执行时，它会创建类对象，然后把其引用值存储在前面所使用的变量名。

就像`def`一样，`class`语句也是真正的可执行代码。直到Python抵达并运行定义的`class`语句前，你的类都不存在。

#### 一般形式

`class`是复合语句，其主体一般出现在缩进的语句块中。以下是`class`语句的一般形式：

```python
class name(superclass,...):  # Assign to name
    attr = value             # Shared class data
    def method(self,...):    # Methods
        self.attr = value    # Per-instance data
```

在`class`语句内，任何赋值语句都会产生类属性和特殊命名的方法重载运算符。例如，名为`__init__`的函数会在实例对象构造时调用（如果定义过的话）。

#### 例子

类几乎就是命名空间，因此，类就像模块和函数：

- 就像函数一样，`class`语句是本地作用域，由内嵌的赋值语句建立的变量名，就存在于这个本地作用域内。
- 就像模块内的变量名，在`class`语句内赋值的变量名就变成类对象的属性；而内嵌的`def`则会创建类对象的方法。

类和它们的不同之处在于，类的命名空间是Python继承的基础。在类或实例对象中找不到的所引用的属性，就会从继承树中的其他类获取。

例如，把简单的非函数的对象赋值给类属性，就会产生 ***数据属性*** ，由所有实例共享：

```python
>>> class SharedData:
        spam = 42         # Generates a class data attribute
>>> x = SharedData()      # Make two instances
>>> y = SharedData()
>>> x.spam, y.spam        # They inherit and share 'spam' (a.k.a. SharedData.spam)
(42, 42)
```

我们可以通过类名称修改类属性，或者通过实例或类引用类属性：

```python
>>> SharedData.spam = 99
>>> x.spam, y.spam, SharedData.spam
(99, 99, 99)
```

> 这与C++的静态数据成员的概念有些类似：也就是存储在类中的成员，与实例不相关。

通过实例而不是类来给变量名`spam`赋值，就会在该实例内创建或修改变量名，而不是在共享的类中：

```python
>>> x.spam = 88
>>> x.spam, y.spam, SharedData.spam
(88, 99, 99)
```

下面的例子，我们把相同的变量名存储在类对象（由类中的`data`赋值运算所创建）和实例对象中（由`__init__`中的`self.data`赋值运算所创建）：

```python
class MixedNames:                           # Define class
    data = 'spam'                           # Assign class attr
    def __init__(self, value):              # Assign method name
        self.data = value                   # Assign instance attr
    def display(self):
        print(self.data, MixedNames.data)   # Instance attr, class attr
```

结果就是`data`存在于2个地方：在实例对象内，以及在实例继承变量名的类中。
```python
>>> x = MixedNames(1)         # Make two instance objects
>>> y = MixedNames(2)         # Each has its own data
>>> x.display(); y.display()  # self.data differs, MixedNames.data is the same
1 spam
2 spam
```

利用这些技术把属性存储在不同对象内，我们可以决定属性的可见范围。附加在类上时，变量名是共享的；附加在实例上时，变量名是属于每个实例的数据，而不是共享的行为或数据。

### 29.2 方法

方法位于`class`语句的主体内，是由`def`语句建立的函数对象。从抽象的视角来看，方法替实例对象提供了要继承的行为。

方法和函数的唯一区别就是：方法的第一个参数（按惯例被称为`self`）总是接收调用方法的实例对象。如下所示：

```python
instance.method(args...)
```

Python会自动将以上形式的 *实例方法调用* 翻译成以下形式的 *类方法函数调用* ：

```python
class.method(instance, args...)
```

事实上，这两种调用形式在Python中都有效。

#### 例子

```python
class NextClass:                # Define class
    def printer(self, text):    # Define method
        self.message = text     # Change instance
        print(self.message)     # Access instance
```

```python
>>> x = NextClass()             # Make instance
>>> x.printer('instance call')  # Call its method
instance call
>>> x.message                   # Instance changed
'instance call'
```

方法能通过实例或类本身两种方法其中的任意一种进行调用。例如，可以通过类的名称调用`printer`，只要明确地传递了一个实例给`self`参数：

```python
>>> NextClass.printer(x, 'class call')   # Direct class call
class call
>>> x.message                            # Instance changed again
'class call'
```

实际上，在默认情况下，如果尝试不带任何实例调用方法时，就会引发错误：

```python
>>> NextClass.printer('bad call')
TypeError: unbound method printer() must be called with NextClass instance...
```

#### 调用超类构造函数

在构造时，Python会找出并且只调用一个`__init__`方法，所以，如果要保证子类的构造函数也会执行超类构造时的逻辑，一般都必须通过类明确地调用超类的`__init__`方法。

```python
class Super:
    def __init__(self, x):
    ...default code...

class Sub(Super):
    def __init__(self, x, y):
    Super.__init__(self, x)     # Run superclass __init__
    ...custom code...           # Do my init actions

I = Sub(1, 2)
```

#### 其他方法调用的可能性

***静态方法*** 可让你编写不预期第一参数为实例对象的方法。

***类方法*** 当调用的时候接受一个类而不是一个实例，并且它可以用来管理基于每个类的数据。

> Python 也有一个内置函数 `super` ，该函数允许更通用地回调一个超类的方法。

### 29.3 继承

`class`语句作为命名空间工具是Python变量名继承的基础。

在Python中，每当使用`object.attr`形式的表达式对对象进行点号运算时，就会发生继承，而且涉及了搜索属性定义树（一或多个命名空间）。

#### 属性树的构造

下图总结了命名空间树构造以及填入变量名的方式。通常来说：

- 实例属性是由对方法内`self`属性进行赋值运算而生成的。
- 类属性是通过`class`语句内的语句（赋值语句）而生成的。
- 超类的连接是通过`class`语句首行的括号内列出类而生成的。

![Figure 29-1](_static/images/Part_6_Classes_and_OOP.assets/1556719020836.png)


#### 继承方法的专有化
重新定义继承变量名的概念引出了各种专有化技术。例如，子类可以完全取代继承的属性，提供超类可以找到的属性，并且通过已覆盖的方法回调超类来扩展超类的方法。下面是如何进行扩展的例子：
```python
>>> class Super:
        def method(self):
            print('in Super.method')
>>> class Sub(Super):
        def method(self):                   # Override method
            print('starting Sub.method')    # Add actions here
            Super.method(self)              # Run default action
            print('ending Sub.method')
```

`Sub`在重新`Super`的方法时，又回调了`Super`中的方法，换句话说，`Sub.method`只是扩展了`Super.method`的行为，而不要完全取代了它：
```python
>>> x = Super()        # Make a Super instance
>>> x.method()         # Runs Super.method
in Super.method
>>> x = Sub()          # Make a Sub instance
>>> x.method()         # Runs Sub.method, calls Super.method
starting Sub.method
in Super.method
ending Sub.method
```

#### 类接口技术
扩展只是一种与超类接口的方式。下面所展示的`specialize.py`文件定义了多个类，示范了一些常用技巧：
- `Super`：定义一个`method`函数以及在子类中期待一个动作的`delegate`。
- `Inheritor`：没有提供任何新的变量名，因此会获得`Super`中定义的一切内容。
- `Replacer`：用自己的版本覆盖`Super`的`method`。
- `Extender`：覆盖并回调默认`method`，从而定制`Super`的`method`。
- `Provider`：实现`Super`的`delegate`方法预期的`action`方法。

```python
class Super:
    def method(self):
        print('in Super.method')                 # Default behavior
    def delegate(self):
        self.action()                            # Expected to be defined

class Inheritor(Super):                          # Inherit method verbatim
    pass

class Replacer(Super):                           # Replace method completely
    def method(self):
        print('in Replacer.method')

class Extender(Super):                           # Extend method behavior
    def method(self):
        print('starting Extender.method')
        Super.method(self)
        print('ending Extender.method')

class Provider(Super):                           # Fill in a required method
    def action(self):
        print('in Provider.action')

if __name__ == '__main__':
    for klass in (Inheritor, Replacer, Extender):
        print('\n' + klass.__name__ + '...')
        klass().method()
    print('\nProvider...')
    x = Provider()
    x.delegate()
```

以下是执行这个文件时的结果：
```python
% python specialize.py
Inheritor...
in Super.method
Replacer...
in Replacer.method
Extender...
starting Extender.method
in Super.method
ending Extender.method
Provider...
in Provider.action
```


#### 抽象超类
***抽象超类*** 是指类的部分行为默认是由其子类所提供的。如果预期的方法没有在子类中定义，当继承搜索失败时，Python会引发未定义变量名的异常。

类的编写者偶尔会使用`assert`语句引发内置的异常`NotImplementedError`，使得这种子类需求更为明显。下面是使用`assert`来实现抽象超类的实例：

```python
class Super:
    def delegate(self):
        self.action()
    def action(self):
        assert False, 'action must be defined!'  # If this version is called
```
```python
>>> X = Super()
>>> X.delegate()
AssertionError: action must be defined!
```

此外，有些类只在该类的不完整方法中直接使用`raise`语句产生`NotImplementedError`异常：

```python
class Super:
    def delegate(self):
        self.action()
    def action(self):
        raise NotImplementedError('action must be defined!')
```

```python
>>> X = Super()
>>> X.delegate()
NotImplementedError: action must be defined!
```

对于子类的实例，除非子类提供了预期实现的方法来重写超类中的默认方法，否则将得到异常：

```python
>>> class Sub(Super): pass
>>> X = Sub()
>>> X.delegate()
NotImplementedError: action must be defined!
>>> class Sub(Super):
        def action(self): print('spam')
>>> X = Sub()
>>> X.delegate()
spam
```


##### Python 3.X和2.6+ 中的抽象超类

在Python 3.X和Python 2.6+中，抽象超类需要由子类填充的方法，可以使用特殊的类语法来实现。

在Python 3.X，我们在一个`class`头部使用一个关键字参数，以及特殊的`@`装饰器语法：

```python
from abc import ABCMeta, abstractmethod
class Super(metaclass=ABCMeta):
    @abstractmethod
    def method(self, ...):
        pass
```

具有这样抽象方法的类不能产生实例，除非在类树的较低层级定义了该方法。例如，在Python 3.X中，与前一小节的例子等价的特殊语法如下：

```python
>>> from abc import ABCMeta, abstractmethod
>>>
>>> class Super(metaclass=ABCMeta):
        def delegate(self):
            self.action()
        @abstractmethod
        def action(self):
            pass
>>> X = Super()
TypeError: Can't instantiate abstract class Super with abstract methods action
>>> class Sub(Super): pass
>>> X = Sub()
TypeError: Can't instantiate abstract class Sub with abstract methods action
>>> class Sub(Super):
def action(self): print('spam')
>>> X = Sub()
>>> X.delegate()
spam
```

按照这种方式编写代码，带有一个抽象方法的类是不能继承的（即，我们不能通过它来创建实例），除非其所有的抽象方法都已经在子类中定义了。

### 29.4 命名空间：结论

#### 简单变量名

无点号的简单变量名遵循函数LEGB作用域法则。

- 赋值语句（`X = value`）：除非简单变量`X`被声明为`global`或`nonlocal`，否则简单变量名的赋值语句在当前作用域内，创建或改变变量名`X`。
- 引用（`X`）：按照LEGB作用域法则搜索变量名`X`。

#### 属性名称

点号的属性名指的是特定对象的属性，并且遵循模块和类的规则。就类和实例对象而言，引用规则增加了继承搜索这个流程。

- 赋值语句（`object.X = value`）：点号运算符在对象的命名空间内创建或修改属性名`X`。继承树的搜索只发生在属性引用时，而不是属性的赋值运算时。
- 引用（object.X）：对基于类的对象而言，会在对象内搜索属性`X`，然后是其上所有可读取的类（使用继承搜索流程）。对于不是基于类的对象而言（例如，模块），则是从对象中直接读取属性`X`。

#### Python命名空间的“禅”

文件`manynames.py`示范了本书中遇到的所有命名空间的概念和规则：

```python
# File manynames.py
X = 11                 # Global (module) name/attribute (X, or manynames.X)

def f():
    print(X)           # Access global X (11)

def g():
    X = 22             # Local (function) variable (X, hides module X)
    print(X)

class C:
    X = 33             # Class attribute (C.X)
    def m(self):
        X = 44         # Local variable in method (X)
        self.X = 55    # Instance attribute (instance.X)

if __name__ == '__main__':
    print(X)           # 11: module (a.k.a. manynames.X outside file)
    f()                # 11: global
    g()                # 22: local
    print(X)           # 11: module name unchanged
    obj = C()          # Make instance
    print(obj.X)       # 33: class name inherited by instance
    obj.m()            # Attach attribute name X to instance now
    print(obj.X)       # 55: instance
    print(C.X)         # 33: class (a.k.a. obj.X if no X in instance)
    #print(C.m.X)      # FAILS: only visible in method
    #print(g.X)        # FAILS: only visible in function
```

在另一个文件`otherfile.py`内导入`manynames.py`模块并读取其中定义的变量名：

```python
# otherfile.py
import manynames

X = 66
print(X)               # 66: the global here
print(manynames.X)     # 11: globals become attributes after imports

manynames.f()          # 11: manynames's X, not the one here!
manynames.g()          # 22: local in other file's function

print(manynames.C.X)   # 33: attribute of class in other module
I = manynames.C()
print(I.X)             # 33: still from class here
I.m()
print(I.X)             # 55: now from instance!
```

#### 命名空间字典

对象命名空间实际上是以字典的形式实现的，并且可以由内置属性`__dict__`查看。属性点号运算其实内部就是字典的索引运算，而属性继承其实就是搜索链接的字典。

首先我们定义一个超类和一个带方法的子类，而这些方法会将数据保存到实例中：

```python
>>> class Super:
        def hello(self):
            self.data1 = 'spam'
>>> class Sub(Super):
        def hola(self):
            self.data2 = 'eggs'
```

实例中有个`__class__`属性链接到它的类；而类有个`__bases__`属性，它是一个元组，其中包含了通往更高的超类的链接。

```python
>>> X = Sub()
>>> X.__dict__ # Instance namespace dict
{}
>>> X.__class__ # Class of instance
<class '__main__.Sub'>
>>> Sub.__bases__ # Superclasses of class
(<class '__main__.Super'>,)
>>> Super.__bases__ # () empty tuple in Python 2.X
(<class 'object'>,)
```

因为属性实际上是Python的字典键，所以既可以通过属性点号运算，也可以通过命名空间字典键索引运算，来对属性进行读取和赋值：

```python
>>> X.data1, X.__dict__['data1']
('spam', 'spam')
>>> X.data3 = 'toast'
>>> X.__dict__
{'data2': 'eggs', 'data3': 'toast', 'data1': 'spam'}
>>> X.__dict__['data3'] = 'ham'
>>> X.data3
'ham'
```

但属性点号运算会执行继承搜索，所以可以存取命名空间字典键索引运算无法读取的属性。

最后，内置函数`dir`能用在任何带有属性的对象，`dir(object)`类似于`object.__dict__.keys()`调用：

```python
>>> X = Sub()
>>> X.hello()
>>> X.hola()
>>> X.data3 = 'ham'
>>> X.__dict__
{'data1': 'spam', 'data2': 'eggs', 'data3': 'ham'}
>>> dir(X)
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'data1', 'data2', 'data3', 'hello', 'hola']
>>> dir(Sub)
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'hello', 'hola']
>>> dir(Super)
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'hello']
```


#### 命名空间链接

实例和类的特殊属性`__class__`和`__bases__`可以在程序代码内查看继承层级。例如，可以用来显示类树：

```python
#!python
"""
classtree.py: Climb inheritance trees using namespace links,
displaying higher superclasses with indentation for height
"""
def classtree(cls, indent):
    print('.' * indent + cls.__name__) # Print class name here
    for supercls in cls.__bases__:     # 递归到所有超类
        classtree(supercls, indent+3)  # May visit super > once

def instancetree(inst):
    print('Tree of %s' % inst)         # Show instance
    classtree(inst.__class__, 3)       # Climb to its class

def selftest():
    class A: pass
    class B(A): pass
    class C(A): pass
    class D(B,C): pass
    class E: pass
    class F(D,E): pass
    instancetree(B())
    instancetree(F())

if __name__ == '__main__': 
    selftest()
```

在Python 3.X中运行时，包含了隐式的`object`超类的树会自动被添加到独立的类顶端。因为在Python 3.X中，所有的类都是新式类。

```python
C:\code> c:\python33\python classtree.py
Tree of <__main__.selftest.<locals>.B object at 0x00000000029216A0>
...B
......A
.........object
Tree of <__main__.selftest.<locals>.F object at 0x00000000029216A0>
...F
......D
.........B
............A
...............object
.........C
............A
...............object
......E
.........object
```

上例中，由点号所表示的缩进表示类树的高度。

### 29.5 回顾文档字符串

***文档字符串*** 是出现在各种结构顶部的字符串常量，由Python在相应对象的`__doc__`属性自动保存。它适用于模块文件、函数定义，以及类和方法。

如下的文件`docstr.py`展示了文档字符串可以在代码中出现的位置，其中所有文档字符串都可以是三重引号的块：

```python
"I am: docstr.__doc__"

def func(args):
    "I am: docstr.func.__doc__"
    pass

class spam:
    "I am: spam.__doc__ or docstr.spam.__doc__ or self.__doc__"
    def method(self):
        "I am: spam.method.__doc__ or self.method.__doc__"
        print(self.__doc__)
        print(self.method.__doc__)
```

文档字符串可以在运行时保持，所以可以在运行时使用其`__doc__`属性来获取文档：
```python
>>> import docstr
>>> docstr.__doc__
'I am: docstr.__doc__'
>>> docstr.func.__doc__
'I am: docstr.func.__doc__'
>>> docstr.spam.__doc__
'I am: spam.__doc__ or docstr.spam.__doc__ or self.__doc__'
>>> docstr.spam.method.__doc__
'I am: spam.method.__doc__ or self.method.__doc__'
>>> x = docstr.spam()
>>> x.method()
I am: spam.__doc__ or docstr.spam.__doc__ or self.__doc__
I am: spam.method.__doc__ or self.method.__doc__
```




### 29.6 类VS模块

模块和类的区别：

- 模块：
  - 实现 数据和逻辑的包。
  - 通过Python文件和其他语言的扩展来创建。
  - 通过导入来使用。
  - Python编程结构的顶层（top-level）形式。
- 类：
  -  实现新的全部特性（full-featured）的对象。
  -  由`class`语句创建。
  -  通过调用来使用。
  -  总是位于一个模块中。
  -  支持运算符重载、多实例生成和继承。





---


## 第30章 运算符重载

### 30.1 基础知识

实际上，***运算符重载*** 只是意味着在类方法中**拦截**内置的操作，即，当类的实例出现在内置操作中，Python自动调用你的方法，并将该方法的返回值变成响应操作的结果。

- 运算符重载让类拦截常规的Python运算。
- 类可重载所有Python表达式运算符。
- 类也可重载打印、函数调用、属性点号运算等内置运算。
- 重载使类实例的行为像内置类型。
- 重载是通过提供特殊名称的类方法来实现的。

运算符重载方法并非必需的，并且通常也不是默认的；如果你没有编写或继承一个运算符重载方法，只是意味着你的类不支持相应的操作。

#### 构造函数和表达式：`__init__`和`__sub__`

如下文件`number.py`中的`Number`类提供了一个方法`__init__`来拦截实例的构造函数，`__sub__`则捕获减法表达式：

```python
# File number.py
class Number:
    def __init__(self, start):             # On Number(start)
        self.data = start
    def __sub__(self, other):              # On instance - other
        return Number(self.data - other)   # Result is a new instance
```

```python
>>> from number import Number              # Fetch class from module
>>> X = Number(5)                          # Number.__init__(X, 5)
>>> Y = X - 2                              # Number.__sub__(X, 2)
>>> Y.data                                 # Y is new Number instance
3
```

#### 常见的运算符重载方法

所有重载方法的名称前后都有两个下划线字符，以便把同类中定义的变量名区别开来。

特殊方法名称和表达式或运算的映射关系，是由Python语言预先定义好的。例如，方法名`__add__`按照Python语言的定义，无论`__add__`方法的代码实际在做什么，总是对应到了表达式`+`。

下表中列出了一些最常用的重载方法：

|   Method   |   Implements   |   Called for   |
| :--- | :--- | :--- |
|   `__init__`   |   Constructor   |   Object creation: `X = Class(args)`   |
|   `__del__`   |   Destructor   |   Object reclamation of `X`   |
|   `__add__`   |   Operator `+`   |   `X + Y`, `X += Y` if no `__iadd__`   |
|   `__or__`   |   Operator `|` (bitwise OR)   |   `X | Y`,` X |= Y` if no `__ior__`   |
|   `__repr__`, `__str__`   |   Printing, conversions   |   `print(X)`,` repr(X)`, `str(X)`   |
|   `__call__`   |   Function calls    |   `X(*args, **kargs)`   |
|   `__getattr__`   |   Attribute fetch   |   `X.undefined`   |
|   `__setattr__`   |   Attribute assignment   |   `X.any = value`   |
|   `__delattr__`   |   Attribute deletion   |   `del X.any`   |
|   `__getattribute__`   |   Attribute fetch   |   `X.any`   |
|   `__getitem__`   |   Indexing, slicing, iteration   |   `X[key]`, `X[i:j]`, for loops and other iterations if no `__iter__`   |
|   `__setitem__`   |   Index and slice assignment   |   `X[key] = value`, `X[i:j] = iterable`   |
|   `__delitem__`   |   Index and slice deletion   |   `del X[key]`, `del X[i:j]`   |
|   `__len__`   |   Length   |   `len(X)`, truth tests if no `__bool__`   |
|   `__bool__`   |   Boolean tests   |   `bool(X)`, truth tests (named `__nonzero__` in 2.X)   |
|   `__lt__`, `__gt__`,`__le__`, `__ge__`,`__eq__`, `__ne__`   |   Comparisons   |   `X < Y`, `X > Y`, `X <= Y`, `X >= Y`, `X == Y`, `X != Y` (or else `__cmp__` in 2.X only)   |
|   `__radd__`   |   Right-side operators   |   `Other + X`   |
|   `__iadd__`   |   In-place augmented operators   |   `X += Y` (or else `__add__`)   |
|   `__iter__`, `__next__`   |   Iteration contexts   |   `I=iter(X)`, `next(I)`; for loops, in if no `__contains__`, all comprehensions, `map(F,X)`, others (`__next__` is named next in 2.X)   |
|   `__contains__`   |   Membership test   |   `item in X` (any iterable)   |
|   `__index__`   |   Integer value   |   `hex(X)`, `bin(X)`, `oct(X)`, `O[X]`, `O[X:]` (replaces 2.X `__oct__`, `__hex__`)   |
|   `__enter__`, `__exit__`   |   Context manager (Chapter 34)   |   `with obj as var:`   |
|   `__get__`, `__set__`,`__delete__`   |   Descriptor attributes (Chapter 38)   |`X.attr`, `X.attr = value`, `del X.attr`|
|   `__new__`   |   Creation (Chapter 40)   |   Object creation, before `__init__`   |



### 30.2 索引和分片：`__getitem__`和`__setitem__`

对于实例的索引运算，如果在类中定义了或继承了`__getitem__`，则会自动调用`__getitem__`。当实例`X`出现在`X[i]`这样的索引运算中时，Python会调用这个实例的`__getitem__`方法（如果有的话），把`X`作为第一个参数传递，并且方括号内的索引值传递给第二个参数。例如，下面类将返回索引值的平方：

```python
>>> class Indexer:
        def __getitem__(self, index):
            return index ** 2
>>> X = Indexer()
>>> X[2]                  # X[i] calls X.__getitem__(i)
4
>>> for i in range(5):
print(X[i], end=' ')      # Runs __getitem__(X, i) each time
0 1 4 9 16
```

#### 拦截分片

除了索引，对于分片表达式也会调用`__getitem__`。例如，对内置列表的分片：

```python
>>> L = [5, 6, 7, 8, 9]
>>> L[2:4] # Slice with slice syntax: 2..(4-1)
[7, 8]
>>> L[1:]
[6, 7, 8, 9]
>>> L[:-1]
[5, 6, 7, 8]
>>> L[::2]
[5, 7, 9]
```

实际上，我们可以手动传递一个`slice`对象来进行分片。`slice`对象有3个属性：`start`,`stop`,`step`；其中任何一个都可以为`None`，如果为`None`则会被忽略。

```python
>>> L[slice(2, 4)] # Slice with slice objects
[7, 8]
>>> L[slice(1, None)]
[6, 7, 8, 9]
>>> L[slice(None, −1)]
[5, 6, 7, 8]
>>> L[slice(None, None, 2)]
[5, 7, 9]
```

对于具有`__getitem__`的类，`__getitem__`方法既针对基本索引调用，又针对分片（带有一个分片对象）调用。

如下类`Indexer`将会处理分片：

```python
>>> class Indexer:
        data = [5, 6, 7, 8, 9]
        def __getitem__(self, index): # Called for index or slice
            print('getitem:', index)
            return self.data[index] # Perform index or slice
```

当针对索引调用时，参数像前面一样是一个整数：
```python
>>> X = Indexer()
>>> X[0] # Indexing sends __getitem__ an integer
getitem: 0
5
>>> X[1]
getitem: 1
6
>>> X[-1]
getitem: −1
9
```

当针对分片调用时，方法接收一个分片对象：
```python
>>> X[2:4] # Slicing sends __getitem__ a slice object
getitem: slice(2, 4, None)
[7, 8]
>>> X[1:]
getitem: slice(1, None, None)
[6, 7, 8, 9]
>>> X[:-1]
getitem: slice(None, −1, None)
[5, 6, 7, 8]
>>> X[::2]
getitem: slice(None, None, 2)
[5, 7, 9]
```

如果需要，`__getitem__`可以测试它参数的类型，并取出`slice`对象边界。
```python
>>> class Indexer:
        def __getitem__(self, index):
            if isinstance(index, int): # Test usage mode
                print('indexing', index)
            else:
                print('slicing', index.start, index.stop, index.step)
>>> X = Indexer()
>>> X[99]
indexing 99
>>> X[1:99:2]
slicing 1 99 2
>>> X[1:]
slicing 1 None None
```

与`__getitem__`类似，`__setitem__`方法拦截索引和分片赋值。在Python 3.X中，它为后者接收一个`slice`对象，这个`slice`对象可能被传递到另一个索引赋值中，或者也可能以同样的方式直接被使用：
```python
class IndexSetter:
    def __setitem__(self, index, value): # Intercept index or slice assignment
        ...
        self.data[index] = value # Assign index or slice
```

#### Python 3.X的`__index__`不是索引
Python 3.X的`__index__`不是为了索引拦截（index interception）。当需要时，或者实例被转换数字字符串的内置函数（built-ins）使用时，这个方法返回一个整数值。其实，这个方法如果被命名为`__asindex__`，可能会更贴切一些。

```python
>>> class C:
        def __index__(self):
            return 255
>>> X = C()
>>> hex(X) # Integer value
'0xff'
>>> bin(X)
'0b11111111'
>>> oct(X)
'0o377'
```


### 30.3 索引迭代：`__getitem__`

for语句的作用是从0到更大的索引值，重复对序列进行索引运算，直到检测到越界的异常`IndexError`。因此，`__getitem__`也可以作为Python中的一种*退而求其次的（fallback）*重载迭代方式。

如果`__getitem__`被定义，for循环每趟循环都会连续地以更高的偏移量（offsets）来调用类的`__getitem__`：

```python
>>> class StepperIndex:
        def __getitem__(self, i):
            return self.data[i]
>>> X = StepperIndex() # X is a StepperIndex object
>>> X.data = "Spam"
>>>
>>> X[1] # Indexing calls __getitem__
'p'
>>> for item in X: # for loops call __getitem__
        print(item, end=' ') # for indexes items 0..N
S p a m
```

Python中所有的迭代环境（iteration contexts），例如，成员关系检测`in`，列表解析，内置函数`map`、列表和元组赋值运算以及类型构造方法也会自动调用`__getitem__`（如果定义了的话）：

```python
>>> 'p' in X # All call __getitem__ too
True
>>> [c for c in X] # List comprehension
['S', 'p', 'a', 'm']
>>> list(map(str.upper, X)) # map calls (use list() in 3.X)
['S', 'P', 'A', 'M']
>>> (a, b, c, d) = X # Sequence assignments
>>> a, c, d
('S', 'a', 'm')
>>> list(X), tuple(X), ''.join(X) # And so on...
(['S', 'p', 'a', 'm'], ('S', 'p', 'a', 'm'), 'Spam')
>>> X
<__main__.StepperIndex object at 0x000000000297B630>
```



### 30.4 可迭代对象：`__iter__`和`__next__`

Python中所有的迭代环境（iteration contexts）都会先尝试`__iter__`方法，仅当没有找到`__iter__`方法时，才退而尝试`__getitem__`方法，并反复地通过偏移量（offsets）来索引，直到`IndexError`异常被触发。也就是说，可迭代对象接口被给予优先级并会首先被尝试，只有在对象不支持迭代协议的时候，才会尝试索引运算。一般来讲，我们应该优先使用`__iter__`，它能够比`__getitem__`更好地支持一般的迭代环境。

从技术角度来讲，迭代环境的工作机制是通过将一个可迭代对象作为参数传递给内置函数`iter`来调用可迭代对象的`__iter__`方法，这个`__iter__`方法应该返回一个**迭代器对象**。如果成功返回了迭代器对象，Python就会重复调用这个迭代器对象的`__next__`方法，直到`StopIteration`异常被引发。内置函数`next`也可用来作为手动迭代的一种便捷方式，即，`next(I)`和`I.__next__()`相同。

#### 用户自定义迭代器

在`__iter__`机制中，类就是通过迭代器协议来实现用户定义的迭代器的。

例如，下面的文件`iters.py`定义了用户定义的迭代器类来生成平方值：

```python
# File squares.py
class Squares:
    def __init__(self, start, stop): # Save state when created
        self.value = start - 1
        self.stop = stop
    def __iter__(self):             # Get iterator object on iter
        return self
    def __next__(self):             # Return a square on each iteration
        if self.value == self.stop: # Also called by next built-in
            raise StopIteration     # 引发StopIteration异常
        self.value += 1
        return self.value ** 2
```

其中，迭代器对象就是实例`self`自身，因为类实现了`__next__`方法。

当导入后，它的实例出现在迭代器环境（iteration contexts）中就会像内建的（built-ins）一样：

```python
% python
>>> from squares import Squares
>>> for i in Squares(1, 5):        # for calls iter, which calls __iter__
        print(i, end=' ')          # Each iteration calls __next__
1 4 9 16 25
```

手动迭代`Squares`类实例：

```python
>>> X = Squares(1, 5)  # Iterate manually: what loops do
>>> I = iter(X)        # iter calls __iter__
>>> next(I)            # next calls __next__ (in 3.X)
1
>>> next(I)
4
...more omitted...
>>> next(I)
25
>>> next(I)            # Can catch this in try statement
StopIteration
```

同时，因为`Squares`类既实现了`__iter__`也实现了`__next__`，所以，它的实例的迭代器就是其自身，可以直接迭代其自身：

```python
>>> X = Squares(1,5)
>>> I = iter(X)
>>> X is I
True
>>> id(X)
140309082954832
>>> id(I)
140309082954832
>>> next(X)
1
>>> next(X)
4
>>> next(X)
9
>>> next(X)
16
>>> next(X)
25
>>> next(X)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 9, in __next__
StopIteration
```

因为`__iter__`会在调用过程中明确地保留迭代状态信息，所以比`__getitem__`简单地通过索引偏移量实现迭代具有更好的通用性。

##### Single versus multiple scans
和`__getitem__`不同的是，`__iter__`只循环一次，而不是循环多次。所以每次开始新的迭代，都得创建一个新的迭代器对象。例如，`Square`类只循环一次，循环之后就变为空：

```python
>>> X = Squares(1, 5)          # Make an iterable with state
>>> [n for n in X]             # Exhausts items: __iter__ returns self
[1, 4, 9, 16, 25]
>>> [n for n in X]             # Now it's empty: __iter__ returns same self
[]
>>> [n for n in Squares(1, 5)] # Make a new iterable object
[1, 4, 9, 16, 25]
>>> list(Squares(1, 3))        # A new object for each new __iter__ call
[1, 4, 9]
```

通过为每次迭代创建一个新实例，您将获得迭代状态的新副本：
```python
>>> 36 in Squares(1, 10)      # Other iteration contexts
True
>>> a, b, c = Squares(1, 3)   # Each calls __iter__ and then __next__
>>> a, b, c
(1, 4, 9)
>>> ':'.join(map(str, Squares(1, 5)))
'1:4:9:16:25'
```

就像单扫描（single-scan）内置函数（如`map`），转换为列表也支持多次扫描（multiple scans），但会增加时间和空间性能成本：
```python
>>> X = Squares(1, 5)
>>> tuple(X), tuple(X)         # Iterator exhausted in second tuple()
((1, 4, 9, 16, 25), ())
>>> X = list(Squares(1, 5))
>>> tuple(X), tuple(X)
((1, 4, 9, 16, 25), (1, 4, 9, 16, 25))
```

##### Classes versus generators

如果用生成器函数或生成器表达式编写这个例子，可能会更简单一些。和类不同的是，生成器函数会自动在迭代中存储其状态。

生成器函数实现：
```python
>>> def gsquares(start, stop):
        for i in range(start, stop + 1):
            yield i ** 2
>>> for i in gsquares(1, 5):
        print(i, end=' ')
1 4 9 16 25
```

生成器表达式实现：
```python
>>> for i in (x ** 2 for x in range(1, 6)):
        print(i, end=' ')
1 4 9 16 25
```

#### 多迭代器的对象

迭代器对象可以定义成一个独立的类，有其自己的状态信息，从而能够支持相同数据的多个迭代。要达到多个迭代器的效果，`__iter__`只需为迭代器定义新的状态对象，而不是为每个迭代器请求返回`self`。

例如，下面的`SkipObject`类定义了一个在迭代时跳过每一项元素的可迭代对象。因为迭代器对象会在每次迭代时都重新创建，所以能够支持多个处于激活状态下的循环。

```python
#!python3
# File skipper.py
class SkipObject:
    def __init__(self, wrapped):             # Save item to be used
        self.wrapped = wrapped
    def __iter__(self):
        return SkipIterator(self.wrapped)    # New iterator each time

class SkipIterator:
    def __init__(self, wrapped):
        self.wrapped = wrapped               # Iterator state information
        self.offset = 0
    def __next__(self):
        if self.offset >= len(self.wrapped): # Terminate iterations
            raise StopIteration
        else:
            item = self.wrapped[self.offset] # else return and skip
            self.offset += 2
            return item

if __name__ == '__main__':
    alpha = 'abcdef'
    skipper = SkipObject(alpha)      # Make container object
    I = iter(skipper)                # Make an iterator on it
    print(next(I), next(I), next(I)) # Visit offsets 0, 2, 4
    
    for x in skipper:                # for calls __iter__ automatically
        for y in skipper:                # Nested fors call __iter__ again each time
            print(x + y, end=' ')            # Each iterator has its own state, offset
```

运行时，这个例子工作起来就像是对内置字符串进行嵌套循环一样，因为每个循环都会获得独立的迭代器对象来记录自己的状态信息，所以每个激活状态下的循环都有自己在字符串中的位置：

```python
% python skipper.py
a c e
aa ac ae ca cc ce ea ec ee
```

##### Classes versus slices

我可以使用内置工具来达到类似的效果。例如，用第三参数边界值进行分片运算来跳过元素：

```python
>>> S = 'abcdef'
>>> for x in S[::2]:             # 每个分片表达式偶会一次性把结果列表存储到内存中
        for y in S[::2]:         # New objects on each iteration
            print(x + y, end=' ')
aa ac ae ca cc ce ea ec ee
```

但上例中这样做会使每个分片表达式偶会一次性把结果列表存储到内存中，而不是像迭代器那样一次产生一个值，所以会比迭代器消耗更多的内存。为了接近于使用迭代器的效果，我们需要事先创建一个已分片的对象来步进：

```python
>>> S = 'abcdef'
>>> S = S[::2]
>>> S
'ace'
>>> for x in S:
        for y in S: # Same object, new iterators
            print(x + y, end=' ')
aa ac ae ca cc ce ea ec ee
```



#### Coding Alternative：`__iter__` plus `yield`

在某些应用中，通过将`__iter__`方法和`yield`生成器函数语句相结合，可以最小化用户定义的可迭代对象的编码要求。因为生成器函数自动地保存本地变量的状态，并创建所需的迭代器方法。这补足（complement）了我们从**类**所获得的状态保持（state retention）和其他实用性（utility）。

包含`yield`语句的任何函数都将被转换为生成器函数。当被调用时，它将返回一个新的生成器对象。这个返回的生成器对象具有本地作用域和代码位置的自动状态保持（retention），以及一个自动被创建的`__iter__`方法和`__next__`方法（在Python 2.X中则是`next`）。其中，`__iter__`方法简单地返回生成器对象其自身；而`__next__`方法则启动该生成器函数，或者从停止的位置恢复它。

```python
>>> def gen(x):
        for i in range(x): yield i ** 2
>>> G = gen(5)             # Create a generator with __iter__ and __next__
>>> G.__iter__() == G      # Both methods exist on the same object
True
>>> I = iter(G)            # Runs __iter__: generator returns itself
>>> next(I), next(I)       # Runs __next__ (next in 2.X)
(0, 1)
>>> list(gen(5))           # Iteration contexts automatically run iter and next
[0, 1, 4, 9, 16]
```

即使带有`yield`的生成器函数恰好也名为`__iter__`，这也依然可行。任何时候，只要被一个迭代环境工具调用，这样的方法就将返回一个带有`__next__`方法的新生成器对象。这也使得作为方法而被编码在类中的生成器函数可以访问保存在实例属性和本地作用域变量中的状态。

例如，下面的类等价于之前`squares.py`可迭代对象`Squares`：

```python
# File squares_yield.py
class Squares:                       # __iter__ + yield generator
    def __init__(self, start, stop): # __next__ is automatic/implied
        self.start = start
        self.stop = stop
    def __iter__(self):
        for value in range(self.start, self.stop + 1):
            yield value ** 2
```

就像之前一样，`for`循环和其他迭代工具可以自动迭代这个类的实例：

```python
% python
>>> from squares_yield import Squares
>>> for i in Squares(1, 5): print(i, end=' ')
1 4 9 16 25
```

调用结果对象的`next`接口生成按需的结果：

```python
>>> S = Squares(1, 5)     # Runs __init__: class saves instance state
>>> S
<squares_yield.Squares object at 0x000000000294B630>
>>> I = iter(S)           # Runs __iter__: returns a generator
>>> I
<generator object __iter__ at 0x00000000029A8CF0>
>>> next(I)
1
>>> next(I)               # Runs generator's __next__
4
...etc...
>>> next(I)               # Generator has both instance and local scope state
StopIteration
```

我们可以将这个生成器方法命名为其他名字而不是`__iter__`，以此来手动迭代，例如`Squares(1,5).gen`。使用被迭代工具自动调用的方法名`__iter__`将直接跳过手动属性获取和调用步骤：

```python
class Squares:            # Non __iter__ equivalent (squares_manual.py)
    def __init__(...):
        ...
    def gen(self):
        for value in range(self.start, self.stop + 1):
            yield value ** 2
% python
>>> from squares_manual import Squares
>>> for i in Squares(1, 5).gen(): print(i, end=' ')
...same results...
>>> S = Squares(1, 5)
>>> I = iter(S.gen())    # Call generator manually for iterable/iterator
>>> next(I)
...same results...
```

但是，将生成器编码为`__iter__`会切断代码中的中间人。两种方案最终都会为每次迭代创建一个新的生成器对象：

- 有`__iter__`，则迭代触发`__iter__`，这将返回一个具有`__next__`的新生成器。
- 没有`__iter__`，则你的代码被调用，以创建一个生成器，这将返回生成器本身以获得其`__iter__`方法。

对比之前`squares.py`中的带有`__next__`的版本，你就会发现，新的`squares_yield.py`版本比之前少了4行。在某种意义上，这种方案减少了类编码要求，很像是第17章中的闭包函数（closure functions）。但是在这个例子中，也使用了函数式和OOP技术的组合，而不是类的替代方案。例如，生成器方法仍然利用`self`属性。

##### Multiple iterators with yield

除了代码简洁以外，前一部分基于`__iter__`和`yield`组合的用户自定义可迭代对象还有一个重要的好处：它也支持多个激活的迭代器（multiple active iterators）。

这自然是因为每次调用`__iter__`都是对生成器函数的调用，该函数返回一个新生成器，该生成器具有自己的本地作用域副本以保持状态：

```python
% python
>>> from squares_yield import Squares # Using the __iter__/yield Squares
>>> S = Squares(1, 5)
>>> I = iter(S)
>>> next(I); next(I)
1
4
>>> J = iter(S)                       # With yield, multiple iterators automatic
>>> next(J)
1
>>> next(I)                           # I is independent of J: own local state
9
```

即使生成器函数是单扫描（single-scan）可迭代对象，在迭代环境中对`__iter__`的隐式调用也会使新的生成器支持新的独立扫描：

```python
>>> S = Squares(1, 3)
>>> for i in S: # Each for calls __iter__
>>> for j in S:
>>> print('%s:%s' % (i, j), end=' ')
>>> 1:1 1:4 1:9 4:1 4:4 4:9 9:1 9:4 9:9
```

在不使用`yield`的情况下要做同样的事需要一个额外的类来显式地和手动地存储迭代器状态：
```python
# File squares_nonyield.py
class Squares:
    def __init__(self, start, stop): # Non-yield generator
        self.start = start           # Multiscans: extra object
        self.stop = stop
    def __iter__(self):
        return SquaresIter(self.start, self.stop)

class SquaresIter:
    def __init__(self, start, stop):
        self.value = start - 1
        self.stop = stop
    def __next__(self):
        if self.value == self.stop:
            raise StopIteration
        self.value += 1
        return self.value ** 2
```

这和使用`yield`的多扫描（mlti-scan）版本一样奏效，但多了很多显式的代码：
```python
% python
>>> from squares_nonyield import Squares
>>> for i in Squares(1, 5): print(i, end=' ')
1 4 9 16 25
>>>
>>> S = Squares(1, 5)
>>> I = iter(S)
>>> next(I); next(I)
1
4
>>> J = iter(S)            # Multiple iterators without yield
>>> next(J)
1
>>> next(I)
9
>>> S = Squares(1, 3)
>>> for i in S:           # Each for calls __iter___
        for j in S:
            print('%s:%s' % (i, j), end=' ')
1:1 1:4 1:9 4:1 4:4 4:9 9:1 9:4 9:9
```

最后，基于生成器（generator-based）的方法同样会移除之前`skipper.py`例子中额外的迭代器类。因为生成器的自动方法和本地变量状态保持，使得代码从原来的16行缩短到了9行：
```python
# File skipper_yield.py
class SkipObject:                  # Another __iter__ + yield generator
    def __init__(self, wrapped):   # Instance scope retained normally
        self.wrapped = wrapped     # Local scope state saved auto
    def __iter__(self):
        offset = 0
        while offset < len(self.wrapped):
            item = self.wrapped[offset]
            offset += 2
            yield item
```

这和非`yield`多扫描（multiscan）版本一样地奏效，但是少了很多显式代码：
```python
% python
>>> from skipper_yield import SkipObject
>>> skipper = SkipObject('abcdef')
>>> I = iter(skipper)
>>> next(I); next(I); next(I)
'a'
'c'
'e'
>>> for x in skipper: # Each for calls __iter__: new auto generator
        for y in skipper:
            print(x + y, end=' ')
aa ac ae ca cc ce ea ec ee
```


### 30.5 成员关系 `__contains__`、`__iter__`和`__getitem__`

运算符重载往往是**多个层级**的：类可以提供特定的方法，或者用作退而求其次的选项的更通用的替代方案：

- Python 2.X中使用`__lt__`这样的特殊方法来表示少于比较，或者使用通用的`__cmp__`。Python 3.X只使用特殊的方法，而不是`__cmp__`。
- 类似地，布尔测试先尝试一个特定的`__bool__`（以给出一个明确的`True`/`False`结果），并且，如果没有它，将会退而求其次到更通用的`__len__`（一个非零的长度意味着`True`）。

在迭代领域，类通常把`in`成员关系运算符实现为一个迭代，使用`__iter__`方法或`__getitem__`方法。要支持更加特定的成员关系，类可以编写一个`__contains__`方法。当它们都出现的时候，`__contains__`优先于`__iter__`，`__iter__`优先于`__getitem__`。

如下的类编写了3个方法和测试成员关系以及应用于一个实例的各种迭代环境。调用时，其方法会打印出跟踪消息：

```python
# File contains.py
from __future__ import print_function # 2.X/3.X compatibility

class Iters:
    def __init__(self, value):
        self.data = value
        
    def __getitem__(self, i):         # Fallback for iteration
        print('get[%s]:' % i, end='') # Also for index, slice
        return self.data[i]
    
    def __iter__(self):               # Preferred for iteration
        print('iter=> ', end='')      # Allows only one active iterator
        self.ix = 0
        return self
    
    def __next__(self):
        print('next:', end='')
        if self.ix == len(self.data): raise StopIteration
        item = self.data[self.ix]
        self.ix += 1
        return item
    
    def __contains__(self, x):        # Preferred for 'in'
        print('contains: ', end='')
        return x in self.data
    
    next = __next__                   # 2.X/3.X compatibility

if __name__ == '__main__':
    X = Iters([1, 2, 3, 4, 5])     # Make instance
    print(3 in X)                  # Membership
    for i in X:                    # for loops
        print(i, end=' | ')
        
    print()
    print([i ** 2 for i in X])     # Other iteration contexts
    print( list(map(bin, X)) )
    
    I = iter(X)                    # Manual iteration (what other contexts do)
    while True:
        try:
            print(next(I), end=' @ ')
        except StopIteration:
            break
```

上例中，类有一个支持多扫描的`__iter__`方法，但在任意时间点，只能有一个激活的扫描。因为每次迭代尝试都会重置扫描指针。下面是在迭代方法中使用`yield`实现的支持多扫描的，并具有更少代码量的类`Iters`：

```python
# contains_yield.py
class Iters:
    def __init__(self, value):
        self.data = value
        
    def __getitem__(self, i):          # Fallback for iteration
        print('get[%s]:' % i, end='')  # Also for index, slice
        return self.data[i]
    
    def __iter__(self):                # Preferred for iteration
        print('iter=> next:', end='')  # Allows multiple active iterators
        for x in self.data:            # no __next__ to alias to next
            yield x
            print('next:', end='')
            
    def __contains__(self, x):         # Preferred for 'in'
        print('contains: ', end='')
        return x in self.data
```

对于Python 3.X和2.X，当以上任一版本的文件运行时，特定的`__contains__`方法拦截成员关系；通用的`__iter__`捕获其他迭代环境以至`__next__`被重复地调用（不管是显式编码的还是通过`yield`隐式编码的）；而`__getitem__`不会被调用。其输出如下所示：

```python
contains: True          # 成员关系in运算会优先调用__contains__
iter=> next:1 | next:2 | next:3 | next:4 | next:5 | next:
iter=> next:next:next:next:next:next:[1, 4, 9, 16, 25]
iter=> next:next:next:next:next:next:['0b1', '0b10', '0b11', '0b100', '0b101']
iter=> next:1 @ next:2 @ next:3 @ next:4 @ next:5 @ next:
```

如果我们将`__contains__`方法注释掉，成员关系现在被路由到了通用的`__iter__`：

```python
iter=> next:next:next:True     # 退而求其次，成员关系被__iter__拦截,并开始迭代元素，以检测成员关系
iter=> next:1 | next:2 | next:3 | next:4 | next:5 | next:
iter=> next:next:next:next:next:next:[1, 4, 9, 16, 25]
iter=> next:next:next:next:next:next:['0b1', '0b10', '0b11', '0b100', '0b101']
iter=> next:1 @ next:2 @ next:3 @ next:4 @ next:5 @ next:
```

最后，如果`__contains__`和`__iter__`都注释掉，则索引`__getitem__`替代方法会被调用，其输出如下：

```python
get[0]:get[1]:get[2]:True   # 退而求其次，成员关系被__getitem__拦截，并开始索引遍历元素，以检测成员关系
get[0]:1 | get[1]:2 | get[2]:3 | get[3]:4 | get[4]:5 | get[5]:
get[0]:get[1]:get[2]:get[3]:get[4]:get[5]:[1, 4, 9, 16, 25]
get[0]:get[1]:get[2]:get[3]:get[4]:get[5]:['0b1', '0b10', '0b11', '0b100','0b101']
get[0]:1 @ get[1]:2 @ get[2]:3 @ get[3]:4 @ get[4]:5 @ get[5]:
```

正如我们所看到的，`__getitem__`方法更加通用，除了迭代，它还拦截显示索引和分片：

```python
>>> from contains import Iters
>>> X = Iters('spam')              # Indexing
>>> X[0]                           # __getitem__(0)
get[0]:'s'
>>> 'spam'[1:]                     # Slice syntax
'pam'
>>> 'spam'[slice(1, None)]         # Slice object
'pam'
>>> X[1:]                          # __getitem__(slice(..))
get[slice(1, None, None)]:'pam'
>>> X[:-1]
get[slice(None, −1, None)]:'spa'
>>> list(X)                        # And iteration too!
iter=> next:next:next:next:next:['s', 'p', 'a', 'm']
```



### 30.6 属性访问：`__getattr__`和`__setattr__`
在Python中，类也可以拦截（intercept）基本属性访问。对于一个由类创建的对象，点号运算符表达式`object.attribute`也可以由代码实现，用于引用、赋值和删除上下文（contexts）。

#### 属性引用

`__getattr__`方法会拦截属性引用。

当我们对属性名称和实例进行点号运算时，如果Python无法通过继承树搜索流程找到这个属性，则`__getattr__`方法会作为钩子被调用，以拦截和响应属性点号运算。

`__getattr__`的第一个参数为`self`，第二个参数为属性名称的字符串；而该方法的返回值将被作为属性的值。

例子如下：

```python
>>> class Empty:
        def __getattr__(self, attrname): # 当访问属性，而属性又未定义时，会被__getattr__拦截
            if attrname == 'age':
                return 40     # 当符合条件时，此方法返回一个值40作为X.age点号表达式的结果
            else:
                raise AttributeError(attrname)
>>> X = Empty()
>>> X.age     # 由于对象X本身没有属性，所以对X.age的访问会转至__getattr__方法
40
>>> X.name
...error text omitted...
AttributeError: name
```
由于对象X没有属性，所以对属性的访问会转至`__getattr__`方法，而当属性名称符合条件时（即，属性名称的字符串为`age`），该方法返回整数值40；而当属性名称不符合条件时，则引发内置的`AttributeError`异常。这让属性`age`看起来想实际存在的属性。实际上，这样我们可以用此方法实现**动态计算**的属性。

#### 属性的赋值

`__setattr__`方法会拦截所有属性赋值，并允许你对其进行验证（validate）或转变（transform）。如果定义了这个方法，`self.attr = value`会变成`self.__setattr__('attr', value)`。

这个方法使用起来比较tricky，因为对任何`self`的属性进行赋值都会调用`__setattr__`方法。这可能会造成`__setattr__`被重复调用，导致无穷递归循环（堆栈溢出异常）。

要使用这个方法，可以通过将实例属性赋值编码为属性字典键的赋值，来避免循环。也就说，使用`self.__dict__['name'] = x`，而不是`self.name = x`。因为你不是赋值给`__dict__`本身，这避免了循环的发生。

```python
>>> class Accesscontrol:
        def __setattr__(self, attr, value):
            if attr == 'age':
                self.__dict__[attr] = value + 10             # Not self.name=val or setattr()
            else:
                raise AttributeError(attr + ' not allowed')
>>> X = Accesscontrol()
>>> X.age = 40                               # Calls __setattr__
>>> X.age
50
>>> X.name = 'Bob'
...text omitted...
AttributeError: name not allowed
```

如果你将上面代码中的`__dict__`赋值改变为以下任一一种赋值方式，都会触发无限递归循环和异常。当`age`在类的外部被赋值时，不管是使用点号运算符还是使用内置函数`setattr`，都会失败。

```python
self.age = value + 10              # Loops
setattr(self, attr, value + 10)    # Loops (attr is 'age')
```


在类内部对另一个变量名进行赋值也会触发递归的`__setattr__`调用，尽管在类中以手动`AttributeError`异常而终止的情况较少：
```python
self.other = 99       # Recurs but doesn't loop: fails
```


通过一个调用将任何属性赋值都转至更高层级的超类，而不是对`__dict__`的键进行赋值，也可以避免递归循环：
```python
self.__dict__[attr] = value + 10 # OK: doesn't loop
object.__setattr__(self, attr, value + 10) # OK: doesn't loop (new-style only)
```

#### 属性的删除

第3个属性管理方法是`__delattr__`，它传递属性名称字符串，并在删除任何属性时被调用（例如，`del object.attr`）。就像`__setattr__`，它必须通过`__dict__`和超类来避免递归循环。


> As we’ll learn in Chapter 32, attributes implemented with new-style class features such as slots and properties are not physically stored in the instance’s `__dict__` namespace dictionary (and slots may even preclude its existence entirely!). Because of this, code that wishes to support such attributes should code `__setattr__` to assign with the object.`__setattr__` scheme shown here, not by self.`__dict__` indexing unless it’s known that subject classes store all their data in the instance itself. In Chapter 38 we’ll also see that the new-style `__getattribute__` has similar requirements. This change is mandated in Python 3.X, but also applies to 2.X if new-style classes are used.


#### 其他属性管理工具

在Python中还有其他方式来管理属性访问：

- `__getattribute__`方法拦截所有的属性获取，而不只是那些未定义的，但是当使用它的时候，你必须比使用`__getattr__`更加小心地避免循环。
- 内置函数`property`内置函数允许我们把方法和特定类属性上的获取和设置操作关联起来。
- ***描述符（descriptors）***提供了一个协议，用于将一个类的`__get__`和`__set__`方法与对特定类属性的访问关联起来。
- ***插槽（slots）***属性被声明在类中，但创建隐式的存储是在每个实例中。

#### 模拟实例属性的私有性：Part 1

下面文件`private0.py`中的代码把上一个例子通用化了，让每个子类都有自己的私有变量名列表，这些变量名无法通过其实例进行赋值：

```python
class PrivateExc(Exception): pass              # More on exceptions in Part VII

class Privacy:
    def __setattr__(self, attrname, value):    # On self.attrname = value
        if attrname in self.privates:
            raise PrivateExc(attrname, self)   # Make, raise user-define except
        else:
            self.__dict__[attrname] = value    # Avoid loops by using dict key

class Test1(Privacy):
    privates = ['age']

class Test2(Privacy):
    privates = ['name', 'pay']
    def __init__(self):
        self.__dict__['name'] = 'Tom'          # To do better, see Chapter 39!

if __name__ == '__main__':
    x = Test1()
    y = Test2()
    x.name = 'Bob'  # Works
    #y.name = 'Sue' # Fails
    print(x.name)
    y.age = 30      # Works
    #x.age = 40     # Fails
    print(y.age)
```

实际上，虽然Python不支持`private`声明，但也可以通过这种方式实现属性私有性（即无法在类外对属性名进行修改）。不过，这只是一部分解决方案，使用 ***类装饰器*** 能更加通用地拦截和验证属性。

捕获属性引用值和赋值可以支持 ***委托*** ，这也是一种设计技术。它可以让控制器对象包裹内嵌的对象，增加新行为，并且把其他运算传回包装的对象。

### 30.7 字符串重表示：`__repr__`和`__str__`
如果定义了`__repr__`或`__str__`，当类实例被打印或被转换成字符串时，`__repr__`或`__str__`就会被自动调用。这两个方法允许你为你的对象定义一种更好的显示格式。

记住，`__str__`和`__repr__`都必须返回字符串。

对于一个类的实例对象，其默认显示既没有用也不好看：

```python
>>> class adder:
        def __init__(self, value=0):
            self.data = value                  # Initialize data
        def __add__(self, other):
            self.data += other                 # Add other in place (bad form?)
>>> x = adder()                                # Default displays
>>> print(x)                                   # 这样的打印输出基本上没什么实际意义
<__main__.adder object at 0x00000000029736D8>
>>> x
<__main__.adder object at 0x00000000029736D8>
```

但是编写和继承字符串表示方法允许我们自定义显示：

```python
>>> class addrepr(adder):                       # Inherit __init__, __add__
        def __repr__(self):                     # Add string representation
            return 'addrepr(%s)' % self.data    # Convert to as-code string
>>> x = addrepr(2)                              # Runs __init__
>>> x + 1                                       # Runs __add__ (x.add() better?)
>>> x                                           # Runs __repr__
addrepr(3)
>>> print(x)                                    # Runs __repr__
addrepr(3)
>>> str(x), repr(x)                             # Runs __repr__ for both
('addrepr(3)', 'addrepr(3)')
```

上例中，`__repr__`使用基本字符串格式来将被管理的`self.data`对象转换为一种更友好的字符串显示。

#### 为什么要两个显示方法？

针对不同用户（audience），Python提供2个显示方法来支持可替代的显示方案：

- 打印操作会首先尝试`__str__`和`str`内置函数（`print`运行的内部等价形式）。它通常返回一个用户友好的显示。
- 如果`__str__`不存在，那么`__repr__`会被用于所有环境中，如，交互模式的回应（echoes），函数`repr`，函数`print`，函数`str`，以及nested appearances。该方法通常返回一个类编码（as-code）字符串，可以用来重新创建对象，或者给开发者一个详细的显示。

注意，如果没有定义`__str__`，打印还是使用`__repr__`，但反之则不成立，例如，交互模式响应只使用`__repr__`：

```python
>>> class addstr(adder):
        def __str__(self):                      # __str__ but no __repr__
            return '[Value: %s]' % self.data    # Convert to nice string
>>> x = addstr(3)
>>> x + 1
>>> x                                            # Default __repr__
<__main__.addstr object at 0x00000000029738D0>
>>> print(x)                                     # Runs __str__
[Value: 4]
>>> str(x), repr(x)
('[Value: 4]', '<__main__.addstr object at 0x00000000029738D0>')
```

**如果想让所有环境都有统一的显示，`__repr__`是最佳选择。但通过分别定义这两个方法，可以让不同环境支持不同显示。**

```python
>>> class addboth(adder):
        def __str__(self):
            return '[Value: %s]' % self.data # User-friendly string
        def __repr__(self):
            return 'addboth(%s)' % self.data # As-code string
>>> x = addboth(4)
>>> x + 1
>>> x        # Runs __repr__
addboth(5)
>>> print(x) # Runs __str__
[Value: 5]
>>> str(x), repr(x)
('[Value: 5]', 'addboth(5)')
```



#### 显示用法的注意事项（Usage Notes）
1. **记住`__str__`和`__repr__`都必须返回字符串，其他类型的结果不会转换并会引发错误，因此，如果有必要，确保用一个转换器（如，`str`或`%`）处理它们。**
2. **根据一个容器的字符串转换逻辑，`__str__`的用户友好显示可能只适用于对象出现在一个打印操作的顶层时。当对象嵌套在较大的对象中，则可能依然会使用其`__repr__`或默认方法打印。**

```python
>>> class Printer:
        def __init__(self, val):
            self.val = val
        def __str__(self):         # Used for instance itself
            return str(self.val)   # Convert to a string result
>>> objs = [Printer(2), Printer(3)]
>>> for x in objs: print(x) # __str__ run when instance printed
# But not when instance is in a list!
2
3
>>> print(objs)     # 对于嵌套在列表中的2个Printer实例，依然使用__repr__进行打印
[<__main__.Printer object at 0x000000000297AB38>, <__main__.Printer obj...etc...>]
>>> objs
[<__main__.Printer object at 0x000000000297AB38>, <__main__.Printer obj...etc...>]
```

如果想让所有环境都有统一的显示，请使用`__repr__`：
```python
>>> class Printer:
        def __init__(self, val):
            self.val = val
        def __repr__(self):            # __repr__ used by print if no __str__
            return str(self.val)       # __repr__ used if echoed or nested
>>> objs = [Printer(2), Printer(3)]
>>> for x in objs: print(x)            # No __str__: runs __repr__
2
3
>>> print(objs)                        # Runs __repr__, not ___str__
[2, 3]
>>> objs
[2, 3]
```

3. 非常微妙的是，在一些极少的环境中，显示方法可能也会触发无限递归循环。因为某些对象的显示包含其他对象的显示。一个显示触发一个正在被显示的对象的显示，这是有可能的，因此会产生循环。

### 30.8 右侧加法和原处加法的使用：`__radd__`和`__iadd__`

对于每个二元表达式，我可以实现 *左操作* 、*右操作*、*原地操作* 三种变体。虽然如果你不同时编写这三种变体的代码，也会应用缺省值，但是你的对象得角色决定了你需要编写多少个变体。

#### 右侧（Right-Side）加法

例如，`__add__`方法并不支持在`+`运算符右侧使用实例对象：

```python
>>> class Adder:
        def __init__(self, value=0):
            self.data = value
        def __add__(self, other):
            return self.data + other
>>> x = Adder(5)
>>> x + 2
7
>>> 2 + x
TypeError: unsupported operand type(s) for +: 'int' and 'Adder'
```

为了实现更通用的表达式，并支持可互换的（commutative-style）操作符，就需要连同`__radd__`方法一起实现。

只有当`+`运算符右侧的对象是你的类实例，而左侧的对象不是你的类实例时，Python才会调用`__radd__`方法。其他所有情况下，则会调用左侧对象的`__add__`方法。

```python
class Commuter1:
    def __init__(self, val):
        self.val = val
    def __add__(self, other):
        print('add', self.val, other)
        return self.val + other
    def __radd__(self, other):
        print('radd', self.val, other)
        return other + self.val
    
>>> from commuter import Commuter1
>>> x = Commuter1(88)
>>> y = Commuter1(99)
>>> x + 1 # __add__: instance + noninstance
add 88 1
89
>>> 1 + y # __radd__: noninstance + instance
radd 99 1
100
>>> x + y # __add__: instance + instance, triggers __radd__
add 88 <commuter.Commuter1 object at 0x00000000029B39E8>
radd 99 88
187
```

注意，在`__add__`中，`self`是在`+`的右侧，而`other`是在左侧；而在`__radd__`中的顺序则与之相反。

当不同类的实例混合地出现在表达式中，Python会优先选择左侧的那个类。

当我们把两个实例相加的时候，Python运行`__add__`，这反过来通过简化左边的操作数来触发`__radd__`。

##### 在`__radd__`中重用`__add__`


#### 原处（In-Place）加法



### 30.9 Call表达式：`__call__`



### 30.10 比较：`__lt__`、`__gt__`和其他方法



### 30.11 布尔测试：`__bool__`和`__len__`



### 30.12 对象析构函数：`__del__`





---


## 第31章 类的设计





---



## 第32章 类的高级话题

