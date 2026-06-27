---
title: python学习笔记16-zip()函数
description: ""
slug: 2021-06-26-python学习笔记16-zip()函数
date: 2021-06-26
image: python.webp
categories:
  - 编程语言
tags:
  - python
  - 函数
  - zip
---
Python中的`zip`函数是一个非常实用的内置函数，它主要用于将多个可迭代对象（如列表、元组等）中对应位置的元素打包成一个个元组，然后返回由这些元组组成的zip对象（在Python 3中，zip对象是一个迭代器）。如果各个迭代器的元素个数不一致，则返回列表长度与最短的对象相同，利用`*`操作符，可以将zip对象解压为列表。

#### 基本用法

在这个例子中，`zip(list1, list2)`会生成一个zip对象，该对象包含了三个元组，每个元组都是`list1`和`list2`中对应位置的元素组成的。

```python
list1 = [1, 2, 3]  
list2 = ['a', 'b', 'c']  
  
# 使用zip函数  
zipped = zip(list1, list2)  
  
# 转换为列表查看结果  
print(list(zipped))  
# 输出: [(1, 'a'), (2, 'b'), (3, 'c')]
```

#### 与多个可迭代对象一起使用

`zip`函数也可以与多于两个的可迭代对象一起使用：

```python
list1 = [1, 2, 3]  
list2 = ['a', 'b', 'c']  
list3 = [True, False, True]  
  
# 使用zip函数  
zipped = zip(list1, list2, list3)  
  
# 转换为列表查看结果  
print(list(zipped))  
# 输出: [(1, 'a', True), (2, 'b', False), (3, 'c', True)]
```

#### 解压zip对象

`zip`对象可以通过`*`操作符在函数调用中解压，或者通过`list()`, `tuple()`, `dict()`等函数转换为列表、元组或字典（转换为字典时需要额外的处理，因为`zip`对象直接转换会得到元组的列表或元组，而不是字典）。

```python
# 解压zip对象到多个变量  
a, b = zip(*[(1, 'a'), (2, 'b')])  
print(list(a), list(b))  
# 输出: [1, 2] ['a', 'b']  
  
# 转换为字典（需要额外的处理，因为zip直接转换得到的是元组的列表）  
keys = ['name', 'age']  
values = ['Alice', 30]  
dict_from_zip = dict(zip(keys, values))  
print(dict_from_zip)  
# 输出: {'name': 'Alice', 'age': 30}
```

#### 遍历ZIP对象

```python
num1 = ['张三','李四','王五','赵六']
num2 = [19,31,23,32]
for name,age in zip(num1,num2):
    print('姓名',name,' 年龄',age)
    '''
    姓名 张三  年龄 19
    姓名 李四  年龄 31
    姓名 王五  年龄 23
    姓名 赵六  年龄 32
    '''
```

两个可迭代对象长度不一致

可以看到两个数组的元素个数不一致，返回的zip对象长度将与最短的可迭代对象相同。

```python
num1 = ['张三','李四','王五','赵六']
num2 = [19,31,23,32,22]
for name,age in zip(num1,num2):
    print('姓名',name,' 年龄',age)
    '''
    姓名 张三  年龄 19
    姓名 李四  年龄 31
    姓名 王五  年龄 23
    姓名 赵六  年龄 32
    '''
```

#### 注意事项

-   `zip`函数返回的是一个迭代器，这意味着你只能遍历它一次。如果你需要再次遍历，你需要将结果转换为列表或其他可迭代对象。
-   当`zip`函数中的可迭代对象长度不一致时，返回的zip对象长度将与最短的可迭代对象相同。
-   在Python 2中，`zip`函数直接返回列表；在Python 3中，它返回的是一个zip对象，需要转换为列表或其他类型才能查看其内容。

