---
title: python学习笔记8-列表
description: ""
slug: 2021-05-22-python学习笔记8-列表
date: 2021-05-22
image: python.webp
categories:
  - 编程语言
tags:
  - python
  - 列表
  - list
---

### **列表**

> Python中的列表（List）是一种非常强大且灵活的数据结构其他语言中被称为**数组**，它是Python中使用最频繁的数据类型，它允许你存储一个有序的集合。列表可以包含不同类型的元素，比如整数、浮点数、字符串、甚至是其他列表（即列表的嵌套）。 列表用 定义，数据 之间使用 , 分隔。 列表的 索引 从 0 开始， 索引 就是数据在 列表 中的位置编号，索引 又可以被称为 下标。 从列表中取值时，如果 超出索引范围，程序会报错。

### 语法

```python
# 创建一个空列表  
my_list = []
# 使用list关键字创建一个空列表
my_list2 = list()

# 创建一个包含几个整数的列表  
numbers = [1, 2, 3, 4, 5]

# 创建一个包含不同类型元素的列表  
mixed_list = [1, 'a', 3.14, [1, 2, 3]]
```

### 列表的索引

> Python列表（List）是一种非常灵活且功能强大的数据结构，它允许你存储一个有序的元素集合。列表中的每个元素都可以通过索引（Index）来访问，索引是元素在列表中的位置标识。

#### 索引的基本概念

-   **索引值**：索引值用于指定列表中元素的位置。在Python中，索引值从0开始，即列表中的第一个元素索引为0，第二个元素索引为1，依此类推。
-   **正向索引**：从列表的开始（即索引0）到列表的末尾，按照元素在列表中出现的顺序进行索引。
-   **负向索引**：从列表的末尾开始，倒数第一个元素的索引为-1，倒数第二个元素的索引为-2，依此类推。

#### 访问列表元素

你可以使用索引来访问列表中的元素。

```python
# 定义一个列表  
my_list = ['apple', 'banana', 'cherry', 'date']  
  
# 使用正向索引访问元素  
print(my_list[0])  # 输出: apple  
print(my_list[1])  # 输出: banana  
  
# 使用负向索引访问元素  
print(my_list[-1])  # 输出: date  
print(my_list[-2])  # 输出: cherry
```

#### 索引的边界

当你尝试访问一个不存在的索引时（即索引超出了列表的边界），Python会抛出一个`IndexError`异常。

```python
# 尝试访问不存在的索引  
print(my_list[4])  # IndexError: list index out of range  
print(my_list[-5])  # IndexError: list index out of range
```

#### 索引的切片

> 除了使用单个索引来访问元素外，你还可以使用切片（Slicing）来访问列表中的一段元素。Python列表的切片（Slicing）是一种非常强大且灵活的功能，它允许你访问列表中的一部分元素。切片操作通过指定开始索引、结束索引（可选）和步长（可选）来工作。

##### 切片语法

```python
列表名[开始索引:结束索引:步长]
```

-   **开始索引**：切片开始的位置（包含）。如果省略，默认为0。
-   **结束索引**：切片结束的位置（不包含）。如果省略，默认为列表的长度。
-   **步长**：切片时元素之间的间隔。如果省略，默认为1。

##### 切片基本使用

```python
my_list = ['a', 'b', 'c', 'd', 'e', 'f', 'g']  
  
# 访问前三个元素  
print(my_list[:3])  # 输出: ['a', 'b', 'c']  
  
# 访问从索引3到末尾的元素（不包括索引3的元素）  
print(my_list[3:])  # 输出: ['d', 'e', 'f', 'g']  
  
# 访问所有元素（等同于不切片）  
print(my_list[:])  # 输出: ['a', 'b', 'c', 'd', 'e', 'f', 'g']  
  
# 使用步长，每隔一个元素取一个  
print(my_list[::2])  # 输出: ['a', 'c', 'e', 'g']  
  
# 反向切片，从末尾开始到开头  
print(my_list[::-1])  # 输出: ['g', 'f', 'e', 'd', 'c', 'b', 'a']
```

##### 切片进阶使用

###### 负数索引与切片

负数索引允许你从列表的末尾开始访问元素。

```python
# 使用负数索引进行切片  
print(my_list[-3:])  # 输出: ['e', 'f', 'g']  
print(my_list[-4:-1])  # 输出: ['d', 'e', 'f']  
print(my_list[-1::-1])  # 输出: ['g', 'f', 'e', 'd', 'c', 'b', 'a']，反向切片到第一个元素
```

###### 步长为负数

当步长为负数时，切片会从右向左进行。

