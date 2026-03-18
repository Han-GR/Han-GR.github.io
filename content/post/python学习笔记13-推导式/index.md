---
title: python学习笔记13-推导式
description: ""
slug: 2021-06-10-python学习笔记13-推导式
date: 2021-06-10
image: python.webp
categories:
  - 编程语言
tags:
  - python
  - 推导式
---
### 一、列表推导（List Comprehension）

Python中的列表推导（List Comprehension）是一种简洁的构建列表的方法。它允许你从一个已存在的列表（或其他可迭代对象）中快速生成一个新的列表，而不需要编写完整的循环结构。列表推导不仅可以使代码更加简洁，而且在某些情况下还可以提高执行效率。

##### 基本语法：


```python
[expression for item in iterable]
```

-   `expression` 是对每一个 `item` 进行处理的表达式，其结果将被添加到新列表中。
-   `item` 是可迭代对象 `iterable` 中的元素。
-   `iterable` 是一个可以迭代的对象，比如列表、元组、字符串、集合等。

#### 1. 简单的列表推导

假设我们有一个数字列表，想要创建一个新列表，其中包含原列表中每个元素的平方。

使用for循环的实现方法,这样看起来比较繁琐，而且效率比较低

```python
numbers = [1, 2, 3, 4, 5]
numbers2 = []
for num in numbers:
    numbers2.append(num**2)
print(numbers2) # [1, 4, 9, 16, 25]
```

使用列表推导式,这样的代码看起来就非常的整洁，效率也比较好

```python
numbers = [1, 2, 3, 4, 5]
#首先遍历numbers数组，然后将每一个元素赋值给x，x**2，就是numbers数组中每一个元素**2
numbers2 = [x**2 for x in numbers]
print(numbers2) # [1, 4, 9, 16, 25]
```

#### 2. 带条件的列表推导

可以在列表推导式中加入条件，满足条件的数据才会进入到最终的列表里面

