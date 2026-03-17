---
title: python学习笔记2
description: ""
slug: 2021-04-11-python学习笔记2
date: 2021-04-11
image: assets/cover/python.webp
categories:
  - 编程语言
tags:
  - python
---


### 变量

* * *

> 在Python中，变量是一个用于存储数据值的容器。与一些其他编程语言不同，Python中的变量不需要显式声明类型；它们会根据分配给它们的值自动确定类型。这意味着你可以将整数、浮点数、字符串、列表、元组、字典、集合等不同类型的值赋给同一个变量，但每次赋值后，变量的类型会随之改变。

#### 变量的命名规则

在Python中，变量名必须遵循以下规则：

1.  **只能包含字母、数字和下划线**：变量名可以包含字母（a-z, A-Z）、数字（0-9）以及下划线（\_）。
2.  **不能以数字开头**：变量名不能以数字开头，但可以在名称的其余部分包含数字。
3.  **区分大小写**：Python是大小写敏感的，因此`myVar`和`myvar`会被视为两个不同的变量。
4.  **不能使用Python的保留字**：Python有一些保留字（也称为关键字），如`if`、`for`、`class`等，这些词不能用作变量名。

```python
#输出保留字列表
import keyword #import是导入包的意思，
 
print(keyword.kwlist) # 输出保留字列表
```

#### 变量的声明与赋值

在Python中，变量的声明和赋值是同时进行的。当你将一个值赋给一个变量时，Python会自动声明这个变量。例如：

```python
x = 5  # 声明一个名为x的变量，并将其赋值为5  
y = "Hello, World!"  # 声明一个名为y的变量，并将其赋值为字符串"Hello, World!"
```

#### 变量的类型

虽然Python变量不需要显式声明类型，但每个变量都有一个与之关联的类型，这个类型取决于赋给它的值。你可以使用`type()`函数来检查变量的类型：


```python
x = 10 # 整数类型
y = "张三" # 字符串类型
o = 3.14 # 浮点数类型
z = [1,2,3,4] # 数组(列表)类型 后面会讲到
a = {'name':'李四','age':19} # 字典类型 后面会讲到
b = {'北京','上海','广州','深圳'} # 集合类型 后面会讲到
p = True # 布尔类型

print(type(x))  # 输出：<class 'int'>，因为x是一个整数
print(type(y))  # 输出：<class 'str'>，因为y是一个字符串
print(type(o))  # 输出：<class 'float'>，因为o是一个浮点数
print(type(z))  # 输出：<class 'list'>，因为z是一个数组
print(type(a))  # 输出：<class 'dict'>，因为a是一个字典
print(type(b))  # 输出：<class 'set'>，因为b是一个集合
print(type(p))  # 输出：<class 'bool'>，因为p是一个布尔值
```

#### 变量的内存地址

> 在Python中，变量并不是直接存储数据本身，而是存储了对数据对象的引用（或者说是“指针”的概念，尽管Python内部并不直接使用“指针”这个词）。这意味着变量实际上存储的是数据对象在内存中的地址。然而，由于Python的内存管理机制（包括垃圾回收机制），直接暴露内存地址给开发者并不是Python设计的一部分，因此我们不能直接像C或C++那样通过指针来操作内存地址。 尽管如此，Python提供了一些方法来间接查看或理解变量的“内存地址”概念，尽管这些通常是通过对象的身份（identity）来表现的，而不是直接的内存地址。

##### 如何查看变量的“内存地址”

在Python中，你可以使用`id()`函数来获取一个对象的“身份”，这个函数返回的是一个整数，这个整数在对象的生命周期内是唯一的，并且在大多数实现中，这个整数可以被视为对象在内存中的地址（尽管这不是严格保证的，因为它依赖于Python解释器的具体实现）。

```python
x = 5
print(id(x))  # 输出x变量的内存地址：4367788464

y = "hello"
print(id(y))  # 输出y变量的内存地址：4368876848
```

##### 一个变量指向多个值？

在Python中，一个变量不能“直接”指向多个值。变量在任何给定时间都只引用（或“指向”）一个对象。但是，你可以通过改变变量的引用来使其指向不同的对象。

```python
a = 10
print(id(a))  # 变量a的内存地址：4309609040 

#给a变量重新赋值
a = 20
print(id(a))  # 变量a的内存地址：4309609360，与上一个不同，因为a现在引用了一个不同的对象
```

在这个例子中，变量`a`最初引用了一个值为10的对象，然后我们改变了`a`的引用，使其指向了一个值为20的新对象。这并不意味着`a`“同时”指向了两个值；它只是改变了它的引用。

#### 不可变对象与可变对象