```python
# 步长为负数  
print(my_list[5:1:-1])  # 输出: ['f', 'e', 'd']  
# 注意：当步长为负数时，如果start大于stop，切片会正常工作；如果start小于stop，切片将返回空列表  
print(my_list[1:5:-1])  # 输出: []
```

###### 切片赋值

切片不仅可以用来访问列表的一部分，还可以用来修改这部分内容。

```python
# 切片赋值  
my_list[1:4] = ['B', 'C', 'D']  
print(my_list)  # 输出: ['a', 'B', 'C', 'D', 'e', 'f', 'g']  
  
# 替换为更短的列表  
my_list[1:4] = ['X']  
print(my_list)  # 输出: ['a', 'X', 'e', 'f', 'g']  
  
# 替换为更长的列表  
my_list[1:2] = ['Y', 'Z']  
print(my_list)  # 输出: ['a', 'Y', 'Z', 'e', 'f', 'g']
```

###### 切片与列表推导式结合

切片可以与列表推导式结合使用，以创建新的列表或对现有列表进行转换。

```python
# 使用切片和列表推导式  
squared = [x**2 for x in my_list[:3]]  # 对前三个元素求平方  
print(squared)  # 输出: [1, 4, 9]  
  
# 过滤和转换  
filtered_and_squared = [x**2 for x in my_list if x.isalpha() and x.islower()][:3]  
# 注意：这里的切片是在列表推导式生成的列表上进行的  
print(filtered_and_squared)  # 输出可能因my_list内容而异，例如: ['a', 'b', 'c']的平方，但只取前三个符合条件的
```

###### 反转列表

使用切片和步长为负数来反转列表。

```python
my_list = [1, 2, 3, 4, 5]  
reversed_list = my_list[::-1]  
print(reversed_list)  # 输出: [5, 4, 3, 2, 1]
```

###### 列表去重（不考虑顺序）

虽然切片本身不直接用于去重，但我们可以结合集合（Set）和切片来实现去重（注意这会丢失原始顺序）。

```python
my_list = [1, 2, 2, 3, 4, 4, 5]  
unique_list = list(dict.fromkeys(my_list))  # 使用字典的键来去重，但会丢失顺序  
# 或者使用列表推导式和集合（保持顺序需要更复杂的方法）  
unique_list_ordered = []  
[unique_list_ordered.append(x) for x in my_list if x not in unique_list_ordered]  
print(unique_list_ordered)  # 输出: [1, 2, 3, 4, 5]
```

### 列表的+和\*运算

#### 列表的“加法”操作：拼接

在Python中，列表的“加法”操作实际上是通过拼接（Concatenation）来实现的，即使用`+`操作符将两个或多个列表合并成一个新的列表。

```python
list1 = [1, 2, 3]  
list2 = [4, 5, 6]  
  
# 使用+操作符拼接列表  
result = list1 + list2  
print(result)  # 输出: [1, 2, 3, 4, 5, 6]
```

#### 列表的“乘法”操作：重复

列表的“乘法”操作是通过重复列表中的元素来实现的，使用`*`操作符。这里的“乘法”并不是传统意义上的算术乘法，而是将列表重复指定的次数。

```python
list1 = [1, 2, 3]  
  
# 使用*操作符重复列表  
result = list1 * 3  
print(result)  # 输出: [1, 2, 3, 1, 2, 3, 1, 2, 3]
```

#### 使用列表推导式进行条件拼接

虽然列表推导式不是算术运算，但它可以用于根据条件拼接列表元素，实现更复杂的逻辑。

```python
list1 = [1, 2, 3, 4]  
list2 = [5, 6, 7, 8]  
  
# 使用列表推导式根据条件拼接列表  
# 例如，只拼接两个列表中对应位置上的偶数  
result = [x if x % 2 == 0 else y for x, y in zip(list1, list2) if x % 2 == 0 or y % 2 == 0]  
# 注意：这个示例实际上没有直接拼接两个列表，而是根据条件选择了元素  
# 如果要直接拼接偶数，可以这样做：  
even_result = [x for pair in zip(list1, list2) for x in pair if x % 2 == 0]  
# 但这会丢失原始列表的结构，因为zip会同时迭代两个列表  
  
# 更简单的偶数拼接（不考虑对应位置）  
even_list1 = [x for x in list1 if x % 2 == 0]  
even_list2 = [y for y in list2 if y % 2 == 0]  
result_even = even_list1 + even_list2  
print(result_even)  # 输出可能是 [2, 4, 6, 8]，具体取决于list1和list2的内容
```

#### 使用`itertools.chain`进行链式拼接

