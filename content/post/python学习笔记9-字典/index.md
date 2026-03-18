---
title: python学习笔记9-字典
description: ""
slug: 2021-05-24-python学习笔记9-字典
date: 2021-05-24
image: python.webp
categories:
  - 编程语言
tags:
  - python
  - 字典
  - dict
---
### 字典

> Python中的字典（Dictionary）是一种非常强大且灵活的数据结构，用于存储键值对（key-value pairs）。字典是可变的，并且可以包含任意类型的对象作为键或值。在字典中，每个键都是唯一的，并且每个键都映射到一个值。 和列表的区别

-   列表 是 有序 的对象集合
-   字典 是 无序 的对象集合

|    |                               |                                                                          |
|-----|-------------------------------|--------------------------------------------------------------------------|
| 新增 |        dictkey = value        |                  通过键来新增或修改键值对。如果键不存在，则新增；如果键已存在，则修改其值。                   |
|   |         update(other)         |  使用另一个字典（或可迭代键值对对象）中的键值对来更新当前字典。如果other中的键在字典中已存在，则其值将被覆盖；如果不存在，则将被添加。   |
| 修改 |      dictkey = new_value      |                       修改字典中指定键的值。如果键不存在，则相当于新增键值对。                       |
|   | setdefault(key, default=None) |       如果字典中不存在指定的键，则添加键并将值设为默认值（默认为None）。如果键已存在，则返回该键对应的值，但不会修改它。        |
| 删除 |          del dictkey          |                     删除字典中指定键的键值对。如果键不存在，将引发KeyError。                     |
|   |    pop(key, default=None)     | 移除字典中指定键的键值对，并返回该键对应的值。如果键不存在且未指定默认值，将引发KeyError。如果指定了默认值，则在键不存在时返回该默认值。 |
|   |           popitem()           |         移除并返回字典中的最后一对键值对（Python 3.7+ 中按插入顺序）。如果字典为空，则引发KeyError。         |
|   |            clear()            |                           移除字典中的所有键值对，使其变为空字典。                           |

### 字典的基本特点

1.  无序性：字典中的项（键值对）是无序的。你不能通过索引来访问字典中的元素，因为字典不保证元素的顺序。
2.  键的唯一性：字典中的每个键都必须是唯一的，但值则不必。
3.  可变性：字典是可变的，你可以添加、删除或修改字典中的项。
4.  动态性：字典的大小是可变的，你可以根据需要添加或删除键值对。

### 语法

> 字典用 {} 定义 字典使用 键值对 存储数据， 键值对之间使用 , 分隔 键 key 是索引 值 value 是数据 键 和 值 之间使用: 分隔 键必须是唯一的 值 可以取任何数据类型，但 键 只能使用 字符串、数字或 元组

```python
# 创建一个空字典
my_dict = {}
# 使用dict创建一个空字典
my_dict2 = dict()

# 创建一个包含键值对的字典
my_dict = {'name': 'John', 'age': 30, 'city': 'New York'}

# 创建一个包含键值对的字典
my_dict = dict(name='John',age=30, city='New York')
```

#### 注意事项

1.  键的唯一性： 字典中的每个键都必须是唯一的。如果尝试添加或修改一个已经存在的键，其对应的值将被新值替换。
2.  键的不可变性： 字典的键必须是不可变的（immutable）。这意味着你可以使用数字、字符串或元组作为键，但不能使用列表或其他可变类型作为键。
3.  值的可变性： 与键不同，字典的值可以是任何类型，包括可变类型（如列表、字典等）。
4.  动态大小： 字典的大小是动态的，可以根据需要添加或删除键值对。
5.  键的排序： 在Python 3.7及以后的版本中，字典会保持插入顺序（插入顺序即迭代顺序）。然而，这并不意味着字典是排序的或你可以依赖其顺序进行排序操作。如果你需要排序的键值对，应该使用 sorted() 函数对 items() 的结果进行排序。
6.  性能： 字典的查找、插入和删除操作的平均时间复杂度为O(1)，这使得字典成为处理大量键值对数据时的理想选择。
7.  避免使用 dict 作为变量名： dict 是Python中用于创建字典的内置类型。因此，应避免使用 dict 作为变量名，以避免覆盖或混淆内置类型。
8.  空字典： 空字典不包含任何键值对，可以通过 {} 或 dict() 创建。
9.  更新字典： 你可以使用 update() 方法来更新字典，该方法可以接受另一个字典或包含(key, value)对的可迭代对象，并将其内容合并到当前字典中。
10.  字典推导式： 字典推导式提供了一种简洁的方式来创建或更新字典。它们类似于列表推导式，但生成的是字典而不是列表。

