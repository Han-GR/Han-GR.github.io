---
title: python学习笔记12-枚举
description: ""
slug: 2021-06-05-python学习笔记12-枚举
date: 2021-06-05
image: python.webp
categories:
  - 编程语言
tags:
  - python
  - 枚举
  - enumerate
---
### 一、`enumerate的作用`

`enumerate()` 是 Python 中的一个内置函数，它用于将一个可遍历的数据对象（如列表、元组或字符串）组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。`enumerate()` 函数返回一个枚举对象，该对象是一个迭代器，它生成由 `(index, value)` 对组成的元组，其中 `index` 是从 `start`（默认为 0）开始计数的索引，`value` 是从输入的可迭代对象中获取的值。

1.  **索引和值同时获取**：在处理列表或元组时，经常需要同时访问元素的索引和值。使用 `enumerate()` 可以很容易地实现这一点，而无需使用额外的索引变量或计数器。
2.  **简化代码**：在遍历过程中，如果你需要知道当前元素的索引，你可能会使用 `range()` 函数来生成索引，并同时遍历可迭代对象。这通常需要两个独立的变量和一个额外的 `zip()` 或其他方法来同步索引和值。使用 `enumerate()` 可以更简洁地完成这项工作。
3.  **排序和映射**：在需要对数据进行排序或映射时，知道元素的索引可能很重要。例如，你可能想要根据值对列表进行排序，但保留原始索引。使用 `enumerate()` 可以轻松实现这一点，然后再根据需要进行排序和映射。
4.  **文件处理**：在处理文件时，`enumerate()` 可以用来追踪当前正在处理的行号。这在记录日志、输出错误信息或生成带行号的输出时非常有用。
5.  **构建字典**：在需要将元素及其索引作为键值对存储在字典中时，`enumerate()` 可以非常方便地生成这些键值对。
6.  **数据分析和可视化**：在进行数据分析和可视化时，经常需要处理序列数据（如时间序列数据）。`enumerate()` 可以帮助你在处理这些数据时轻松地访问每个数据点的索引和值。
7.  **代码可读性**：使用 `enumerate()` 可以使代码更加清晰和易于理解。当你看到 `for index, value in enumerate(some_list):` 这样的代码时，你立即就能明白这段代码正在同时遍历索引和值。

### 二、基本语法

```python
number = ['北京','上海','广州','深圳','杭州','郑州']
#普通方式遍历列表中的元素，并输出索引
i = 0
for num in number:
    print('城市',num,'索引',i)
    i += 1
'''
城市 北京 索引 0
城市 上海 索引 1
城市 广州 索引 2
城市 深圳 索引 3
城市 杭州 索引 4
城市 郑州 索引 5
'''

#使用enumerate
for num,city in enumerate(number):
    print('城市', city, '索引', num)
'''
城市 北京 索引 0
城市 上海 索引 1
城市 广州 索引 2
城市 深圳 索引 3
城市 杭州 索引 4
城市 郑州 索引 5
'''
```

##### 参数

-   `iterable`：一个可迭代对象，如列表、元组、字符串等。
-   `start`（可选）：下标起始位置，默认为 0。

##### 返回值

`enumerate()` 返回一个枚举对象，该对象是一个迭代器，每次迭代返回一个包含索引和值的元组。

### 三、指定起始索引

你可以通过 `start` 参数来指定索引的起始值。

```python
for index, value in enumerate(['a', 'b', 'c'], start=1):  
    print(f"Index: {index}, Value: {value}")  
  
# 输出:  
# Index: 1, Value: a  
# Index: 2, Value: b  
# Index: 3, Value: c
```

### 四、与其他迭代器的结合使用

`enumerate()` 可以与任何可迭代对象结合使用，包括文件对象、生成器等。

```python
# 示例：枚举文件中的行  
with open('example.txt', 'r') as file:  
    for index, line in enumerate(file, start=1):  
        print(f"Line {index}: {line.strip()}")
```

### 五、列表推导式中使用

虽然 `enumerate()` 通常用于 for 循环中，但你也可以在列表推导式等表达式中使用它。