对于更复杂的拼接场景，特别是当你有多个列表需要拼接时，可以使用`itertools.chain`来更优雅地实现。

```python
from itertools import chain  
  
list1 = [1, 2, 3]  
list2 = [4, 5, 6]  
list3 = [7, 8, 9]  
  
# 使用itertools.chain进行链式拼接  
result = list(chain(list1, list2, list3))  
print(result)  # 输出: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### 注意事项

-   列表的“加法”和“乘法”操作与数值类型的算术运算有本质区别。
-   列表拼接和重复操作不会修改原始列表，而是返回一个新的列表。
-   对于更复杂的列表操作，如条件拼接或链式拼接，可能需要使用列表推导式、`itertools`模块等工具。

### 列表的判断

> Python列表的判断通常指的是对列表中的元素进行条件检查，以决定执行哪些操作。这包括检查列表是否为空、检查列表中是否包含特定元素、检查列表中所有元素是否满足某个条件等。

#### 判断是否包含指定内容

```python
#定义一个列表
lis = ['zhangsan','lisi','wangwu']
#判断列表中是否包含 zhangsan这个元素
if 'zhangsan' in lis:
    print('包含')
else:
    print('不包含')
#输出：包含
```

#### 检查列表是否为空

```python
# 初始化一个空列表  
my_list = []  
  
# 检查列表是否为空  
# 由于my_list是空的，所以条件不满足，执行else分支  
if my_list:  
    print("列表不为空")  # 这行代码不会执行  
else:  
    print("列表为空")  # 这行代码会执行，输出“列表为空”  
  
# 填充列表后再次检查  
# 向my_list中添加元素1, 2, 3  
my_list = [1, 2, 3]  
  
# 再次检查列表是否为空  
# 由于my_list现在不是空的，所以条件满足，执行if分支  
if my_list:  
    print("列表不为空")  # 这行代码会执行，输出“列表不为空”  
else:  
    print("列表为空")  # 这行代码不会执行
```

这段代码的主要目的是演示如何使用`if`语句来检查Python列表是否为空。首先，它初始化了一个空列表`my_list`，并使用`if`语句检查该列表是否为空（即是否包含任何元素）。由于列表是空的，所以执行了`else`分支，输出了“列表为空”。

然后，代码通过向`my_list`中添加元素（1, 2, 3）来填充列表，并再次使用`if`语句检查列表是否为空。这次，由于列表不再为空，条件满足，因此执行了`if`分支，输出了“列表不为空”。

#### 检查列表中是否存在满足某个条件的元素

使用`any()`函数可以检查列表中是否存在至少一个满足条件的元素。

```python
# 初始化一个包含整数的列表  
my_list = [1, 2, 3, 4, 5]  
  
# 检查列表中是否存在偶数  
# 使用any()函数结合生成器表达式来检查列表中是否有元素满足条件x % 2 == 0（即是否为偶数）  
# 如果有任何一个元素满足条件，any()返回True，否则返回False  
if any(x % 2 == 0 for x in my_list):  
    print("列表中存在偶数")  # 因为列表中有2和4，所以输出这句话  
else:  
    print("列表中不存在偶数")  # 这行代码不会执行  
  
# 检查列表中是否存在大于5的元素  
# 类似地，使用any()函数结合生成器表达式来检查列表中是否有元素满足条件x > 5  
# 如果有任何一个元素满足条件，any()返回True，否则返回False  
if any(x > 5 for x in my_list):  
    print("列表中存在大于5的元素")  # 列表中没有元素大于5，所以这行代码不会执行  
else:  
    print("列表中不存在大于5的元素")  # 因为列表中没有大于5的元素，所以输出这句话
```

首先定义了一个包含整数的列表`my_list`。然后，它使用`any()`函数结合生成器表达式来检查列表中是否存在偶数（即任何元素除以2的余数为0）和是否存在大于5的元素。根据检查结果，它会打印出相应的信息。在这个例子中，列表中存在偶数（2和4），但不存在大于5的元素。

#### 检查列表是否包含重复元素

```python
def has_duplicates(lst):  
    # 使用集合来检查重复元素  
    # 集合是一个无序的不重复元素集，因此通过将列表转换为集合并比较长度，  
    # 如果长度不同，说明原列表中存在重复元素  
    return len(lst) != len(set(lst))    
  
# 初始化一个不包含重复元素的列表  
my_list = [1, 2, 3, 4, 5]  
# 调用has_duplicates函数检查列表是否包含重复元素  
if has_duplicates(my_list):  
    print("列表包含重复元素")  # 这行代码不会执行，因为列表中没有重复元素  
else:  
    print("列表不包含重复元素")  # 这行代码会执行，输出“列表不包含重复元素”  
  