> 在Python中，对象根据其是否可以在创建后被修改分为不可变对象（immutable objects）和可变对象（mutable objects）。这种分类对于理解Python的内存管理、变量赋值、函数参数传递等方面非常重要。

##### 不可变对象（Immutable Objects）

不可变对象一旦创建，其内部状态就不能被改变。如果尝试修改一个不可变对象，实际上会创建一个新的对象，而不是修改原始对象。Python中的不可变对象包括：

*   整数（int）
*   浮点数（float）
*   字符串（str）
*   元组（tuple）
*   布尔值（True 和 False，实际上是int的子类）
*   None（空值，也是单例对象）

```python
# 字符串是不可变的  
s = "hello"  
s_new = s.replace("e", "a")  # 创建一个新的字符串对象  
print(id(s) == id(s_new))  # 输出False，s和s_new的内存地址不同，因为s和s_new是不同的对象  
  
# 元组也是不可变的  
t = (1, 2, 3)  
t_new = (1, 2, 3, 4)  # 创建一个新的元组对象  
print(id(t) == id(t_new))  # 输出False，因为t和t_new是不同的对象  
  
# 尝试修改元组中的元素会抛出TypeError  
# t[1] = 20  # 这是不允许的
```

##### 可变对象（Mutable Objects）

可变对象在创建后可以被修改。这意味着你可以改变对象内部的状态，而不会创建新的对象。Python中的可变对象包括：

*   列表（list）
*   字典（dict）
*   集合（set）
*   自定义的类对象（如果类中定义了修改对象状态的方法）

```python
# 列表是可变的  
l = [1, 2, 3]
print(id(l)) # 输出列表l的内存地址：4336280832
l.append(4)  # 修改列表对象l,其实就是想列表l中添加了一个元素
print(id(l))  # 再次输出列表l的内存地址：4336280832，注意这个id在修改前后不会改变

# 字典也是可变的  
d = {'a': 1, 'b': 2}
print(id(d)) # 输出字典d的内存地址：4302206208
d['c'] = 3  # 修改字典对象d  
print(id(d))  # 再次输出字典d的内存地址：4302206208，同样在修改前后不会改变

# 集合也是可变的  
s = {1, 2, 3}
print(id(s)) # 输出集合s的内存地址：4310771072
s.add(4)  # 修改集合对象s  
print(id(s))  # 再次输出集合s的内存地址：4310771072，修改前后不会改变
```

##### 不可变对象与可变对象的区别对编程的影响

1.  **赋值与传递**：当不可变对象被赋值给另一个变量时，或者作为函数参数传递时，实际上是在传递对象的引用（或者说是对象的“身份”或“内存地址”的副本）。但是，由于对象是不可变的，所以这种传递是安全的，因为原始对象不会被修改。对于可变对象，赋值或传递实际上是在共享同一个对象，因此一个变量对对象的修改会影响到所有引用该对象的变量。
2.  **内存管理**：Python的垃圾回收机制会跟踪对象的引用计数。对于不可变对象，由于它们不能被修改，因此当没有任何变量引用它们时，它们就可以被安全地回收。对于可变对象，由于它们的状态可以改变，因此Python的垃圾回收机制需要更加复杂地处理它们的生命周期。
3.  **线程安全**：由于不可变对象在创建后不能被修改，因此它们在多线程环境中是线程安全的。相比之下，可变对象需要额外的同步机制来确保线程安全。

#### 变量的作用域

Python中变量的作用域是一个关键概念，它决定了变量在程序中哪些部分是可访问的。Python的作用域规则相对直观，但理解它们对于编写清晰、可维护的代码至关重要。下面我将详细解释Python中的变量作用域，包括全局作用域、局部作用域、闭包以及相关的关键字（如`global`和`nonlocal`），并提供代码案例来加深理解。

##### 1\. 全局作用域（Global Scope）

全局作用域是指在程序的最外层定义的变量所拥有的作用域。这些变量在整个程序中都是可见的，包括所有的函数内部。但是，在函数内部直接修改全局变量需要特别注意，因为默认情况下，在函数内部赋值会创建一个新的局部变量（如果变量名已存在）或一个新的全局变量（如果变量名不存在于全局作用域中）。

###### 1.1、错误修改全局变量案例

```python
# 定义全局变量
global_var = "I am global"
#定义一个名字叫做my_function的函数
def my_function():
    # 尝试访问全局变量
    print(global_var) 

    '''
    尝试修改全局变量（错误的方式） 
    如果这样直接修改全局变量的值，会出现报错， 因为这样就会导致global_var变成一个局部变量
    上面还有一个输出global_var的语句呢，这个时候输出的就是局部变量的值，但是局部变量是在下面修改全局变量的时候才被创建
    所以上面的 print(global_var) 是获取不到 局部变量global_var的，所以会报错
    '''
    global_var = "I tried to modify global, but this creates a new local variable"
    print(global_var)

#访问my_function函数
my_function()

#输出运行之后的报错信息：UnboundLocalError: local variable 'global_var' referenced before assignment
```