### 访问字典中的值

> Python字典的取值方式非常直接和灵活，主要涉及到通过键（key）来访问对应的值（value）。

#### 直接通过键访问

这是最常见的取值方式，直接使用方括号 \[\] 和键名来访问对应的值。

```python
# 创建一个字典  
my_dict = {'name': 'John', 'age': 30, 'city': 'New York'}  
  
# 通过键名访问值  
name = my_dict['name']  # 直接通过键 'name' 访问值  
print(name)  # 输出: John  
  
# 尝试访问不存在的键会抛出 KeyError  
# try:  
#     print(my_dict['country'])  # 这将引发 KeyError  
# except KeyError as e:  
#     print(f"KeyError: {e}")  
  
# 为了避免 KeyError，可以使用 get() 方法  
country = my_dict.get('country', 'Unknown')  # 如果键不存在，则返回 'Unknown'  
print(country)  # 输出: Unknown
```

#### 使用 get() 方法

get() 方法提供了一种更安全的访问字典项的方式，如果指定的键不存在，则返回一个默认值（默认为 None），而不是抛出 KeyError。

```python
# 使用 get() 方法访问存在的键  
name_with_get = my_dict.get('name', 'Default Name')  # 第二个参数是默认值，这里不会被使用  
print(name_with_get)  # 输出: John  
  
# 使用 get() 方法访问不存在的键，并返回默认值  
default_city = my_dict.get('country', 'Unknown City')  
print(default_city)  # 输出: Unknown City
```

### 字典的判断

#### 判断键（Key）是否存在

```python
my_dict = {'a': 1, 'b': 2, 'c': 3}  
# 判断键 'a' 是否存在  
if 'a' in my_dict:  
    print("键 'a' 存在")  # 预期输出: 键 'a' 存在  
else:  
    print("键 'a' 不存在")  
# 判断键 'd' 是否存在  
if 'd' in my_dict:  
    print("键 'd' 存在")  
else:  
    print("键 'd' 不存在")  # 预期输出: 键 'd' 不存在
```

#### 判断值（Value）是否存在

由于直接判断值是否存在比较复杂（因为可能有多个键对应相同的值），我们通常需要遍历字典或使用其他方法。

```python
# 定义一个字典，包含键值对：'a' 对应 1, 'b' 对应 2, 'c' 对应 3, 'd' 对应 1  
my_dict = {'a': 1, 'b': 2, 'c': 3, 'd': 1}  
  
# 初始化一个标志变量，用于标记值 1 是否存在，初始值为 False  
value_exists = False  
  
# 遍历字典中的所有值  
for value in my_dict.values():  
    # 如果当前遍历到的值等于 1  
    if value == 1:  
        # 将标志变量设置为 True，表示值 1 存在  
        value_exists = True  
        # 找到值 1 后，跳出循环，因为不需要继续检查其他值了  
        break  
  
# 根据标志变量的值，打印相应的信息  
if value_exists:  
    # 如果 value_exists 为 True，则打印 "值 1 存在"  
    print("值 1 存在")  # 预期输出: 值 1 存在  
else:  
    # 如果 value_exists 为 False，则打印 "值 1 不存在"（但在这个例子中，由于字典中确实包含值 1，所以这行代码不会被执行）  
    print("值 1 不存在")  
  
# 另一种判断值 1 是否存在于字典中的方法是使用 any() 函数和生成器表达式  
# any() 函数会检查给定的迭代器是否至少有一个元素为 True  
# 生成器表达式 (value == 1 for value in my_dict.values()) 会遍历字典中的所有值，并生成一个布尔值的迭代器  
# 如果迭代器中至少有一个 True（即至少有一个值等于 1），any() 函数就会返回 True  
if any(value == 1 for value in my_dict.values()):  
    # 如果 any() 函数返回 True，则打印 "值 1 存在"  
    print("值 1 存在")  # 预期输出: 值 1 存在  
else:  
    # 如果 any() 函数返回 False（即字典中没有值等于 1），则打印 "值 1 不存在"（但在这个例子中，这行代码不会被执行）  
    print("值 1 不存在")
```