# 修改列表以包含重复元素  
my_list = [1, 2, 3, 4, 5, 1]  
# 再次调用has_duplicates函数检查修改后的列表是否包含重复元素  
if has_duplicates(my_list):  
    print("列表包含重复元素")  # 这行代码会执行，因为列表中存在重复元素（数字1）  
else:  
    print("列表不包含重复元素")  # 这行代码不会执行
```

定义了一个名为`has_duplicates`的函数，该函数接受一个列表`lst`作为参数，并使用集合来检查列表中是否存在重复元素。它通过比较原列表的长度和将该列表转换为集合后的长度来实现这一点，因为集合不允许重复元素。如果两个长度不相等，说明原列表中存在重复元素，函数返回`True`；否则，返回`False`。

然后，代码使用了一个不包含重复元素的列表`my_list`来测试`has_duplicates`函数，并打印出相应的结果。接着，它修改了`my_list`以包含重复元素，并再次使用`has_duplicates`函数进行检查，最后打印出检查结果。

#### 检查列表是否按升序排列

```python
def is_sorted(lst):  
    # 遍历列表，从第一个元素到倒数第二个元素（因为我们要比较相邻的元素）  
    for i in range(len(lst) - 1):  
        # 如果当前元素大于其后一个元素，说明列表未按升序排列  
        if lst[i] > lst[i + 1]:  
            return False  # 返回False，表示列表未按升序排列  
    return True  # 遍历完所有相邻元素对后，如果没有发现逆序对，则返回True，表示列表已按升序排列  
  
# 初始化一个按升序排列的列表  
my_list = [1, 2, 3, 4, 5]  
# 调用is_sorted函数检查列表是否已按升序排列  
if is_sorted(my_list):  
    print("列表已按升序排列")  # 输出“列表已按升序排列”  
else:  
    print("列表未按升序排列")  # 这行代码不会执行  
  
# 修改列表以不按升序排列  
my_list = [5, 4, 3, 2, 1]  
# 再次调用is_sorted函数检查修改后的列表是否已按升序排列  
if is_sorted(my_list):  
    print("列表已按升序排列")  # 这行代码不会执行  
else:  
    print("列表未按升序排列")  # 输出“列表未按升序排列”
```

定义了一个名为`is_sorted`的函数，用于检查传入的列表是否已按升序排列。它通过遍历列表中的每个元素（除了最后一个），并比较每个元素与其后一个元素的大小来实现。如果在遍历过程中发现任何一对相邻元素是逆序的（即前一个元素大于后一个元素），则函数立即返回`False`，表示列表未按升序排列。如果遍历完所有相邻元素对后没有发现逆序对，则函数返回`True`，表示列表已按升序排列。

然后，代码通过两个示例（一个已按升序排列的列表和一个未按升序排列的列表）来演示`is_sorted`函数的使用。

### 列表的遍历

> 在Python中，列表（List）是一种非常常用的数据结构，它支持多种遍历方式。遍历列表意味着按顺序访问列表中的每一个元素。

#### 基本的for循环遍历

这是最直接、最常用的遍历列表的方式。

```python
# 定义一个列表  
my_list = [1, 2, 3, 4, 5]  
  
# 使用for循环遍历列表  
for item in my_list:  # 遍历列表中的每个元素  
    print(item)  # 打印当前元素  
  
# 运行结果：  
# 1  
# 2  
# 3  
# 4  
# 5
```

#### 使用索引遍历

```python
# 定义一个列表  
my_list = ['a', 'b', 'c', 'd', 'e']  
  
# 使用索引遍历列表  
for index in range(len(my_list)):  # 遍历从0到列表长度减1的索引  
    print(f"Index: {index}, Value: {my_list[index]}")  # 打印索引和对应的值  
  
# 运行结果：  
# Index: 0, Value: a  
# Index: 1, Value: b  
# Index: 2, Value: c  
# Index: 3, Value: d  
# Index: 4, Value: e
```

#### 使用`enumerate()`遍历

```python
# 定义一个列表  
my_list = ['apple', 'banana', 'cherry']  
  
# 使用enumerate()遍历列表，同时获取索引和值  
for index, fruit in enumerate(my_list):  # enumerate返回索引和值的元组  
    print(f"Index {index}: {fruit}")  # 打印索引和对应的值  
  
# 运行结果：  
# Index 0: apple  
# Index 1: banana  
# Index 2: cherry
```

#### 使用`map()`函数遍历

```python
# 定义一个列表  
my_list = [1, 2, 3, 4, 5]  
  
# 定义一个函数，用于平方  
def square(x):  
    return x**2  
  
