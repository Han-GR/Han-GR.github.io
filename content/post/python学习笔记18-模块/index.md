---
title: python学习笔记18-模块
description: ""
slug: 2021-07-08-python学习笔记18-模块
date: 2021-07-08
image: python.webp
categories:
  - 编程语言
tags:
  - python
  - 模块
---
### 模块

> Python中的模块是一种将相关代码组织在一起的方式，以便更容易地重用和维护。模块可以是Python文件（以.py结尾），也可以是C或C++扩展，或者是已经编译成共享库（动态链接库）的扩展。Python标准库包含了大量的模块，用于执行各种任务，如文件操作、网络编程、字符串处理等。

### 模块的定义和导入

1.  模块 就好比是 工具包，要想使用这个工具包中的工具，就需要 导入 import 这个模块
2.  每一个以扩展名 py 结尾的 python 源代码文件都是一个 模块
3.  在模块中定义的 全局变量 、 函数 都是模块能够提供给外界直接使用的工具

#### 定义模块

要定义一个模块，只需创建一个Python文件，并在其中编写代码。例如，创建一个名为mymodule.py的文件：

```python
# mymodule.py
def greet(name):
    return f"Hello, {name}!"
```

#### 导入模块

要使用模块中的函数、类或变量，首先需要导入该模块。有几种导入模块的方法：

##### 直接导入模块

```python
import mymodule
print(mymodule.greet("Alice"))  # 输出: Hello, Alice!
```

##### 导入模块中的特定部分

```python
from mymodule import greet
print(greet("Bob"))  # 输出: Hello, Bob!
```

##### 导入模块并为其指定别名

```python
import mymodule as mm
print(mm.greet("Charlie"))  # 输出: Hello, Charlie!
```

##### 导入模块中的所有内容（不推荐，可能会导致命名冲突）

```python
from mymodule import *
print(greet("David"))  # 输出: Hello, David!
```

### 使用标准库模块

> Python标准库包含了许多模块，这些模块提供了大量用于执行常见任务的函数和类。例如，os模块用于与操作系统交互，sys模块提供了一些与Python解释器交互的函数，re模块用于正则表达式匹配等。

例如：
```python
import os
print(os.getcwd())  # 获取当前工作目录
```

#### 常用模块

|          |                                                                            |
|----------|----------------------------------------------------------------------------|
|    os    |              os 模块提供了许多与操作系统交互的函数，例如创建、移动和删除文件和目录，以及访问环境变量等。               |
|   sys    | sys 模块提供了与 Python 解释器和系统相关的功能，例如解释器的版本和路径，以及与 stdin、stdout 和 stderr 相关的信息。 |
|   time   |                  time 模块提供了处理时间的函数，例如获取当前时间、格式化日期和时间、计时等。                  |
| datetime |              datetime 模块提供了更高级的日期和时间处理函数，例如处理时区、计算时间差、计算日期差等。              |
|  random  |                   random 模块提供了生成随机数的函数，例如生成随机整数、浮点数、序列等。                   |
|   math   |                    math 模块提供了数学函数，例如三角函数、对数函数、指数函数、常数等。                    |
|    re    |                     re 模块提供了正则表达式处理函数，可以用于文本搜索、替换、分割等。                     |
|   json   |      json 模块提供了JSON编码和解码函数，可以将Python对象转换为JSON格式，并从JSON格式中解析出Python对象。      |
|  urllib  |           urllib 模块提供了访问网页和处理URL的功能，包括下载文件、发送POST请求、处理cookies等。            |

##### random

-   获取随机数，需要引入random库
-   import random

|                                |                                                                   |
|--------------------------------|-------------------------------------------------------------------|
| randrange(start, stop[, step]) | start指定范围的起始值包含本身，默认是0；stop指定范围的结束值不包含本身；step为步长，默认步长是1。该函数返回一个整数 |
|      randint(start, end)       |               返回[start, end]之间的一个随机整数，start必须小于end                |
|            random()            |                       返回一个[0.0, 1.0)之间的随机小数                       |
|          choice(seq)           |                     返回一个序列（列表、元组、字符串）中的一个随机元素                     |
|          shuffle(seq)          |                          将序列元素随机排列（打乱顺序）                          |

##### math

-   操作数字的运算
-   import math

