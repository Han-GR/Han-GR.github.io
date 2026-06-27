---
title: python学习笔记19-异常处理机制
description: ""
slug: 2021-07-11-python学习笔记19-异常处理机制
date: 2021-07-11
image: python.webp
categories:
  - 编程语言
tags:
  - python
  - 异常处理
  - try
  - except
---
### 什么是异常处理机制

> Python中的异常处理机制是一种用于捕获和处理程序运行时可能出现的错误或异常情况的机制。它允许程序在出现错误时不会立即崩溃，而是能够优雅地处理这些错误，或者采取一些恢复措施。

-   如果代码没有语法问题，可以运行，但会出运行时的错误，例如除零错误，下标越界等问题，这种在运行期间检测到的错误被称为异常 。
-   出现了异常必须处理否则程序会终止执行，用户体验会很差。
-   Python支持程序员自己处理检测到的异常。
-   可以使用try-except语句进行异常的检测和处理。

### 基本结构

Python的异常处理主要通过try、except、else和finally这几个关键字来实现。

#### try和except

-   try块：包含可能引发异常的代码。
-   except块：处理特定异常。可以有多个except块来处理不同类型的异常。
##### 语法

```python
try:
    可能会出现异常的代码
except 异常类型:
    报错后要执行的代码
```

异常处理机制就是当程序遇到报错的时候，不会终止程序的执行，而是会捕捉到异常并进行处理 但是这个异常类型是需要手动写上去的，可以指定捕捉异常， 例如：ZeroDivisionError: division by zero，报错，除数不能为0，ZeroDivisionError就是除数不能为零的异常类型 可以把这个异常类型写到except上，这样程序只会捕获这个异常，遇到这个异常才会执行except下面的代码， 如果遇到的是其他异常还是会报错的

##### 案例

```python
try:  
    # 可能引发异常的代码  
    result = 10 / 0  
except ZeroDivisionError:  
    # 处理ZeroDivisionError异常的代码  
    print("除数不能为零！")
#输出结果：除数不能为零！
```

-   try块：
    -   try: 关键字后面跟随的代码块是尝试执行的代码。
    -   在这个例子中，尝试执行的代码是 result = 10 / 0。这行代码试图将10除以0，这在数学上是未定义的，并且在大多数编程语言中会导致运行时错误。
    -   在Python中，尝试除以0会引发一个ZeroDivisionError异常。
-   except块：
    -   except ZeroDivisionError: 关键字后面跟随的代码块是当try块中的代码引发ZeroDivisionError异常时执行的代码。
    -   在这个例子中，当10 / 0引发ZeroDivisionError时，程序将不会崩溃，而是跳转到except块并执行其中的代码。
    -   except块中的代码是 print(“除数不能为零！”)，它输出一条错误信息到控制台。
-   程序输出：
    -   由于10 / 0引发了ZeroDivisionError，程序执行了except块中的代码，输出了“除数不能为零！”。
    -   因此，程序的输出结果是“除数不能为零！”。
-   程序流程：
    -   程序首先尝试执行try块中的代码。
    -   当try块中的代码引发ZeroDivisionError时，程序跳转到except ZeroDivisionError:块。
    -   执行except块中的代码，输出错误信息。
    -   except块执行完毕后，程序继续向下执行（如果有的话）。在这个例子中，except块后面没有其他代码，所以程序结束。

#### else块

else块是可选的，当try块中的代码没有引发任何异常时，else块中的代码会被执行。

##### 语法

该机制就是如果遇到异常执行except里面的内容，如果没有报错，执行else里面的内容

```python
try:
...
except 异常类型:
...
else
...
```

##### 案例

```python
try:  
    # 可能引发异常的代码  
    result = 10 / 2  
except ZeroDivisionError:  
    # 处理ZeroDivisionError异常的代码  
    print("除数不能为零！")  
else:  
    # 如果没有异常发生，执行这里的代码  
    print("计算成功，结果是：", result)
```