# 使用map()函数对列表中的每个元素应用square函数  
squared_list = list(map(square, my_list))  # map返回迭代器，需要转换为列表  
print(squared_list)  
  
# 运行结果：  
# [1, 4, 9, 16, 25]  
  
# 进阶案例：结合lambda表达式  
squared_list_lambda = list(map(lambda x: x**2, my_list))  
print(squared_list_lambda)  
  
# 运行结果：  
# [1, 4, 9, 16, 25]
```

#### 使用`filter()`函数遍历

```python
# 定义一个列表  
my_list = [1, 2, 3, 4, 5, 6]  
  
# 定义一个函数，用于判断偶数  
def is_even(x):  
    return x % 2 == 0  
  
# 使用filter()函数筛选列表中的偶数  
even_list = list(filter(is_even, my_list))  # filter返回迭代器，需要转换为列表  
print(even_list)  
  
# 运行结果：  
# [2, 4, 6]  
  
# 进阶案例：结合lambda表达式  
even_list_lambda = list(filter(lambda x: x % 2 == 0, my_list))  
print(even_list_lambda)  
  
# 运行结果：  
# [2, 4, 6]
```

### 列表的常用函数

|    |           |                                   |
|-----|-----------|-----------------------------------|
| 新增 | append()  |           在列表末尾添加一个新的元素           |
|   | extend()  | 在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表） |
|   | insert()  |            在指定位置插入一个元素            |
| 删除 | remove()  |          移除列表中某个值的第一个匹配项          |
|   |   pop()   |   移除列表中的一个元素（默认最后一个元素），并返回该元素的值   |
|   |    del    |    使用del语句可以删除列表中的单个元素或者一个元素范围    |
|   |  clear()  |            移除列表中的所有元素             |
| 修改 |   索引赋值    |          通过索引直接修改列表中的元素值          |
|   |  slice赋值  |        通过切片操作修改列表中的一个元素范围         |
| 统计 |  count()  |           返回列表中元素出现的次数            |
|   |   len()   |            返回列表中元素的个数             |
| 排序 |  sort()   |           对列表中的元素进行就地排序           |
|   | sorted()  |       对可迭代对象进行排序，并返回一个新的列表        |
|   | reverse() |             反转列表中的元素              |

#### 列表-新增

> 在Python中，列表（List）是一种非常灵活的数据结构，它允许你存储一系列的元素，并且这些元素可以是不同类型的。对于列表的新增操作，主要有三种方法：`append()`、`extend()` 和 `insert()`。

##### append()

`append()` 方法用于在列表的末尾添加一个元素。它只接受一个参数，即要添加到列表末尾的元素。

```python
# 初始化一个列表  
my_list = [1, 2, 3]  
  
# 使用 append() 方法在列表末尾添加一个元素  
my_list.append(4)  # 将4添加到my_list的末尾  
  
# 打印修改后的列表  
print("After appending 4:", my_list)  # 输出: After appending 4: [1, 2, 3, 4]
```

##### extend()

`extend()` 方法用于在列表的末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表）。它接受一个可迭代对象（如列表、元组、集合等）作为参数，并将该可迭代对象中的每个元素添加到原列表的末尾。

```python
# 初始化一个列表  
my_list = [1, 2, 3]  
  
# 创建一个要添加到my_list末尾的列表  
other_list = [4, 5, 6]  
  
# 使用 extend() 方法将 other_list 中的所有元素添加到 my_list 的末尾  
my_list.extend(other_list)  # 将[4, 5, 6]添加到my_list的末尾  
  
# 打印修改后的列表  
print("After extending with [4, 5, 6]:", my_list)  # 输出: After extending with [4, 5, 6]: [1, 2, 3, 4, 5, 6]
```

##### insert()

`insert()` 方法用于将指定元素插入到列表的指定位置。它接受两个参数：第一个是索引（指定插入位置），第二个是要插入的元素。请注意，如果索引超出列表长度，该元素将被添加到列表的末尾。

```python
# 初始化一个列表  
my_list = [1, 2, 4, 5]  
  
# 使用 insert() 方法在索引为2的位置插入元素3  
# 注意：索引从0开始，所以这里的2指的是列表中的第三个位置（在4之前）  
my_list.insert(2, 3)  # 在索引为2的位置插入3  
  
# 打印修改后的列表  
print("After inserting 3 at index 2:", my_list)  # 输出: After inserting 3 at index 2: [1, 2, 3, 4, 5]  
  