根据numbers列表使用列表推导式生成一个新列表，但是只获取偶数

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
'''
首先先看正常的列表推导式
x for x in numbers
先获取到numbers数组，循环遍历numbers数组，将数组中的每个元素赋值给x
然后x不进行任何操作

接下来就看判断条件
if x % 2 == 0
会判断每一个x，也就是数组中的每一个元素，只有条件为true的元素才会进入到最终的数组中
'''
numbers2 = [x for x in numbers if x % 2 == 0]
print(numbers2) # [2, 4, 6, 8, 10]
```

#### 3. 嵌套列表推导

列表推导也可以嵌套，以处理更复杂的数据结构。

使用列表推导式获取二维数组中的所有一维数组的元素,并将每个元素加1然后写入到一个新一维数组中

```python
#定义一个二维数组
numbers = [
    [1,2,3],
    [4,5,6],
    [7,8,9]
]
'''
首先先看 第一个循环
for num in numbers
这段语句就是获取到numbers这个二维数组中的每一个元素，也就是一维数组

然后再看 第二个循环
for x in num
第一个循环把获取到的每一个一维数组赋值给num，现在循环遍历num将每个元素赋值给x

然后x+1,就是每个元素加1，然后赋值给最终的数组
'''
numbers2 = [x+1 for num in numbers for x in num]
print(numbers2) # [2, 3, 4, 5, 6, 7, 8, 9, 10]
```

在嵌套列表推导中，第一个 `for` 循环遍历外层的可迭代对象（在这个例子中是 `numbers` 的每一行），而第二个 `for` 循环遍历内层的可迭代对象（在这个例子中是每行的元素）。

#### 二、集合推导（Set Comprehension）

在Python中，集合推导（Set Comprehension）是一种简洁且强大的工具，用于从一个或多个迭代器快速创建集合（set）。它类似于列表推导（List Comprehension），但结果是一个集合，这意味着结果中的元素是唯一的，且顺序不保证。集合推导的基本语法遵循了集合的特性，即不允许重复元素，并且元素是无序的。

##### 基本语法：

```python
{expression for item in iterable}
```

-   `expression`：是一个表达式，用于从每个`item`中生成新的元素。
-   `item`：是迭代器`iterable`中的当前元素。
-   `iterable`：是一个可迭代对象，如列表、元组、字符串、字典等。
-   `condition`（可选）：是一个条件表达式，用于筛选满足条件的元素。如果条件为真，则当前元素`item`会被包含在结果集合中。

##### 1. 基本集合推导

```python
# 创建一个包含1到10的平方数的集合  
squared = {x**2 for x in range(1, 11)}  
print(squared)  # 输出可能是 {1, 4, 9, 16, 25, 36, 49, 64, 81, 100}，注意顺序可能不同
```

##### 2. 带条件的集合推导

```python
# 创建一个包含1到10之间偶数的平方的集合  
even_squared = {x**2 for x in range(1, 11) if x % 2 == 0}  
print(even_squared)  # 输出可能是 {4, 16, 36, 64, 100}，注意顺序可能不同
```

##### 3. 从字典推导集合

```python
# 假设我们有一个字典，我们想从它的键中创建一个集合  
d = {'a': 1, 'b': 2, 'c': 3, 'd': 4}  
keys_set = {key for key in d}  
print(keys_set)  # 输出可能是 {'a', 'b', 'c', 'd'}，注意顺序可能不同
```

##### 4. 嵌套集合推导

虽然嵌套集合推导不常见，但它们也是可能的。但请注意，这可能会导致代码难以理解和维护。

```python
# 假设我们有两个列表，我们想找出所有可能的（x, y）对，其中x来自第一个列表，y来自第二个列表  
list1 = [1, 2]  
list2 = [3, 1, 4]  
pairs = {(x, y) for x in list1 for y in list2}  
print(pairs)  # 输出可能是 {(1, 3), (1, 1), (1, 4), (2, 3), (2, 1), (2, 4)}，注意顺序可能不同
```

#### 三、字典推导（Dictionary Comprehension）

Python中的字典推导（Dictionary Comprehension）是一种简洁而强大的方式，用于从可迭代对象（如列表、元组或其他可迭代对象）中创建字典。字典推导类似于列表推导（List Comprehension），但结果是一个字典而不是列表。

##### 基本语法


```python
{key: value for (key, value) in iterable}
```

##### 1. 简单的字典推导

假设我们有一个包含元组的列表，每个元组代表一个名字和对应的年龄，我们想将这些数据转换为一个字典：

```python
people = [('Alice', 30), ('Bob', 25), ('Charlie', 35)]  
  
# 使用字典推导创建字典  
people_dict = {name: age for name, age in people}  
  
print(people_dict)  
# 输出: {'Alice': 30, 'Bob': 25, 'Charlie': 35}
```

##### 2. 包含条件的字典推导

如果我们只想包含年龄大于30岁的人，可以添加条件表达式：

```python
people = [('Alice', 30), ('Bob', 25), ('Charlie', 35)]  
  
# 使用字典推导，并添加条件  
people_over_30 = {name: age for name, age in people if age > 30}  
  
print(people_over_30)  
# 输出: {'Charlie': 35}
```

##### 3. 复杂的键或值计算

假设我们有一个包含员工ID和姓名的列表，但我们想要将员工ID作为键，并将“Employee\_”前缀添加到姓名作为值：

```python
employees = [(1, 'Alice'), (2, 'Bob'), (3, 'Charlie')]  
  
# 使用字典推导，并计算键和值  
employee_dict = {id: 'Employee_' + name for id, name in employees}  
  
print(employee_dict)  
# 输出: {1: 'Employee_Alice', 2: 'Employee_Bob', 3: 'Employee_Charlie'}
```

##### 4. 嵌套循环和条件

虽然字典推导中直接使用嵌套循环不是直接支持的（因为字典的键必须是唯一的），但你可以通过其他方式（如列表推导或生成器表达式）来间接实现嵌套逻辑，并在字典推导中使用其结果。不过，对于简单的场景，通常建议避免在字典推导中进行过于复杂的嵌套或条件逻辑，以保持代码的可读性。