首先通过传统的遍历方法来检查字典中是否存在值 1，并设置了一个标志变量 value\_exists 来记录检查结果。然后，我们使用了 any() 函数和生成器表达式来实现相同的检查，但这种方法更加简洁和高效，因为它在找到第一个匹配项时就会停止遍历。在两种情况下，由于字典 my\_dict 中确实包含值 1，所以都会打印出 “值 1 存在”。

#### 判断键值对（Key-Value Pair）是否存在

```python
# 定义一个字典，包含三个键值对：'a' 对应 1, 'b' 对应 2, 'c' 对应 3  
my_dict = {'a': 1, 'b': 2, 'c': 3}  
  
# 判断键值对 ('a', 1) 是否存在于字典中  
# 这通过检查元组 ('a', 1) 是否作为元素存在于 my_dict.items() 返回的迭代器中来实现  
# my_dict.items() 返回一个包含字典中所有(键, 值)对的视图对象  
if ('a', 1) in my_dict.items():  
    # 如果 ('a', 1) 在 my_dict.items() 中找到，则打印 "键值对 ('a', 1) 存在"  
    print("键值对 ('a', 1) 存在")  # 预期输出: 键值对 ('a', 1) 存在  
else:  
    # 如果 ('a', 1) 不在 my_dict.items() 中，则打印 "键值对 ('a', 1) 不存在"  
    # 但由于 ('a', 1) 确实在字典中，所以这行代码不会被执行  
    print("键值对 ('a', 1) 不存在")  
  
# 判断键值对 ('d', 4) 是否存在于字典中  
# 使用与上面相同的方法，但这次检查的是 ('d', 4)  
if ('d', 4) in my_dict.items():  
    # 如果 ('d', 4) 在 my_dict.items() 中找到，则打印 "键值对 ('d', 4) 存在"  
    # 但由于 ('d', 4) 不在字典中，所以这行代码不会被执行  
    print("键值对 ('d', 4) 存在")  
else:  
    # 如果 ('d', 4) 不在 my_dict.items() 中，则打印 "键值对 ('d', 4) 不存在"  
    # 这是预期的输出，因为 ('d', 4) 确实不在字典中  
    print("键值对 ('d', 4) 不存在")  # 预期输出: 键值对 ('d', 4) 不存在
```

首先定义了一个包含三个键值对的字典 my\_dict。然后，我们使用 in 关键字和 my\_dict.items() 方法来判断特定的键值对是否存在于字典中。my\_dict.items() 返回一个视图对象，该对象包含了字典中所有的(键, 值)对。我们通过检查元组（如 (‘a’, 1) 或 (‘d’, 4)）是否作为元素存在于这个视图中来做出判断。根据这些键值对是否实际存在于字典中，相应的打印语句会被执行。

#### 字典是否为空