# 尝试在超出列表长度的索引处插入元素  
my_list.insert(len(my_list), 6)  # 实际上这等同于在列表末尾追加元素  
print("After inserting 6 at the end:", my_list)  # 输出: After inserting 6 at the end: [1, 2, 3, 4, 5, 6]
```

#### 列表-删除

> 在Python中，列表（List）是一种非常灵活的数据结构，它支持多种删除元素的方式。以下是关于列表删除操作的详细解释，包括`remove()`、`pop()`、`del`语句以及`clear()`方法的知识点 **这些操作都是就地修改列表的，即它们会直接修改原列表而不是返回一个新的列表。在删除元素时，特别是使用\*\***`remove()`\***\*和\*\***`pop()`\***\*时，需要注意元素的存在性和索引的有效性，以避免出现错误。**

##### remove()

`remove()` 方法用于移除列表中第一个匹配指定值的元素。如果列表中不存在该元素，则抛出`ValueError`异常。

```python
# 初始化一个列表  
my_list = [1, 2, 3, 4, 3, 5]  
  
# 使用 remove() 方法移除列表中第一个值为3的元素  
my_list.remove(3)  # 移除第一个3  
  
# 打印修改后的列表  
print("After removing the first 3:", my_list)  # 输出: After removing the first 3: [1, 2, 4, 3, 5]  
  
# 尝试移除一个不存在的元素  
# try:  
#     my_list.remove(6)  # 这将抛出ValueError  
# except ValueError:  
#     print("Value 6 not found in the list")
```

##### pop()

`pop()` 方法用于移除列表中的一个元素（默认是最后一个元素），并返回该元素的值。可以指定要移除元素的索引。

```python
# 初始化一个列表  
my_list = [1, 2, 3, 4, 5]  
  
# 使用 pop() 方法移除并返回最后一个元素  
last_element = my_list.pop()  # 默认移除最后一个元素  
  
# 打印修改后的列表和移除的元素  
print("After popping the last element:", my_list)  # 输出: After popping the last element: [1, 2, 3, 4]  
print("Popped element:", last_element)  # 输出: Popped element: 5  
  
# 使用索引移除特定位置的元素  
second_element = my_list.pop(1)  # 移除索引为1的元素（即第二个元素）  
  
# 打印修改后的列表和移除的元素  
print("After popping the second element:", my_list)  # 输出: After popping the second element: [1, 3, 4]  
print("Popped element:", second_element)  # 输出: Popped element: 2
```

##### del

`del` 语句用于删除列表中的元素，可以是整个列表，也可以是列表中的特定元素（通过索引）或切片。

```python
# 初始化一个列表  
my_list = [1, 2, 3, 4, 5]  
  
# 使用 del 语句删除特定索引的元素  
del my_list[2]  # 删除索引为2的元素（即第三个元素）  
  
# 打印修改后的列表  
print("After deleting the third element:", my_list)  # 输出: After deleting the third element: [1, 2, 4, 5]  
  
# 使用 del 语句删除切片  
del my_list[1:3]  # 删除索引从1到2的元素（不包括3）  
  
# 打印修改后的列表  
print("After deleting a slice:", my_list)  # 输出: After deleting a slice: [1, 5]  
  
# 使用 del 语句删除整个列表（通常不这么做，仅作为示例）  
# del my_list  
# print(my_list)  # 这将引发NameError，因为my_list已被删除
```

##### clear()

`clear()` 方法用于删除列表中的所有元素，使其变为空列表。

```python
# 初始化一个列表  
my_list = [1, 2, 3, 4, 5]  
  
# 使用 clear() 方法删除列表中的所有元素  
my_list.clear()  
  
# 打印修改后的列表  
print("After clearing the list:", my_list)  # 输出: After clearing the list: []
```

#### 列表-修改

> 在Python中，列表（List）是一种可变的数据结构，意味着你可以直接修改列表中的元素而不需要创建新的列表。列表的修改通常包括通过索引直接赋值来修改特定位置的元素，以及通过切片赋值来修改列表的多个元素。

##### 索引赋值

索引赋值是指通过指定元素的索引来修改列表中该位置的元素。这是修改列表中单个元素最直接的方法。

```python
# 初始化一个列表  
my_list = [1, 2, 3, 4, 5]  
  
# 使用索引赋值修改列表中的第三个元素（索引为2）  
my_list[2] = 30  # 将索引为2的元素修改为30  
  
# 打印修改后的列表  
print("After modifying the third element:", my_list)  # 输出: After modifying the third element: [1, 2, 30, 4, 5]  
  