|              |      |                            |
|--------------|------|----------------------------|
| math.ceil()  | 向上取整 |    math.ceil(18.1) #19     |
| math.floor() | 向下取整 |    math.floor(18.1) #18    |
| math.sqrt()  | 求平方根 | math.sqrt(100) # 结果应为10.0， |

##### re

-   正则表达式处理
-   可以用于文本搜索、替换、分割等

|        |                                               |
|--------|-----------------------------------------------|
|   Ls   |                匹配任意空白字符，等价于\s                 |
|   \S   |                   匹配任意非空字符                    |
|   \d   |                匹配任意数字，等价于[0-9]                |
|   \D   |                    匹配任意非数字                    |
|   ^    |                    匹配字符串开始                    |
|   $    |         匹配字符串结束，如果是存在换行，只匹配到换行前的结束字符串         |
|   \A   |                匹配字符串开始（忽略多行模式）                |
|   \Z   |                匹配字符串结束（忽略多行模式）                |
|   \z   |                匹配字符串结束（考虑多行模式）                |
|   G    |                  匹配最后匹配完成的位置                  |
|   \n   |                    匹配一个换行符                    |
|   \t   |                    匹配一个制表符                    |
|   \b   |               匹配字符串的开头或结尾，或单词边界               |
|   .    | 匹配任意字符，除了换行符；当re.DOTALL标记被指定时，则可以匹配包括换行符的任意字符 |
| [...]  |          用来表示一组字符，单独列出：[amk]匹配a、m或k           |
| [^...] |         不在口中的字符：[^abc]匹配除了a、b、c之外的字符          |
|   *    |                  匹配0个或多个的表达式                  |
|   +    |                  匹配1个或多个的表达式                  |
|   ?    |          匹配0个或1个由前面的正则表达式定义的片段，非贪婪方式          |
|  {n}   |                  精确匹配n个前面表达式                  |
| {n,m}  |           匹配n到m次由前面的正则表达式定义的片段，贪婪方式           |
| (...)  |               匹配括号内的表达式，也表示一个组                |

##### turtle

-   1969年诞生，Python语言的标准库之一，入门级的图形绘制函数库
-   它提供了一个海龟，你可以把它理解为一个机器人，只听得懂有限的指令，海龟走过的轨迹绘制成了图形

|                                         |                                                  |
|-----------------------------------------|--------------------------------------------------|
|            forward(d)/fd(d)             |                  向当前画笔方向移动d像素长度                  |
|        backward(d)/back(d)/bk(d)        |                 向当前画笔相反方向移动d像素长度                 |
| goto(x,y)/setpos(x,y)/setposition(x,y)  |                 将画笔移动到坐标为x,y的位置                  |
|                 setx(x)                 |                设置海龟的横坐标为x，纵坐标保持不变                |
|                 sety(y)                 |                设置海龟的纵坐标为y，横坐标保持不变                |
|              penup()/up()               |              提起笔移动，不绘制图形，用于另起一个地方绘制              |
|            pendown()/down()             |               放下笔，移动时绘制图形，缺省时也为绘制                |
|        right(degree)/rt(degree)         |                   顺时针移动degree°                   |
|         left(degree)/lt(degree)         |                   逆时针移动degree°                   |
|      setheading(angle)/seth(angle)      |                  设置海龟的朝向为angle                   |
| circle(radius, extent=None, steps=None) | 绘制圆弧（radius为半径，extent为圆弧范围，steps为绘制圆弧时的步数，后两者可选） |
|          dot(radius, colorstr)          |         绘制一个指定直径（radius）和颜色（colorstr）的圆点         |
|                 home()                  |                 设置当前画笔位置为原点，朝向东                  |

### 自定义模块

除了标准库模块，你还可以创建自己的模块。这些自定义模块可以放在当前脚本的同一目录中，或者放在Python的模块搜索路径（如sys.path）中的某个目录中。

### 包（Packages）

> 包是一种包含多个模块的容器。包通常包含一个\_\_init\_\_.py文件（即使该文件为空），这使得Python将该目录视为一个包。包的结构通常如下

```python
mypackage/
    __init__.py
    module1.py
    module2.py
```

#### 要导入包中的模块，可以使用点（.）表示法：

```python
import mypackage.module1
print(mypackage.module1.some_function())
```