```python
# 定义一个空字典  
empty_dict = {}  
  
# 定义一个非空字典，包含一个键值对：'a' 对应 1  
non_empty_dict = {'a': 1}  
  
# 使用bool()函数判断空字典是否为空  
# bool()函数会将空字典视为False，因为空字典被认为是逻辑上的假  
if not bool(empty_dict):  
    # 如果bool(empty_dict)的结果是False（即empty_dict为空），则打印"empty_dict 为空"  
    print("empty_dict 为空")  # 预期输出: empty_dict 为空  
  
# 使用bool()函数判断非空字典是否不为空  
# bool()函数会将非空字典视为True，因为非空字典被认为是逻辑上的真  
if bool(non_empty_dict):  
    # 如果bool(non_empty_dict)的结果是True（即non_empty_dict不为空），则打印"non_empty_dict 不为空"  
    print("non_empty_dict 不为空")  # 预期输出: non_empty_dict 不为空  
  
# 或者直接通过检查字典的长度来判断其是否为空  
# 字典的len()函数会返回字典中键值对的数量  
if len(empty_dict) == 0:  
    # 如果empty_dict的长度为0，即它没有包含任何键值对，则打印"empty_dict 为空"  
    print("empty_dict 为空")  # 预期输出: empty_dict 为空  
  
# 同样地，通过检查长度来判断非空字典是否不为空  
if len(non_empty_dict) > 0:  
    # 如果non_empty_dict的长度大于0，即它包含至少一个键值对，则打印"non_empty_dict 不为空"  
    print("non_empty_dict 不为空")  # 预期输出: non_empty_dict 不为空
```

两种检查字典是否为空的方法：一种是通过bool()函数，另一种是通过直接检查字典的长度（使用len()函数）。对于空字典，这两种方法都会返回True（对于检查为空的情况）或0（对于通过长度检查的情况），从而允许我们通过条件语句来判断并打印出相应的信息。

### 字典的新增

#### 直接赋值

直接通过指定键来赋值，如果键不存在，则新增该键值对；如果键已存在，则更新其对应的值。

```python
# 初始字典  
my_dict = {'name': 'John', 'age': 30}  
  
# 新增键值对  
my_dict['city'] = 'New York'  # 添加城市信息  
  
print(my_dict)  # {'name': 'John', 'age': 30, 'city': 'New York'}
```

#### 使用dict.update()方法

update() 方法可以使用另一个字典对象来更新当前字典。如果两个字典有相同的键，则当前字典中的值会被更新为另一个字典中相应的值；如果另一个字典中有当前字典中没有的键，则新增该键值对。

```python
# 初始字典  
my_dict = {'name': 'John', 'age': 30}  
  
# 另一个字典  
new_items = {'city': 'New York', 'job': 'Engineer'}  
  
# 更新字典  
my_dict.update(new_items)  
  
print(my_dict)  # {'name': 'John', 'age': 30, 'city': 'New York', 'job': 'Engineer'}
```

#### 使用字典推导式（虽然主要用于修改或创建新字典，但也可实现新增效果）

虽然字典推导式主要用于基于现有字典创建新字典或修改字典，但在某些情况下，通过合并字典或条件判断也可以实现新增效果。

```python
# 初始字典  
my_dict = {'name': 'John', 'age': 30}  
  
# 使用字典推导式（实际这里更多是修改或合并的示例）  
my_dict = {**my_dict, 'city': 'New York'}  # 使用Python 3.5+的解包特性合并字典  
  
print(my_dict)  # {'name': 'John', 'age': 30, 'city': 'New York'}
```

#### 使用collections.defaultdict

虽然这不是直接新增键值对到普通字典的方法，但defaultdict提供了一种方便的方式来自动处理不存在的键，这在某些场景下可以间接实现新增键值对的效果，特别是当你需要为不存在的键设置默认值时。

```python
from collections import defaultdict  
  
# 使用int作为默认值，创建defaultdict  
my_dict = defaultdict(int)  
  
# “新增”键值对，如果键不存在，会自动创建并赋默认值  
my_dict['count'] += 1  # 等同于 if 'count' not in my_dict: my_dict['count'] = 0; my_dict['count'] += 1  
  
print(my_dict)  # defaultdict(<class 'int'>, {'count': 1})  
  
# 转换为普通字典  
my_dict = dict(my_dict)  
print(my_dict)  # {'count': 1}
```

#### 注意点