-   try块：
    -   try: 关键字后面跟随的是可能引发异常的代码。
    -   在这个例子中，尝试执行的代码是 result = 10 / 2。这行代码是合法的，因为它不会引发任何异常（10除以2等于5，是一个有效的数学运算）。
-   except块：
    -   except ZeroDivisionError: 关键字后面跟随的是当try块中的代码引发ZeroDivisionError异常时执行的代码。
    -   然而，在这个例子中，10 / 2不会引发ZeroDivisionError，因此except块中的代码不会被执行。
    -   这个except块在这里是多余的，因为它针对的异常类型在这种情况下不会发生。
-   else块：
    -   else: 关键字后面跟随的是当try块中的代码没有引发任何异常时执行的代码。
    -   在这个例子中，由于10 / 2没有引发异常，程序将跳转到else块并执行其中的代码。
    -   else块中的代码是 print(“计算成功，结果是：”, result)，它输出一条成功信息和计算结果到控制台。
-   程序输出：
    -   由于try块中的代码没有引发异常，程序执行了else块中的代码。
    -   因此，程序的输出结果是“计算成功，结果是： 5”。
-   程序流程：
    -   程序首先尝试执行try块中的代码。
    -   try块中的代码成功执行，没有引发异常。
    -   程序跳转到else块并执行其中的代码。
    -   else块执行完毕后，程序继续向下执行（如果有的话）。在这个例子中，else块后面没有其他代码，所以程序结束。

#### finally块

finally块也是可选的，但非常有用。无论是否发生异常，finally块中的代码都会被执行。常用于释放资源，如关闭文件或网络连接。

##### 语法

该机制就是如果遇到异常执行except里面的内容，如果没有报错，执行else里面的内容， 而不管有没有报错都会之心finally里面的内容，一般常用语释放已经已使用的资源，例如连接数据库的时候，如果连接数据库失败那么直接关闭数据库的资源

```python
try:
...
except 异常类型:
...
else
...
finally
...
```

##### 案例

```python
try:  
    # 可能引发异常的代码  
    result = 10 / 2  
except ZeroDivisionError:  
    # 处理ZeroDivisionError异常的代码  
    print("除数不能为零！")  
else:  
    # 如果没有异常发生，执行这里的代码  
    print("计算成功，结果是：", result)  
finally:  
    # 无论是否发生异常，都会执行这里的代码  
    print("执行完成，清理资源。")
```

-   try块：
    -   try: 关键字后面跟随的是可能引发异常的代码。
    -   在这个例子中，尝试执行的代码是 result = 10 / 2。这行代码是合法的，因为它不会引发任何异常（10除以2等于5，是一个有效的数学运算）。
-   except块：
    -   except ZeroDivisionError: 关键字后面跟随的是当try块中的代码引发ZeroDivisionError异常时执行的代码。
    -   然而，在这个例子中，10 / 2不会引发ZeroDivisionError，因此except块中的代码不会被执行。
    -   这个except块在这里是多余的，因为它针对的异常类型在这种情况下不会发生。但在实际编程中，except块通常用于处理可能发生的特定异常类型。
-   else块：
    -   else: 关键字后面跟随的是当try块中的代码没有引发任何异常时执行的代码。
    -   在这个例子中，由于10 / 2没有引发异常，程序将跳转到else块并执行其中的代码。
    -   else块中的代码是 print(“计算成功，结果是：”, result)，它输出一条成功信息和计算结果到控制台。
-   finally块：
    -   finally: 关键字后面跟随的是无论是否发生异常都会执行的代码。
    -   在这个例子中，无论try块中的代码是否引发异常，finally块中的代码 print(“执行完成，清理资源。”) 都会被执行。
    -   finally块通常用于释放资源，如关闭文件、数据库连接或释放锁等，无论操作是否成功完成。