# 尝试修改一个不存在的索引的元素将会抛出IndexError  
# try:  
#     my_list[5] = 6  # 这将抛出IndexError  
# except IndexError:  
#     print("Index 5 does not exist in the list")
```

##### 切片赋值

切片赋值允许你通过指定一个切片（即列表的一个子序列）来修改列表中的多个元素。你可以将整个切片替换为一个新的可迭代对象（如列表、元组等），或者为空来删除该切片中的所有元素。

###### 替换切片中的元素

```python
# 初始化一个列表  
my_list = [1, 2, 3, 4, 5]  
  
# 使用切片赋值替换列表中第三个到第四个元素（索引为2到3）  
my_list[2:4] = [30, 40]  # 将索引为2到3的元素替换为[30, 40]  
  
# 打印修改后的列表  
print("After replacing a slice:", my_list)  # 输出: After replacing a slice: [1, 2, 30, 40, 5]
```

###### 使用空切片删除元素

```python
# 初始化一个列表  
my_list = [1, 2, 3, 4, 5]  
  
# 使用切片赋值和空列表来删除列表中第三个到第四个元素（索引为2到3）  
my_list[2:4] = []  # 删除索引为2到3的元素  
  
# 打印修改后的列表  
print("After deleting a slice:", my_list)  # 输出: After deleting a slice: [1, 2, 5]
```

#### 列表-统计

> 在Python中，列表（List）是一种非常灵活的数据结构，它提供了多种统计功能，以便我们了解列表中的元素数量、特定元素的出现次数等。

-   `count()` 方法是列表的一个非常有用的方法，它允许我们快速统计列表中某个元素出现的次数
-   `len()` 函数虽然不是列表的专属，但它经常用于获取列表的长度，即列表中元素的数量。

##### count() 方法

`count()` 方法用于统计列表中某个元素出现的次数。它接受一个参数，即要统计的元素，并返回该元素在列表中出现的次数。

```python
# 初始化一个列表  
my_list = [1, 2, 2, 3, 4, 4, 4, 5]  
  
# 使用 count() 方法统计元素 2 出现的次数  
count_of_two = my_list.count(2)  
  
# 打印结果  
print("The number of 2s in the list:", count_of_two)  # 输出: The number of 2s in the list: 2  
  
# 统计元素 4 出现的次数  
count_of_four = my_list.count(4)  
  
# 打印结果  
print("The number of 4s in the list:", count_of_four)  # 输出: The number of 4s in the list: 3  
  
# 尝试统计一个不存在的元素  
count_of_six = my_list.count(6)  
  
# 打印结果  
print("The number of 6s in the list:", count_of_six)  # 输出: The number of 6s in the list: 0
```

##### len() 函数

虽然`len()`不是列表的方法，而是Python的内置函数，但它经常用于获取列表（以及其他可迭代对象）的长度，即列表中元素的数量。

```python
# 初始化一个列表  
my_list = [1, 2, 3, 4, 5]  
  
# 使用 len() 函数获取列表的长度  
length_of_list = len(my_list)  
  
# 打印结果  
print("The length of the list:", length_of_list)  # 输出: The length of the list: 5  
  
# 修改列表并再次获取长度  
my_list.append(6)  # 向列表中添加一个新元素  
new_length_of_list = len(my_list)  
  
# 打印结果  
print("The new length of the list:", new_length_of_list)  # 输出: The new length of the list: 6
```

#### 列表-排序

> 在Python中，列表（List）的排序是一个常见且重要的操作。Python提供了几种不同的方法来实现列表的排序，包括`sort()`方法、`sorted()`函数以及`reverse()`方法。

-   `sort()` 方法用于就地排序列表，支持升序和降序。
-   `sorted()` 函数返回一个新的列表，而不是修改原始列表，也支持升序、降序以及自定义排序逻辑。
-   `reverse()` 方法反转列表中的元素顺序，虽然不是直接的排序方法，但在某些场景下可以作为排序的补充。

##### sort() 方法

`sort()` 方法是列表的一个方法，它会就地（in-place）对列表进行排序，即直接修改原列表，而不是返回一个新的列表。默认情况下，`sort()` 会按照元素的升序排列，但你也可以通过参数来指定其他排序方式。

```python
# 初始化一个未排序的列表  
my_list = [3, 1, 4, 1, 5, 9, 2]  
  
# 使用 sort() 方法对列表进行升序排序  
my_list.sort()  
  
# 打印排序后的列表  
print("Sorted list (ascending):", my_list)  # 输出: Sorted list (ascending): [1, 1, 2, 3, 4, 5, 9]  
  
# 使用 sort() 方法进行降序排序，需要指定 reverse=True  
my_list.sort(reverse=True)  
  