-   直接赋值是最简单直接的方法，但在处理大量数据或频繁更新时，考虑性能问题。
-   update() 方法适用于批量更新字典，可以处理多个键值对的添加或更新。
-   字典推导式提供了强大的灵活性，但在仅新增键值对的场景下可能不是最直接的方法。
-   defaultdict 提供了对不存在的键的自动处理，这在处理计数、分组等任务时非常有用。

### 字典的修改

> 在Python中，字典（Dictionary）的修改是一个常见的操作，它涉及到改变已存在的键值对或添加新的键值对（如果键不存在的话）。

#### 直接修改

最直接的方式是通过指定键来直接修改其对应的值。

```python
# 初始字典  
my_dict = {'name': 'John', 'age': 30}  
  
# 修改age的值  
my_dict['age'] = 31  # 直接修改  
  
# 添加新的键值对  
my_dict['city'] = 'New York'  # 如果键不存在，则新增  
  
print(my_dict)  # 输出: {'name': 'John', 'age': 31, 'city': 'New York'}
```

#### 使用update()方法

update()方法用于更新字典中的键值对，如果键已存在，则覆盖其值；如果键不存在，则新增键值对。

```python
# 初始字典  
my_dict = {'name': 'John', 'age': 30}  
  
# 另一个字典，用于更新  
new_items = {'age': 31, 'city': 'New York'}  
  
# 使用update()方法更新字典  
my_dict.update(new_items)  
  
print(my_dict)  # 输出: {'name': 'John', 'age': 31, 'city': 'New York'}
```

#### 字典推导式（间接修改）

虽然字典推导式主要用于基于现有字典创建新字典，但你可以通过条件逻辑在推导过程中修改或选择性地包含键值对。

```python
# 初始字典  
my_dict = {'name': 'John', 'age': 30, 'job': 'Developer'}  
  
# 假设我们只想保留name和age，并修改age的值  
new_dict = {key: value if key != 'age' else value + 1 for key, value in my_dict.items()}  
  
# 注意：这里实际上是创建了一个新字典，而不是修改原字典  
print(new_dict)  # 输出: {'name': 'John', 'age': 31, 'job': 'Developer'}  # 注意：'job'键被意外保留了，因为条件只检查了'age'  
  
# 正确的做法是使用条件来过滤键  
filtered_dict = {key: value if key != 'job' else value if key == 'age' else value + 1 for key, value in my_dict.items()}  
# 但上面的逻辑依然复杂且不直观，通常我们会直接修改或使用update()  
  
# 更简洁的修改age的方式  
my_dict['age'] = my_dict['age'] + 1  
  
print(my_dict)  # 输出: {'name': 'John', 'age': 31, 'job': 'Developer'}
```

注意：字典推导式通常不用于直接修改原字典，而是用于生成新字典。

### 字典的删除

> 在Python中，字典（Dictionary）的删除操作涉及移除字典中的键值对或清空整个字典。

#### 使用del语句删除指定键

del语句可以直接删除字典中的指定键及其对应的值。如果键不存在，将抛出KeyError。

```python
# 初始字典  
my_dict = {'name': 'John', 'age': 30, 'city': 'New York'}  
  
# 使用del语句删除'city'键  
del my_dict['city']  
  
print(my_dict)  # 输出: {'name': 'John', 'age': 30}  
  
# 尝试删除不存在的键（会抛出KeyError）  
# del my_dict['country']  # Uncomment to see KeyError
```

#### 使用pop()方法删除指定键并返回其值

pop(key, default)方法移除字典中指定的键并返回其值。如果键不存在且提供了默认值，则返回默认值；否则，抛出KeyError。

```python
# 初始字典  
my_dict = {'name': 'John', 'age': 30, 'city': 'New York'}  
  
# 使用pop()方法删除'age'键并获取其值  
age = my_dict.pop('age')  
  
print(f"Removed age: {age}, New dict: {my_dict}")  # 输出: Removed age: 30, New dict: {'name': 'John', 'city': 'New York'}  
  
# 尝试删除不存在的键并指定默认值  
country = my_dict.pop('country', 'Unknown')  
print(f"Country (if exists or default): {country}")  # 输出: Country (if exists or default): Unknown
```