#### 或者，从包中导入特定的部分：

```python
from mypackage.module2 import another_function
another_function()
```

### 虚拟环境

为了避免不同项目之间的依赖冲突，Python提供了虚拟环境（Virtual Environments）。虚拟环境是一个独立的Python环境，在其中安装的模块不会影响全局Python环境。

Python虚拟环境是一个独立的、隔离的Python运行环境，它拥有自己的Python解释器、第三方库和应用程序。这种隔离性使得不同项目之间的依赖关系不会相互干扰，每个项目都可以使用自己独立的Python解释器和第三方库版本。

#### 虚拟环境的作用

1.  隔离性：每个虚拟环境都是独立的，互不影响。这意味着在一个虚拟环境中安装的Python包不会影响其他虚拟环境或全局Python环境。
2.  可定制性：可以根据项目的需求，为每个虚拟环境选择特定的Python版本和安装所需的第三方包。
3.  可复制性：虚拟环境可以轻松地复制和迁移到其他机器上，确保在不同环境中的一致性。
4.  易于管理：通过激活和停用虚拟环境，可以方便地切换到不同的Python项目环境。

#### 如何创建和使用虚拟环境

Python提供了多种创建虚拟环境的方法，包括使用内置的venv模块和第三方库virtualenv。

##### 方法一：使用内置的venv模块（Python 3.3及以上版本）

1.  打开命令行终端：如Windows的命令提示符或PowerShell，macOS和Linux的终端。
2.  进入项目目录：使用cd命令进入你想要创建虚拟环境的项目目录。
3.  创建虚拟环境：使用命令`python -m venv env_name`，其中env\_name是你为虚拟环境指定的名称。
4.  激活虚拟环境：
    -   在Windows上，执行`env_name\Scripts\activate.bat`。
    -   在macOS和Linux上，执行`source env_name/bin/activate`。

激活后，命令行提示符会显示虚拟环境的名称，表明你正在虚拟环境中工作。

1.  安装依赖包：使用pip install 包名命令安装项目所需的库。
2.  运行项目：在虚拟环境中运行你的Python项目。
3.  退出虚拟环境：使用deactivate命令退出当前虚拟环境。

##### 方法二：使用第三方库virtualenv

1.  安装virtualenv：如果还没有安装virtualenv，可以使用`pip install virtualenv`命令进行安装。
2.  创建虚拟环境：使用命令`virtualenv env_name`创建虚拟环境。
3.  激活虚拟环境：与使用venv模块相同，根据操作系统执行相应的激活命令。
4.  后续步骤：安装依赖包、运行项目和退出虚拟环境的步骤与使用venv模块相同。

#### 虚拟环境的迁移和打包

1.  打包依赖包：使用`pip freeze > requirements.txt`命令将当前虚拟环境中的依赖包版本信息导出到requirements.txt文件中。
2.  迁移虚拟环境：在新环境中，使用`pip install -r requirements.txt`命令根据requirements.txt文件安装所有依赖包，从而复制原虚拟环境。

#### 使用国内镜像源安装依赖包

由于网络原因，有时直接从Python官方的PyPI源安装依赖包可能会很慢或失败。此时，可以使用国内镜像源来加速安装过程。例如，使用清华大学的镜像源：

1.  临时使用：在安装依赖包时，使用-i选项指定镜像源，如`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package`。
2.  设置为默认源：使用`pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple`命令将清华大学镜像源设置为默认源。

#### 其他虚拟环境管理工具

除了venv和virtualenv之外，还有一些其他的虚拟环境管理工具，如pipenv和poetry。这些工具提供了更高级的功能，如依赖包管理、虚拟环境创建和删除、项目发布等。

1.  pipenv：集成了pip和virtualenv的功能，并提供了Pipfile和Pipfile.lock文件来管理项目依赖。使用`pip install pipenv`命令安装pipenv后，可以使用pipenv install命令安装依赖包，pipenv shell命令激活虚拟环境等。
2.  poetry：是一个用于Python项目打包和依赖管理的工具。它提供了类似于pipenv的功能，但具有更强大的依赖解析和版本控制能力。使用`pip install poetry`命令安装poetry后，可以使用poetry init命令初始化项目，poetry add命令添加依赖包等。