-   程序输出：
    -   由于try块中的代码没有引发异常，程序执行了else块中的代码，输出了“计算成功，结果是： 5”。
    -   随后，程序执行了finally块中的代码，输出了“执行完成，清理资源。”。 因此，程序的完整输出结果是：


```
计算成功，结果是： 5  
执行完成，清理资源。
```

-   程序流程：
    -   程序首先尝试执行try块中的代码。
    -   try块中的代码成功执行，没有引发异常。
    -   程序跳转到else块并执行其中的代码。
    -   else块执行完毕后，程序继续执行finally块中的代码。
    -   finally块执行完毕后，程序结束。

#### 捕获多种异常

> 如果遇到了多个异常，可能遇到的报错不是自己指定的异常，比如指定的异常是除数不能为0 但是如果在输入数字的时候出现输入成字母的情况还是会报错，但是现在又想如果输入的是字母就会提示说不能输入字母 那么就需要使用连续捕捉异常

可以在一个except块中捕获多种异常，使用逗号分隔异常类型。或者编写多个except块
##### 逗号分隔捕获多种异常

```python
try:  
    # 可能引发异常的代码  
    value = int("abc")  
except (ValueError, TypeError):  
    # 处理ValueError和TypeError异常的代码  
    print("输入的值不是有效的整数。")
```

-   try块：
    -   try: 关键字后面跟随的是可能引发异常的代码。
    -   在这个例子中，尝试执行的代码是 value = int(“abc”)。这行代码试图将字符串 “abc” 转换为整数，但这是一个不合法的操作，因为 “abc” 不是一个有效的整数表示。
-   except块：
    -   except (ValueError, TypeError): 关键字后面跟随的是当try块中的代码引发ValueError或TypeError异常时执行的代码。
    -   在这个例子中，int(“abc”)会引发一个ValueError异常，因为 “abc” 不能被转换为整数。
    -   注意，虽然这里同时捕获了ValueError和TypeError，但在这个特定的操作中，只有ValueError会被引发。TypeError通常发生在操作或函数应用于错误类型的对象时，而在这个例子中，"abc"的类型是str，它是int()函数可以接受的（尽管内容不是有效的整数）。然而，int()函数在内容不是有效整数时会引发ValueError，而不是TypeError。
-   异常处理代码：
    -   当try块中的代码引发ValueError异常时，程序会跳转到except块并执行其中的代码。
    -   except块中的代码是 print(“输入的值不是有效的整数。”)，它输出一条错误信息到控制台。
-   程序输出：
    -   由于int(“abc”)引发了ValueError异常，程序执行了except块中的代码。
    -   因此，程序的输出结果是“输入的值不是有效的整数。”。
-   程序流程：
    -   程序首先尝试执行try块中的代码。
    -   try块中的代码引发了一个ValueError异常。
    -   程序跳转到except块并执行其中的代码。
    -   except块执行完毕后，程序继续向下执行（如果有的话）。在这个例子中，except块后面没有其他代码，所以程序结束。

##### 编写多个except块捕获多种异常

多个except结构 捕获异常的顺序按照先子类后父类的顺序，为了避免遗漏可能出现的异常，可以在最后增加BaseException 捕获异常会先从第一个except去处理，如果处理不了就下一个except去处理，最后最后有一个最大的异常 可以捕获到所有异常的异常类型，保证程序不会报错终止

```python
try :
    a = int(input("请输入一个数字："))
    b = int(input("请再输入一个数字："))
    c = a / b
    print(c)
except ZeroDivisionError :
    print("除数不能为0")
except ValueError :
    print("不可以输入字母")
except BaseException :
    print("出错了")
```

#### 捕获所有异常

使用Exception可以捕获所有异常（除了SystemExit、KeyboardInterrupt和GeneratorExit等系统退出异常）。