```python
# 示例：将枚举结果转换为列表  
lst = ['a', 'b', 'c']  
indexed_lst = [(index, value) for index, value in enumerate(lst, start=1)]  
print(indexed_lst)  
  
# 输出:  
# [(1, 'a'), (2, 'b'), (3, 'c')]
```

### 六、遍历嵌套列表

假设你有一个嵌套列表，即列表中包含其他列表，你希望遍历这个嵌套列表并同时获取外层列表的索引和内层列表的元素。


```python
nested_list = [['apple', 'banana'], ['cherry', 'date'], ['fig', 'grape']]  
for outer_index, inner_list in enumerate(nested_list):  
    for inner_index, fruit in enumerate(inner_list):  
        print(f"Outer Index: {outer_index}, Inner Index: {inner_index}, Fruit: {fruit}")  
  
# 输出:  
# Outer Index: 0, Inner Index: 0, Fruit: apple  
# Outer Index: 0, Inner Index: 1, Fruit: banana  
# Outer Index: 1, Inner Index: 0, Fruit: cherry  
# Outer Index: 1, Inner Index: 1, Fruit: date  
# Outer Index: 2, Inner Index: 0, Fruit: fig  
# Outer Index: 2, Inner Index: 1, Fruit: grape
```

在这个例子中，`enumerate()` 被用于外层循环以获取外层列表的索引，而内层循环则直接遍历内层列表的元素。虽然内层循环没有直接使用 `enumerate()` 来获取内层列表的索引（因为我们可以直接在循环中使用 `enumerate()` 的内部索引），但你可以看到 `enumerate()` 如何与嵌套结构一起工作。

### 七、根据索引筛选元素

假设你有一个列表，并希望根据元素的索引来筛选特定的元素。例如，你可能只想获取索引为偶数的元素。

```python
numbers = [1, 2, 3, 4, 5, 6]  
filtered_numbers = [number for index, number in enumerate(numbers) if index % 2 == 0]  
print(filtered_numbers)  
  
# 输出:  
# [2, 4, 6]
```

在这个例子中，我们使用列表推导式和 `enumerate()` 来创建一个新列表，该列表只包含原始列表中索引为偶数的元素。

### 八、遍历字典并同时获取键和值

虽然字典本身不是一个可迭代对象（你不能直接迭代字典来获取键和值），但你可以使用 `dict.items()` 方法将其转换为一个可迭代对象，该对象包含字典中的键值对。然后，你可以使用 `enumerate()` 来遍历这些键值对，但请注意，`enumerate()` 实际上会为你提供每个键值对的索引（即它们在迭代中的位置），而不是字典中的键。然而，在这个场景中，我们可能更关心键和值本身，而不是它们的索引。不过，为了展示 `enumerate()` 的用法，我们可以这样做：

```python
my_dict = {'a': 1, 'b': 2, 'c': 3}  
for index, (key, value) in enumerate(my_dict.items()):  
    print(f"Index: {index}, Key: {key}, Value: {value}")  
  
# 输出:  
# Index: 0, Key: a, Value: 1  
# Index: 1, Key: b, Value: 2  
# Index: 2, Key: c, Value: 3
```

### 九、结合 lambda 表达式和 enumerate()

虽然 `enumerate()` 通常不与 `lambda` 表达式直接结合使用（因为 `lambda` 主要用于创建匿名函数，而 `enumerate()` 是一个内置函数），但你可以在涉及 `enumerate()` 的更复杂的表达式中使用 `lambda`。例如，你可能想使用 `filter()` 函数和 `lambda` 表达式来根据索引和值筛选元素，尽管在这种情况下，直接使用列表推导式可能更为直观和简单。不过，为了说明目的，这里有一个使用 `filter()` 和 `lambda` 的示例（尽管它并不直接结合 `enumerate()`，但展示了类似的思想）：


```python
numbers = [1, 2, 3, 4, 5, 6]  
# 注意：这个例子并不直接使用 enumerate()，但它展示了如何在类似上下文中使用 lambda  
filtered_numbers = list(filter(lambda x: x[1] % 2 ==
```