#### 使用popitem()方法移除并返回字典中的最后一个键值对（Python 3.7+ 保证按插入顺序）

popitem()方法移除并返回字典中的一个键值对（项）。返回的键值对是一个在字典中任意位置（但在Python 3.7+中，是按照插入顺序的最后一个）的(key, value)元组。如果字典为空，则抛出KeyError。

```python
# 初始字典  
my_dict = {'name': 'John', 'age': 30, 'city': 'New York'}  
  
# 使用popitem()移除最后一个键值对  
last_item = my_dict.popitem()  
  
print(f"Removed item: {last_item}, New dict: {my_dict}")  # 输出可能因Python版本和插入顺序而异，例如: Removed item: ('city', 'New York'), New dict: {'name': 'John', 'age': 30}
```

#### 使用clear()方法清空字典

clear()方法移除字典中的所有项，将其长度变为0。

```python
# 初始字典  
my_dict = {'name': 'John', 'age': 30, 'city': 'New York'}  
  
# 使用clear()清空字典  
my_dict.clear()  
  
print(my_dict)  # 输出: {}
```

#### 注意点

-   使用del语句和pop()方法删除不存在的键时，del会抛出KeyError，而pop()可以指定一个默认值来避免这个错误。
-   popitem()方法的行为在Python 3.6及以前版本中可能不是按插入顺序的，但从Python 3.7开始，字典是按照插入顺序来维护键值对的。
-   clear()方法会清空整个字典，所以在使用之前要确保这是你想要的操作。

### 遍历字典

> 在Python中，字典（Dictionary）的遍历是处理字典数据时非常常见的操作。字典是一种可变容器模型，且可存储任意类型对象，如字符串、数字、元组等其他容器模型。字典的每个元素都是一个键值对（key-value pair），键和值通过冒号（:）分隔，元素之间用逗号（,）分隔，整个字典包括在花括号（{}）中。 遍历字典主要涉及到遍历其键（keys）、值（values）或键值对（items）。

#### 遍历字典的键（Keys）

使用.keys()方法可以获得字典中所有的键，并将其作为一个视图对象返回。然后，你可以遍历这个视图对象来获取所有的键。

```python
# 定义字典  
my_dict = {'name': 'John', 'age': 30, 'city': 'New York'}  
  
# 遍历字典的键  
for key in my_dict.keys():  
    print(key)  # 打印每个键  
  
# 运行结果：  
# name  
# age  
# city
```

#### 遍历字典的值（Values）

类似地，.values()方法返回字典中所有的值，作为一个视图对象。你可以遍历这个对象来获取所有的值。

```python
# 遍历字典的值  
for value in my_dict.values():  
    print(value)  # 打印每个值  
  
# 运行结果：  
# John  
# 30  
# New York
```

#### 遍历字典的键值对（Items）

.items()方法返回字典中所有的键值对，以(key, value)元组的形式。遍历这个对象可以同时获取键和值。

```python
# 遍历字典的键值对  
for key, value in my_dict.items():  
    print(f"Key: {key}, Value: {value}")  # 打印每个键值对  
  
# 运行结果：  
# Key: name, Value: John  
# Key: age, Value: 30  
# Key: city, Value: New York
```

#### 遍历字典时保持顺序（Python 3.7+）

从Python 3.7开始，字典按照插入顺序进行遍历。这意味着.keys(), .values(), 和 .items() 方法返回的元素顺序与键被添加到字典中的顺序一致。

```python
# 在Python 3.7+中，字典保留了插入顺序  
for key, value in my_dict.items():  
    print(f"Key: {key}, Value: {value}")  # 按键的插入顺序打印  
  
# 运行结果（顺序可能与上面的例子相同，但保证与插入顺序一致）：  
# Key: name, Value: John  
# Key: age, Value: 30  
# Key: city, Value: New York
```