```python
try:  
    # 可能引发异常的代码  
    value = int("abc")  
except Exception as e:  
    # 处理所有异常的代码  
    print("发生了一个异常")
```

-   try块：
    -   try: 关键字后跟随的是可能触发异常的代码。
    -   在这个例子中，尝试执行的代码是 value = int(“abc”)。这行代码试图将字符串 “abc” 转换为整数，但由于 “abc” 不是一个有效的整数表示，因此这个操作是不合法的。
-   except块：
    -   except Exception as e: 关键字后跟随的是当try块中的代码引发任何Exception类（或其子类）的异常时执行的代码。
    -   在这个例子中，int(“abc”)会引发一个ValueError异常，因为 “abc” 不能被转换为整数。值得注意的是，ValueError是Exception类的一个子类。
    -   as e这部分代码表示将捕获的异常对象赋值给变量e，虽然在except块的代码中并没有使用这个变量。
-   异常处理代码：
    -   当try块中的代码引发异常时，程序会跳转到except块并执行其中的代码。
    -   在这个except块中，代码是 print(“发生了一个异常”)，它会在控制台输出一条简单的错误信息。
-   程序输出：
    -   由于int(“abc”)引发了ValueError异常，并且ValueError是Exception的子类，因此程序会执行except块中的代码。
    -   因此，程序的输出结果是“发生了一个异常”。
-   程序流程：
    -   程序首先尝试执行try块中的代码。
    -   try块中的代码引发了一个ValueError异常。
    -   由于ValueError是Exception的子类，程序跳转到except Exception as e:块并执行其中的代码。
    -   except块执行完毕后，程序继续向下执行（如果有的话）。在这个例子中，except块后面没有其他代码，所以程序结束。
-   关于异常对象的处理：
    -   在这个例子中，虽然捕获了异常对象并将其赋值给了变量e，但在except块的代码中并没有使用这个变量。在实际编程中，可以通过这个变量来获取有关异常的更详细信息，例如异常的类型、消息和堆栈跟踪。
-   最佳实践：
    -   虽然在这个例子中捕获所有Exception类的异常可能看起来很方便，但通常不推荐这样做，因为它会捕获所有类型的异常，包括那些可能表示程序存在严重问题的异常（如SystemExit、KeyboardInterrupt等）。更好的做法是捕获你可能期望处理的具体异常类型（如ValueError），或者至少捕获更具体的异常基类（如RuntimeError），而不是捕获所有异常。

#### 获取异常信息

在except块中，可以使用as关键字来捕获异常对象，从而获取异常的具体信息。

```python
try:  
    # 可能引发异常的代码  
    value = int("abc")  
except ValueError as e:  
    # 获取并打印异常信息  
    print(f"异常信息：{e}")
```

当int(“abc”)尝试执行时，由于"abc"不是一个有效的整数表示，因此会引发ValueError异常。这个异常被except ValueError as e子句捕获，并且异常对象被赋值给变量e。然后，except块中的代码使用e来打印异常信息。

##### 使用as关键字有几个好处：

-   获取异常信息：通过捕获的异常对象，你可以获取有关异常的详细信息，如异常类型、错误消息和堆栈跟踪。
-   自定义异常处理：你可以根据捕获的异常对象的属性或类型来决定如何处理异常。例如，你可能只想处理特定类型的ValueError，或者你可能想根据不同的错误消息来执行不同的操作。
-   提高代码可读性：在except块中使用变量来引用异常对象可以使代码更清晰、更易于理解。

#### 自定义异常

Python允许用户通过继承内置的Exception类来创建自定义异常。

```python
class MyCustomError(Exception):  
    def __init__(self, message):  
        super().__init__(message)  
        self.message = message  
  
try:  
    # 可能引发自定义异常的代码  
    raise MyCustomError("这是一个自定义异常！")  
except MyCustomError as e:  
    # 处理自定义异常的代码  
    print(f"捕获到自定义异常：{e.message}")
```