###### 1.2、正确修改全局变量案例

```python
# 定义全局变量
global_var = "I am global"
#定义一个名字叫做my_function的函数
def my_function():
    '''
        global表示声明全局变量，global global_var表示global_var是一个全局变量
        表示可以在局部去修改这个全局变量的值
        注意：在函数中使用global语句的时候最好写在函数的最上方
    '''
    global global_var

    # 尝试访问全局变量
    print(global_var) # 输出：I am global

    # 现在再次修改global_var，就是表示修改的全局变量的值
    global_var = "I am modified globally"
    print(global_var)  # 输出: I am modified globally

#访问my_function函数
my_function()
#在外部输出全局变量的值，查看全局变量的值是否被改变
print(global_var) # 输出：I am modified globally
```

##### 2\. 局部作用域（Local Scope）

局部作用域是指在函数内部定义的变量所拥有的作用域。这些变量只能在函数内部被访问和修改。一旦函数执行完毕，这些变量就会被销毁（除非它们被返回或以其他方式传递给外部作用域）。

```python
#定义一个函数
def my_function():
    #在函数中声明一个变量，局部变量
    local_var = "I am local"
    print(local_var)  # 输出: I am local  
#调用函数
my_function()

# 尝试访问局部变量（会导致错误），因为local_var所在的函数已经执行完了，里面变量的资源已经被释放了
print(local_var)  # 报错信息：NameError: name 'local_var' is not defined
```

##### 3\. 闭包（Closures）

闭包是Python中一个高级概念，它涉及到嵌套函数和非局部变量的使用。当一个内部函数引用了其外部函数的局部变量时，就形成了一个闭包。即使外部函数已经执行完毕，其局部变量仍然可以被内部函数访问，因为这些局部变量被绑定在了内部函数的词法环境中。

```python
def outer_function(text):  
    # outer_function的局部变量  
    message = text  
      
    def inner_function():  
        # inner_function可以访问outer_function的局部变量message  
        print(message)  
      
    # 返回内部函数，此时就形成了一个闭包  
    return inner_function  
  
# 调用outer_function并获取闭包  
my_closure = outer_function("Hello from a closure!")  
  
# 调用闭包  
my_closure()  # 输出: Hello from a closure!
```

##### 4\. `global` 和 `nonlocal` 关键字

*   **`global` 关键字**：用于在函数内部声明全局变量，以便在函数内部修改全局变量的值。
*   **`nonlocal` 关键字**：用于在嵌套函数内部声明非局部变量（即外部函数的局部变量），以便在内部函数中修改外部函数的局部变量。

###### `global关键字`

```python
# 定义一个全局变量，并赋予它一个初始值："I am global"  
global_var = "I am global"  
  
# 定义一个名为 my_function 的函数  
def my_function():  
    # 使用 global 关键字声明，接下来的 global_var 引用将是指向全局作用域中的 global_var 变量  
    global global_var  
    # 修改全局变量 global_var 的值为："I was modified globally"  
    global_var = "I was modified globally"  
  
# 调用 my_function 函数  
my_function()  
# 在全局作用域中打印 global_var 的值，输出将会是："I was modified globally"  
print(global_var)  # 输出: I was modified globally
```

注意，如果在函数内部使用global关键字声明全局变量的话，最好要在函数的最开始进行使用，例如global 声明了global\_var全局变量，但是在声明之前，global\_var就在函数中被使用到了，就会出现报错的。

###### **`nonlocal关键字`**

```python
# 定义一个名为 outer_function 的外层函数
def outer_function():
    # 在 outer_function 内部定义一个变量 outer_var 并初始化为 "I am outer"
    outer_var = "I am outer"

    # 在 outer_function 内部定义一个名为 inner_function 的内层函数
    def inner_function():
        # 使用 nonlocal 关键字声明 outer_var 是非局部变量，即它引用的是外层函数的 outer_var
        nonlocal outer_var
        # 修改 outer_var 的值为 "I was modified by an inner function"
        outer_var = "I was modified by an inner function"
        # 打印修改后的 outer_var 的值
        print(outer_var) # 输出：I was modified by an inner function

        # 调用 inner_function

    inner_function()
    # 在 outer_function 内部再次打印 outer_var 的值，展示 inner_function 对它的修改
    print(outer_var)# 输出: I was modified by an inner function


# 调用 outer_function
outer_function()
```