# 打印降序排序后的列表  
print("Sorted list (descending):", my_list)  # 输出: Sorted list (descending): [9, 5, 4, 3, 2, 1, 1]
```

##### sorted() 函数

与`sort()`方法不同，`sorted()`函数会对可迭代对象进行排序，并返回一个新的列表，而不会修改原始的可迭代对象。`sorted()`函数同样支持`reverse`参数以及`key`函数来自定义排序逻辑。

```python
# 初始化一个未排序的列表  
my_list = [3, 1, 4, 1, 5, 9, 2]  
  
# 使用 sorted() 函数对列表进行升序排序，返回新列表  
sorted_list = sorted(my_list)  
  
# 打印排序后的新列表  
print("Sorted new list (ascending):", sorted_list)  # 输出: Sorted new list (ascending): [1, 1, 2, 3, 4, 5, 9]  
  
# 原始列表未被修改  
print("Original list:", my_list)  # 输出: Original list: [3, 1, 4, 1, 5, 9, 2]  
  
# 使用 sorted() 函数进行降序排序  
sorted_list_desc = sorted(my_list, reverse=True)  
  
# 打印降序排序后的新列表  
print("Sorted new list (descending):", sorted_list_desc)  # 输出: Sorted new list (descending): [9, 5, 4, 3, 2, 1, 1]
```

##### reverse() 方法

虽然`reverse()`方法不是直接用于排序的，但它可以反转列表中的元素顺序，这在某些情况下可能是你想要的排序效果（尤其是当你已经有一个有序的列表，但需要反向顺序时）。

```python
# 初始化一个已排序的列表  
my_list = [1, 2, 3, 4, 5]  
  
# 使用 reverse() 方法反转列表  
my_list.reverse()  
  
# 打印反转后的列表  
print("Reversed list:", my_list)  # 输出: Reversed list: [5, 4, 3, 2, 1]
```

### 二维列表

> 在Python中，二维列表（也称为列表的列表）是一种非常有用的数据结构，它允许你存储和操作表格状的数据。二维列表可以看作是多个一维列表（即普通列表）的集合，其中每个一维列表都是二维列表的一个“行”。

#### 创建二维列表

二维列表可以通过多种方式创建，但最直接的方式是嵌套列表字面量。

```python
# 直接创建二维列表  
matrix = [  
    [1, 2, 3],  
    [4, 5, 6],  
    [7, 8, 9]  
]  
  
# 通过循环创建二维列表  
rows, cols = 3, 3  
matrix_by_loop = [[0 for _ in range(cols)] for _ in range(rows)]  
print(matrix_by_loop)  
# 输出: [[0, 0, 0], [0, 0, 0], [0, 0, 0]]
```

#### 访问二维列表的元素

要访问二维列表中的元素，你需要指定两个索引：第一个索引对应于行，第二个索引对应于列。假设你有一个名为`matrix`的二维列表，你可以这样访问它的元素：

```python
matrix = [  
    [1, 2, 3],  
    [4, 5, 6],  
    [7, 8, 9]  
]  
  
# 访问第一行第二列的元素（索引从0开始）  
print(matrix[0][1])  # 输出: 2  
  
# 访问第二行第三列的元素  
print(matrix[1][2])  # 输出: 6
```

#### 遍历二维列表

遍历二维列表（即“二维表”）通常需要使用嵌套循环。外层循环遍历行，内层循环遍历列。

```python
# 遍历二维列表  
for row in matrix:  
    for item in row:  
        print(item, end=' ')  
    print()  # 每遍历完一行后换行  
  
# 输出:  
# 1 2 3   
# 4 5 6   
# 7 8 9
```

#### 修改二维列表的元素

修改二维列表中的元素与访问元素类似，也是通过指定行和列的索引来进行。

```python
# 修改第二行第一列的元素为10  
matrix[1][0] = 10  
  
# 再次遍历以查看修改结果  
for row in matrix:  
    for item in row:  
        print(item, end=' ')  
    print()  
  
# 输出:  
# 1 2 3   
# 10 5 6   
# 7 8 9
```

#### 二维列表的切片

虽然二维列表的切片不像一维列表那样直观，但你可以对每一行（即外层列表的每个元素）进行切片操作。然而，直接对整个二维列表进行切片会得到一个子二维列表，而不是单独的行或列。

```python
# 获取第一行和第二行  
sub_matrix = matrix[0:2]  
print(sub_matrix)  
# 输出: [[1, 2, 3], [10, 5, 6]]  
  
# 注意：直接对二维列表进行列切片并不直接支持，但可以通过列表推导式或循环来实现  
# 获取所有行的第一列  
first_column = [row[0] for row in matrix]  
print(first_column)  
# 输出: [1, 10, 7]
```