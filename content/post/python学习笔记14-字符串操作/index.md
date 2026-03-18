---
title: python学习笔记14-字符串操作
description: ""
slug: 2021-06-18-python学习笔记14-字符串操作
date: 2021-06-18
image: python.webp
categories:
  - 编程语言
tags:
  - python
  - 字符串
---
### 字符串的查询方法

|                         |          |                                                                                                        |
|-------------------------|----------|--------------------------------------------------------------------------------------------------------|
|     查找子字符串第一次出现的位置      | index()  |           返回指定子字符串在字符串中第一次出现的索引位置，如果未找到子字符串，则抛出ValueError异常。可以通过可选参数start和end来指定搜索的起始和结束位置。            |
| 查找子字符串最后一次出现的位置（从右向左搜索） | rindex() |   与index()类似，但搜索方向是从字符串的末尾开始，返回指定子字符串最后一次出现的索引位置。如果未找到子字符串，则抛出ValueError异常。同样支持可选参数start和end来指定搜索范围。   |
|     查找子字符串第一次出现的位置      |  find()  | 返回指定子字符串在字符串中第一次出现的索引位置，如果未找到子字符串，则返回-1。支持可选参数start和end来指定搜索的起始和结束位置。与index()不同，find()在找不到子字符串时不会抛出异常。 |
| 查找子字符串最后一次出现的位置（从右向左搜索） | rfind()  |         与find()类似，但搜索方向是从字符串的末尾开始，返回指定子字符串最后一次出现的索引位置。如果未找到子字符串，则返回-1。支持可选参数start和end来指定搜索范围。          |


```python
'''
字符串的查询操作

index()方法，查找子串substr第一次出现的位置(下标)，如果查找的子串不存在时，则抛出ValueError
rindex()方法，查找子串substr第一次出现的位置(下标)，如果查找的子串不存在时，则抛出ValueError
find()方法，查找子串substr第一次出现的位置(下标)，如果查找的子串不存在时，返回-1
rfind()方法，查找子串substr第一次出现的位置(下标)，如果查找的子串不存在时，返回-1
'''
s = "hello,hello"
print(s.find("lo")) # 3
print(s.index("lo")) # 3
print(s.rfind("lo")) # 9
print(s.rindex("lo")) # 9

#使用index或者rindex，如果查找的内容不存在则报错
#print(s.index("u")) # ValueError: substring not found
#print(s.rindex("u")) # ValueError: substring not found
```

### 字符串转换大小写方法

|                                   |              |                                                                                   |
|-----------------------------------|--------------|-----------------------------------------------------------------------------------|
|        将字符串中的所有小写字母转换为大写字母        |   upper()    |                        返回字符串的一个新版本，其中所有的小写字母都被转换成了对应的大写字母。                        |
|        将字符串中的所有大写字母转换为小写字母        |   lower()    |                        返回字符串的一个新版本，其中所有的大写字母都被转换成了对应的小写字母。                        |
|           翻转字符串中的大小写字母            |  swapcase()  |                返回字符串的一个新版本，其中所有的小写字母都被转换成了大写字母，所有的大写字母都被转换成了小写字母。                 |
|   将字符串的第一个字符转换为大写字母，其余字符转换为小写字母   | capitalize() |         返回字符串的一个新版本，其中字符串的第一个字符被转换成了大写字母（如果它是小写字母的话），而字符串的其余部分被转换成了小写字母。          |
| 将字符串中的每个单词的首字母转换为大写字母，其余字符转换为小写字母 |   title()    | 返回字符串的一个新版本，其中每个单词的首字母都被转换成了大写字母，而单词的其余部分被转换成了小写字母。注意，这里的“单词”是根据空格、标点符号等分隔的字符串部分。 |


```python
# 原始字符串  
original_str = "Hello, World! This is a Test String."  
  
# 使用upper()方法  
upper_str = original_str.upper()  # 将字符串中的所有小写字母转换为大写字母  
print("使用upper():", upper_str)  # 运行结果: 使用upper(): HELLO, WORLD! THIS IS A TEST STRING.  
  
# 使用lower()方法  
lower_str = original_str.lower()  # 将字符串中的所有大写字母转换为小写字母  
print("使用lower():", lower_str)  # 运行结果: 使用lower(): hello, world! this is a test string.  
  
# 使用swapcase()方法  
swapcase_str = original_str.swapcase()  # 翻转字符串中的大小写字母  
print("使用swapcase():", swapcase_str)  # 运行结果: 使用swapcase(): hELLO, wORLD! tHIS IS a tEST sTRING.  
  
# 使用capitalize()方法  
capitalize_str = original_str.capitalize()  # 将字符串的第一个字符转换为大写字母，其余字符转换为小写字母  
print("使用capitalize():", capitalize_str)  # 运行结果: 使用capitalize(): Hello, world! this is a test string.  
  
# 使用title()方法  
title_str = original_str.title()  # 将字符串中的每个单词的首字母转换为大写字母，其余字符转换为小写字母  
print("使用title():", title_str)  # 运行结果: 使用title(): Hello, World! This Is A Test String.  
  
# 注意：title()方法会根据空格和标点符号来识别单词边界  
# 例如，标点符号后的字母也会被转换为大写
```

请注意，title()方法在处理包含标点符号的字符串时，会将标点符号后面的第一个字母也转换为大写，这可能与某些预期不同。比如，在英文中，标点符号（如逗号、句号）后面通常跟随小写字母开始的单词，但title()方法会将这些字母也转换为大写。如果你需要更精细地控制大小写转换，可能需要根据具体情况编写自定义的函数来处理字符串。

### 字符串内容对齐方法

|                               |                           |                                                                              |
|-------------------------------|---------------------------|------------------------------------------------------------------------------|
| 返回一个原字符串居中,并使用空格填充至指定宽度的新字符串  | center(width[, fillchar]) | width参数指定了新字符串的总宽度。fillchar参数是可选的，用于指定填充字符，默认为空格。如果原字符串长度大于或等于width，则返回原字符串。 |
| 返回一个原字符串左对齐,并使用空格填充至指定宽度的新字符串 | ljust(width[, fillchar])  |                     与center()方法类似，但填充发生在字符串的右侧，使得字符串左对齐。                     |
| 返回一个原字符串右对齐,并使用空格填充至指定宽度的新字符串 | rjust(width[, fillchar])  |                     与center()方法类似，但填充发生在字符串的左侧，使得字符串右对齐。                     |
|   返回一个原字符串左端用零填充至指定宽度的新字符串    |       zfill(width)        |   width参数指定了新字符串的总宽度。该方法在字符串的左侧填充零，直到达到指定的宽度。如果原字符串的长度大于或等于width，则返回原字符串。    |

```python
# 原始字符串  
original_str = "Python"  
  
# 使用center()方法  
centered_str = original_str.center(20, '*')  # 将字符串居中，总宽度为20，用'*'填充  
print("使用center():", centered_str)  # 运行结果: 使用center(): ****Python******  
  
# 使用ljust()方法  
ljust_str = original_str.ljust(20, '-')  # 将字符串左对齐，总宽度为20，用'-'填充  
print("使用ljust():", ljust_str)  # 运行结果: 使用ljust(): Python------------  
  
# 使用rjust()方法  
rjust_str = original_str.rjust(20, '_')  # 将字符串右对齐，总宽度为20，用'_'填充  
print("使用rjust():", rjust_str)  # 运行结果: 使用rjust(): ____________Python  
  
# 使用zfill()方法  
zfill_str = original_str.zfill(20)  # 将字符串左端用零填充至总宽度为20  
print("使用zfill():", zfill_str)  # 运行结果: 使用zfill(): 00000000000000Python  
  
# 注意：如果指定的宽度小于或等于原字符串的长度，center(), ljust(), rjust()将返回原字符串的副本  
# 而zfill()则会根据指定的宽度在左侧填充零，无论原字符串的长度如何  
  
# 示例：当指定宽度小于原字符串长度时  
small_width = 4  
small_center = original_str.center(small_width)  
small_ljust = original_str.ljust(small_width)  
small_rjust = original_str.rjust(small_width)  
small_zfill = original_str.zfill(small_width)  # 但zfill的行为与其他方法不同，它会尝试填充，但结果可能不符合预期  
  
print("小宽度center():", small_center)  # 运行结果: 小宽度center(): Python  
print("小宽度ljust():", small_ljust)    # 运行结果: 小宽度ljust(): Python  
print("小宽度rjust():", small_rjust)    # 运行结果: 小宽度rjust(): Python  
print("小宽度zfill() 尝试但不符合预期:", small_zfill)  # 尝试但通常不会按预期工作，因为zfill总是在左侧填充  
# 注意：对于small_zfill，实际上Python不会截断原字符串来适应小宽度，而是直接返回原字符串，因为无法在原字符串左侧填充足够的零来减少其长度。  
# 但这里我保留了“尝试但不符合预期”的注释，以强调zfill的行为与其他方法在处理小宽度时的不同。
```

请注意，对于zfill()方法的注释中关于小宽度的部分，实际上Python的zfill()方法并不会尝试截断原字符串以适应小于原字符串长度的宽度。如果指定的宽度小于原字符串的长度，zfill()将简单地返回原字符串。我在注释中提到的“尝试但不符合预期”是为了强调这一点，并避免引起混淆。然而，在实际代码中，这个特定的示例可能不会按字面意思“尝试”去做什么，因为zfill()的行为是明确定义的。

### 字符串的拆分方法

|                                                |                               |                                                                                                                             |
|------------------------------------------------|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
|      通过指定分隔符对字符串进行拆分，并返回一个包含所有拆分后的子字符串的列表      | split(sep=None, maxsplit=-1)  | sep是分隔符，默认为空格（包括\n, \r, \t, ' '等）。如果sep为None或未指定，则任何空白字符（空格、换行\n、制表符\t等）都将被作为分隔符。maxsplit是可选参数，用于指定拆分的最大次数，默认为-1，表示不限制拆分次数。 |
| 从字符串的末尾开始，通过指定分隔符对字符串进行拆分，并返回一个包含所有拆分后的子字符串的列表 | rsplit(sep=None, maxsplit=-1) |                                   与split()类似，但拆分方向是从字符串的末尾开始。sep和maxsplit参数的含义与split()相同。                                   |

#### split()

```python
# 原始字符串，包含空格作为分隔符  
original_str = "Python is awesome"  
  
# 使用 split() 方法拆分字符串，默认以空格为分隔符  
split_result = original_str.split()  
print("使用 split() 拆分（默认空格分隔）:", split_result)  
# 运行结果: 使用 split() 拆分（默认空格分隔）: ['Python', 'is', 'awesome']  
  
# 使用特定的分隔符拆分字符串  
delimiter = ","  
comma_separated_str = "apple,banana,cherry"  
split_with_delimiter = comma_separated_str.split(delimiter)  
print("使用 split() 和指定分隔符 ',', 拆分字符串:", split_with_delimiter)  
# 运行结果: 使用 split() 和指定分隔符 ',', 拆分字符串: ['apple', 'banana', 'cherry']  
  
# 指定 maxsplit 参数限制拆分的次数  
limited_split = original_str.split(maxsplit=1)  
print("使用 split() 并限制拆分次数为 1:", limited_split)  
# 运行结果: 使用 split() 并限制拆分次数为 1: ['Python', 'is awesome']
```

#### rsplit()

```python
# 原始字符串，与 split() 示例相同  
original_str = "Python is awesome"  
  
# 使用 rsplit() 方法拆分字符串，默认以空格为分隔符，从字符串末尾开始拆分  
rsplit_result = original_str.rsplit()  
print("使用 rsplit() 拆分（默认空格分隔，从末尾开始）:", rsplit_result)  
# 运行结果: 使用 rsplit() 拆分（默认空格分隔，从末尾开始）: ['Python', 'is', 'awesome']  
  
# 指定 maxsplit 参数限制拆分的次数  
# 注意：由于原始字符串中没有重复的分隔符，且字符串长度足够，所以结果与 split() 相同  
limited_rsplit = original_str.rsplit(maxsplit=1)  
print("使用 rsplit() 并限制拆分次数为 1:", limited_rsplit)  
# 运行结果: 使用 rsplit() 并限制拆分次数为 1: ['Python is', 'awesome']  
  
# 当 maxsplit 为 0 时，不拆分字符串，整个字符串作为列表的唯一元素返回  
no_split = original_str.rsplit(maxsplit=0)  
print("使用 rsplit() 并设置 maxsplit 为 0:", no_split)  
# 运行结果: 使用 rsplit() 并设置 maxsplit 为 0: ['Python is awesome']
```

请注意，在rsplit()的最后一个案例中，虽然设置了maxsplit=0，但由于maxsplit的最小有效值是0（表示不拆分），所以整个字符串被作为一个整体返回在列表中。这与split()在maxsplit=0时的行为是一致的。然而，在大多数情况下，你会想要使用maxsplit来限制拆分的次数，特别是在处理可能包含多个分隔符的长字符串时。

此外，rsplit()和split()的主要区别在于拆分的方向。在大多数情况下，如果你不关心拆分的方向，那么split()就足够了。但是，在处理一些特殊情况时，比如当你想要从字符串末尾开始拆分并保留特定数量的分隔符右侧的元素时，rsplit()就非常有用。

### 字符串的判断方法

|                            |                |                                                                                                   |
|----------------------------|----------------|---------------------------------------------------------------------------------------------------|
| 判断字符串是否是有效的Python标识符（如变量名） | isidentifier() |      返回一个布尔值，如果字符串是有效的Python标识符则返回True，否则返回False。有效的标识符不能以数字开头，可以包含字母、数字和下划线，但不能包含空格、特殊字符等。       |
|       判断字符串是否只包含空白字符       |   isspace()    |                  返回一个布尔值，如果字符串中的所有字符都是空白字符（如空格、换行\n、制表符\t等）且非空则返回True，否则返回False。                  |
|      判断字符串是否只包含字母且非空       |   isalpha()    |      返回一个布尔值，如果字符串中的所有字符都是字母（包括Unicode字符库中的字母）且非空则返回True，否则返回False。字母可以是任何语言的字母，如英文、中文、日文等。       |
|    判断字符串是否只包含十进制数字字符且非空    |  isdecimal()   | 返回一个布尔值，如果字符串中的所有字符都是十进制数字字符（0-9）且非空则返回True，否则返回False。该函数仅对十进制数字有效，对于其他形式的数字（如罗马数字、全角数字）会返回False。 |
| 判断字符串是否只包含数字字符且可能包括其他形式的数字 |  isnumeric()   |  返回一个布尔值，如果字符串中的所有字符都是数字字符（包括十进制数字、罗马数字、全角数字等）则返回True，否则返回False。该函数比isdecimal()更广泛，能够识别多种形式的数字。   |
|     判断字符串是否只包含字母或数字且非空     |   isalnum()    |   返回一个布尔值，如果字符串中的所有字符都是字母或数字（包括Unicode字符库中的字母和数字）且非空则返回True，否则返回False。该函数不区分字母的大小写，且支持多种语言的字符。    |

```python
# isidentifier() 案例  
# 检查字符串是否是有效的Python标识符  
str1 = "hello_world"  
print(f"'{str1}' 是有效的Python标识符吗？ {str1.isidentifier()}")  # True  
  
str2 = "123start"  
print(f"'{str2}' 是有效的Python标识符吗？ {str2.isidentifier()}")  # False，因为以数字开头  
  
# isspace() 案例  
# 检查字符串是否只包含空白字符  
space_str = "    \t\n"  
print(f"'{space_str}' 只包含空白字符吗？ {space_str.isspace()}")  # True  
  
non_space_str = " \tHello\n"  
print(f"'{non_space_str}' 只包含空白字符吗？ {non_space_str.isspace()}")  # False，因为包含非空白字符  
  
# isalpha() 案例  
# 检查字符串是否只包含字母  
alpha_str = "helloWorld"  
print(f"'{alpha_str}' 只包含字母吗？ {alpha_str.isalpha()}")  # True，忽略大小写  
  
non_alpha_str = "hello123"  
print(f"'{non_alpha_str}' 只包含字母吗？ {non_alpha_str.isalpha()}")  # False，因为包含数字  
  
# isdecimal() 案例  
# 检查字符串是否只包含十进制数字  
decimal_str = "12345"  
print(f"'{decimal_str}' 只包含十进制数字吗？ {decimal_str.isdecimal()}")  # True  
  
non_decimal_str = "123.45"  
print(f"'{non_decimal_str}' 只包含十进制数字吗？ {non_decimal_str.isdecimal()}")  # False，因为包含小数点  
  
# isnumeric() 案例  
# 检查字符串是否只包含数字字符，包括特殊形式的数字  
numeric_str = "12345"  
print(f"'{numeric_str}' 只包含数字字符吗？ {numeric_str.isnumeric()}")  # True  
  
special_numeric_str = "½"  # 二分之一，Unicode中的数字字符  
print(f"'{special_numeric_str}' 只包含数字字符吗？ {special_numeric_str.isnumeric()}")  # True  
  
# isalnum() 案例  
# 检查字符串是否只包含字母或数字  
alnum_str = "hello123"  
print(f"'{alnum_str}' 只包含字母或数字吗？ {alnum_str.isalnum()}")  # True  
  
non_alnum_str = "hello 123"  
print(f"'{non_alnum_str}' 只包含字母或数字吗？ {non_alnum_str.isalnum()}")  # False，因为包含空格  
  
# 注意：以上案例的运行结果基于Python的默认行为，对于不同的Python版本和设置，结果应该是一致的。
```

### 字符串的替换与合并

|           |           |                                                                                                                                           |
| --------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| 替换字符串中的子串 | replace() | replace()方法用于替换字符串中的指定子字符串。它接受两个参数：要替换的子字符串和替换后的子字符串。当找到要替换的子字符串时，replace()方法会将其替换为指定的替换字符串，并返回替换后的新字符串，原始字符串不会被修改。还可以通过第三个可选参数指定最大替换次数。  |
| 合并字符串序列   | join()    | join()方法是字符串的一个方法，用于将序列（如列表、元组或字符串）中的元素以指定的字符连接生成一个新的字符串。它接受一个可迭代对象作为参数，并返回一个由序列中所有元素通过指定字符连接而成的新字符串。join()方法比传统的字符串拼接操作更高效，特别是在处理大量字符串时。 |
|           |           |                                                                                                                                           |

#### replace()

```python
# 原始字符串  
original_str = "Hello, world! This is a test."  
  
# 使用 replace() 方法替换字符串中的子串  
# 将 "world" 替换为 "Python"  
modified_str = original_str.replace("world", "Python")  
  
# 打印替换后的字符串  
print(f"原始字符串: '{original_str}'")  
print(f"替换后的字符串: '{modified_str}'")  
# 运行结果:  
# 原始字符串: 'Hello, world! This is a test.'  
# 替换后的字符串: 'Hello, Python! This is a test.'  
  
# 使用 replace() 方法并指定最大替换次数  
# 将 "is" 替换为 "was"，但只替换第一次出现的  
limited_replace_str = original_str.replace("is", "was", 1)  
  
# 打印结果  
print(f"原始字符串: '{original_str}'")  
print(f"限制替换次数后的字符串: '{limited_replace_str}'")  
# 运行结果:  
# 原始字符串: 'Hello, world! This is a test.'  
# 限制替换次数后的字符串: 'Hello, world! Thwas is a test.'
```

#### join()

```python
# 使用 join() 方法合并字符串列表  
# 定义一个字符串列表  
str_list = ["Hello", "world", "from", "Python"]  
  
# 使用空格作为分隔符合并字符串列表  
merged_str = " ".join(str_list)  
  
# 打印合并后的字符串  
print(f"合并后的字符串: '{merged_str}'")  
# 运行结果:  
# 合并后的字符串: 'Hello world from Python'  
  
# 使用不同的分隔符合并字符串列表  
# 使用逗号加空格作为分隔符  
comma_separated_str = ", ".join(str_list)  
  
# 打印结果  
print(f"逗号分隔的字符串: '{comma_separated_str}'")  
# 运行结果:  
# 逗号分隔的字符串: 'Hello, world, from, Python'
```

### 字符串的比较

在Python中，字符串的比较是基于字符串中字符的Unicode码点进行的。这意味着字符串比较是区分大小写的，并且从左到右逐个字符地进行比较，直到找到不同的字符或其中一个字符串结束。

### 概念

#### 运算符

**运算符：**\>,>=,<,<=,== ,!=

#### 比较规则

首先比较两个字符串中第一个字符，如果相等则继续比较下一个字符，依次比较下去，直到两个字符串的字符不相等时，其比较结果就是两个字符串的比较结果，两个字符串中的所有后续字符将不再被比较

#### 比较原理

两个字符串进行比较时，比较的是其irdubak value(原始值)，调用内置函数ord可以得到指定字符的ordinal value。与内置函数ord对应的是内置函数chr，调用内置函数chr时指定ordinal value可以得到其对应的字符， 原始值就是字符对应的Ascall值。

#### 区分大小写

Python中的字符串比较是区分大小写的。这意味着大写字母和小写字母被视为不同的字符，因此在比较时会根据它们的Unicode码点值来判断大小。

```python
print("a" < "A")  # 输出: True，因为'a'的Unicode码点小于'A'  
print("Apple" < "apple")  # 输出: False，因为比较的第一个字符'A'和'a'就不相等，且'A' > 'a'
```

#### 逐字符比较

字符串比较是逐字符进行的。如果两个字符串在开头的某个位置之前都是相同的字符，那么比较将继续进行，直到找到第一个不同的字符或字符串结束。

```python
print("apple" < "applier")  # 输出: True，因为'e' < 'i'  
print("hello" == "hell")    # 输出: False，因为长度不同，'o'与空字符（EOF）不等
```

#### 空字符串的比较

空字符串（“”）小于任何非空字符串。

```python
print("" < "a")  # 输出: True，空字符串小于任何非空字符串
```

#### 字符串与数字的比较

字符串和数字之间不能直接进行比较，因为它们的类型不同。如果尝试这样做，Python将抛出一个TypeError。

```python
# 尝试将字符串与数字进行比较  
# print("5" < 5)  # 这将引发TypeError
```

然而，如果字符串和数字可以转换为相同的类型（例如，字符串表示的数字），并且你希望基于它们的数值进行比较，你需要显式地将它们转换为相同的类型。

```python
print(int("5") < 5)  # 输出: False，因为转换后的整数相等
```

#### 字符串长度的影响

在逐字符比较时，字符串的长度也会影响比较结果。较短的字符串在达到其末尾时，可以视为后面跟着无限多个空字符（EOF）。因此，任何非空字符串都大于空字符串，且较短的字符串（在逐字符比较时先结束）会小于较长的字符串（如果在比较结束时还没有找到不同的字符）。

```python
print("apple" < "applepie")  # 输出: True，因为'e'之后'apple'结束，而'applepie'还有字符
```

#### \== 与 is 的区别

在Python中，字符串（String）比较时使用的 == 和 is 操作符虽然都用于比较两个对象，但它们的用途和行为有本质的区别。这种区别不仅限于字符串，也适用于Python中的所有对象。

##### \== 操作符

\==操作符用于比较两个对象的值（value）是否相等。对于字符串来说，如果两个字符串包含完全相同的字符序列（包括大小写和空格等），则这两个字符串被视为相等。

```python
str1 = "hello"  
str2 = "hello"  
  
# 使用 == 比较值  
print(str1 == str2)  # 输出: True  
  
# 即使是两个独立的字符串对象，只要它们的值相同，== 也会返回 True
```

在Python中，由于字符串是不可变的（immutable），所以Python的字符串池（string interning）机制可能会使得具有相同值的字符串字面量在内存中只存储一份副本。但这并不改变\==操作符的行为——它总是比较两个字符串的值是否相同。

##### is 操作符

is操作符用于比较两个对象的身份（identity），即它们是否是内存中的同一个对象。如果两个对象指向内存中的同一个位置，则is操作符返回True。

```python
str1 = "hello"  
str2 = "hello"  
  
# 使用 is 比较身份  
print(str1 is str2)  # 输出: True（在某些Python实现中，比如CPython，由于字符串池，这里的比较结果通常为True）  
  
# 但是，对于通过不同方式创建的字符串，即使它们的值相同，is 也可能返回 False  
str3 = str1 + ""  
print(str1 is str3)  # 输出: False  
  
# 这是因为 str3 是通过字符串拼接创建的，Python 不会将其与 str1 放入同一个字符串池中
```

需要注意的是，is操作符的行为可能会受到Python实现（比如CPython、Jython等）和版本的影响。在CPython中，对于小字符串和常见的字符串字面量，Python会使用一个字符串池来存储它们，以减少内存使用。因此，通过字符串字面量创建的相同值的字符串可能会比较为is True。然而，这种行为并不是Python语言规范的一部分，不应该被依赖。

### 字符串的驻留机制

> 字符串驻留机制是指Python解释器为了节省内存，会维护一个内部的小字符串对象池（也称为“intern pool”或“interned string cache”）。当创建新的字符串对象时，如果字符串内容满足驻留条件（通常是字符串较短且只包含ASCII字符），Python会首先检查这个字符串是否已经存在于驻留池中。如果存在，Python会返回池中已有对象的引用，而不是创建一个新的字符串对象。这样，多个变量可以引用内存中相同的字符串对象，从而减少内存占用。

#### 适用条件

1.  短字符串：在Python 3.x中，驻留通常适用于长度不超过20个字符（包括20个字符）的字符串。这个长度限制可能会根据Python版本和具体实现有所不同，但大多数情况下是20个字符。
2.  只包含ASCII字符：字符串中只包含ASCII字符时，更有可能被驻留。然而，随着Python 3对Unicode的全面支持，这个条件可能变得不那么严格，因为Python 3中的字符串默认就是Unicode字符串。但是，对于只包含ASCII字符的Unicode字符串，它们仍然有可能被驻留。
3.  编译时常量：在代码编译时创建的字符串字面量，如果满足长度条件，则会被驻留。
4.  特定字符串：如空字符串"“、表示布尔值的字符串"True”、“False"以及"None”（尽管"None"实际上不常用作字符串，但理解其背后的机制很重要）等常见字符串几乎总是被驻留。

```python
# 示例代码，展示Python的自动驻留机制  
  
# 通过字符串字面量创建两个相同的字符串  
s1 = "hello"  # 很可能被自动驻留  
s2 = "hello"  # 如果s1被驻留，s2将引用s1的同一个对象  
  
# 通过字符串乘法创建一个新的字符串对象，这个对象通常不会被自动驻留  
s3 = "hello" * 5  # 创建一个较长的字符串，不会被驻留  
  
# 定义一个与s1和s2内容相同，但通过不同方式（如切片）得到的字符串  
# 这个字符串也不会被自动驻留  
s4 = s3[:5]  # s3的前5个字符，即"hello"，但不会被驻留  
  
# 比较这些字符串对象的id，以查看它们是否指向相同的内存地址  
print(f"s1 id: {id(s1)}, s2 id: {id(s2)}, s3 id: {id(s3)}, s4 id: {id(s4)}")  
  
# 执行结果（注意：结果可能因Python版本和解释器实现而异）  
# 在大多数情况下，你会看到s1和s2有相同的id，因为它们都指向了同一个驻留的字符串对象  
# 而s3和s4则会有不同的id，因为它们不是通过驻留机制得到的  
# 例如：  
# s1 id: 4495332912, s2 id: 4495332912, s3 id: 4495349504, s4 id: 4495349488  
  
# 注释：这里的id值仅为示例，实际运行时会有不同的值  
# 重要的是要理解s1和s2的id相同，而s3和s4的id不同，这反映了驻留机制的作用
```

-   字符串字面量：直接写在代码中的字符串，如"hello"。
-   驻留池：Python解释器内部的一个存储区域，用于存储被驻留的字符串对象。
-   自动驻留：Python自动对某些字符串字面量进行驻留，以减少内存占用和提高性能。
-   驻留条件：通常包括字符串的长度（较短）和字符集（ASCII）。
-   内存地址（id）：每个Python对象都有一个唯一的内存地址，可以通过id()函数获取。如果两个对象的id相同，则它们实际上是同一个对象。

#### 优势与局限

##### 优势

1.  节省内存：通过重用现有的字符串对象，避免了不必要的内存分配和释放，从而减少了内存占用。
2.  提高性能：由于减少了内存分配和垃圾收集的开销，以及可能减少了字符串比较等操作的时间复杂度（因为可以直接比较对象引用而不是字符串内容），因此可以提高程序的性能。
3.  简化内存管理：对于开发者来说，字符串驻留机制简化了内存管理的复杂性。开发者不需要显式地管理字符串对象的生命周期，也不需要担心由于字符串对象的重复创建而导致的内存浪费。

##### 局限

1.  非ASCII字符和长字符串：包含非ASCII字符或长度较长的字符串通常不会被驻留。
2.  不可预测性：由于字符串驻留的实现细节可能因Python版本和解释器实现而异，因此很难预测哪些字符串会被驻留。
3.  特定操作不触发驻留：通过程序逻辑动态生成的字符串或字符串切片操作得到的字符串，Python不会自动进行驻留。

#### 手动驻留

如果需要手动驻留一个字符串（无论其长度如何），可以使用sys.intern()函数。这个函数会检查传入的字符串是否已经被驻留，如果没有，则将其驻留并返回驻留后的字符串对象的引用。但请注意，即使使用sys.intern()，长字符串或非ASCII字符串也可能不会按预期驻留，因为它仍然受到Python解释器内部实现细节的限制。

```python
import sys  # 导入sys模块，该模块包含了很多与Python解释器交互的函数  
  
# 定义一个普通的字符串  
s1 = "hello"  
  
# 使用sys.intern()手动驻留字符串  
s2 = sys.intern("hello")  
  
# 比较s1和s2的id（身份标识），查看它们是否指向同一个对象  
print(f"s1 id: {id(s1)}, s2 id: {id(s2)}")  # 由于字符串驻留机制，s1和s2很可能有相同的id  
  
# 为了更清楚地展示手动驻留的效果，我们创建一个新的字符串字面量，但不直接驻留它  
s3 = "hello" * 2  # 创建一个新的字符串，它是"hello"的重复  
  
# 然后，我们手动驻留这个新字符串的一个切片，这个切片理论上会驻留为"hello"  
s4 = sys.intern(s3[:5])  # s3[:5] 得到 "hello"，然后手动驻留它  
  
# 再次比较id，看看s1、s2和s4是否指向同一个对象  
print(f"s1 id: {id(s1)}, s2 id: {id(s2)}, s4 id: {id(s4)}")  # s1, s2, s4应该都有相同的id  
  
# 执行结果（注意：结果可能因Python版本和解释器实现而异，但通常s1, s2, s4的id会相同）  
# s1 id: 4495332912, s2 id: 4495332912  
# s1 id: 4495332912, s2 id: 4495332912, s4 id: 4495332912
```

-   上面的id()函数用于获取对象的“身份标识”（在CPython中，这通常是对象的内存地址）。如果两个对象的id()相同，那么它们实际上是同一个对象。
-   字符串驻留机制并不是Python语言规范的一部分，而是CPython（Python的官方实现）的一种优化手段。因此，其他Python实现（如Jython、PyPy等）可能不提供相同的驻留行为。
-   驻留的字符串必须是可哈希的，因为驻留池本质上是一个字典，其键是字符串对象。
-   驻留机制主要用于优化内存使用和可能的性能提升（通过减少字符串比较等操作的时间复杂度），但它也可能导致一些意外的副作用，比如当期望两个字符串内容相同但实际上是不同对象时。

最后，尽管上面的代码展示了如何使用sys.intern()进行手动驻留，但在实际应用中，通常不需要手动驻留字符串，因为Python的自动驻留机制已经足够有效，并且手动驻留可能会引入不必要的复杂性。

### 字符串的切片操作

> Python中的字符串切片是一个非常强大且灵活的特性，它允许你以多种方式访问和操作字符串的子序列。切片操作基本上遵循\[start:stop:step\]的模式，其中start是切片开始的索引（包含），stop是切片结束的索引（不包含），而step是切片时的步长（默认为1）。如果省略start，则默认为0；如果省略stop，则默认为字符串的长度；如果省略step，则默认为1。

#### 基本切片

-   正向切片：从左到右。

```python
s = "Hello, World!"  
print(s[0:5])  # 输出: Hello  
print(s[7:])   # 输出: World!  
print(s[:5])   # 与s[0:5]相同，输出: Hello  
print(s[6:12]) # 输出: World，注意stop索引是不包含的
```

-   反向切片：通过设置负数的step来实现从右到左的切片。

```python
s = "Hello, World!"  
print(s[-1:])  # 输出: !  
print(s[-6:-1])# 输出: World，从倒数第6个字符到倒数第1个字符之前  
print(s[::-1]) # 输出: !dlroW ,olleH，整个字符串的反转
```

#### 使用步长

步长允许你跳过一些字符。例如，如果你想要每两个字符取一个字符，可以这样做：

```python
s = "Hello, World!"  
print(s[::2])  # 输出: HloWrd!，步长为2
```

同样，步长也可以是负数，结合反向切片，可以实现一些有趣的效果：

```python
s = "Hello, World!"  
print(s[::-2]) # 输出: dloHelrW!，反向切片，步长为-2
```

#### 案例

```python
'''
字符串的切片，原理和列表是一样的
字符串切片也是会产生新的对象
[start:stop,step],分别是开始位置，结束位置，步长，步长就是每个下标的间隔
左包含右不包含
默认步长为1
'''
a1 = 'zhangsan,lisi'
a2 = a1[:8] # 因为没有写开始，所以从下标0开始到下标7
a3 = a1[9:] # 因为没有写结束，所以从下标9开始到最后
a4 = '~'
a5 = a2 + a4 + a3
print(a1,id(a1)) # zhangsan,lisi 4338145200
print(a2,id(a2)) # zhangsan 4339669680
print(a3,id(a3)) # lisi 4339667824
print(a4,id(a4)) # ~ 4338101296
print(a5,id(a5)) # zhangsan~lisi 4339668016
#5个变量的结果都是不一样的，每次都产生了一个新的对象

#步长为2的时候
print(a1[::2]) # zaga,ii
#步长为负数的时候,结果是将字符串倒序输出
print(a1[::-1]) # isil,nasgnahz

#通过反向下标截取lisi
print(a1[-4:])
```

#### 注意事项

-   切片返回的是字符串的一个新副本，原始字符串不会被修改。
-   切片时，如果start或stop索引超出了字符串的实际长度，Python会自动将其调整为字符串的起始或结束位置。
-   负数索引允许你从字符串的末尾开始计数。
-   如果step是0，将引发ValueError异常，因为步长不能为0。

### 格式化字符串

> 在Python中，字符串格式化是一种将数据嵌入到字符串中的过程，允许你创建动态的字符串内容。Python提供了多种字符串格式化的方法，包括老式的%操作符、str.format()方法以及f-string（从Python 3.6开始引入）。

#### 使用%操作符（旧式）

在Python中，使用%操作符进行字符串格式化是一种较旧但仍然广泛支持的方法。这种方法通过%符号后跟一个特定的格式字符（称为格式说明符）来指定如何格式化值，并将这些值插入到字符串中的占位符位置。

##### 语法

```python
"some string %s with a %s" % (value1, value2)
```

其中%s是字符串的占位符，%是格式化操作符，而(value1, value2)是一个元组，包含了要插入到字符串中的值。

##### 格式化符

|    |                                |
|-----|--------------------------------|
| %s |   字符串（或任何对象，通过str()函数转换为字符串）   |
| %d |             十进制整数              |
| %i |          十进制整数（与%d相同）          |
| %u |             无符号整数              |
| %o |              八进制数              |
| %x |          十六进制数（小写字母）           |
| %X |          十六进制数（大写字母）           |
| %e |       科学计数法表示的浮点数（小写字母e）       |
| %E |       科学计数法表示的浮点数（大写字母E）       |
| %f |       十进制浮点数（默认保留小数点后六位）       |
| %g | 根据值的大小自动选择%f或%e（小写字母e用于科学计数法）  |
| %G | 根据值的大小自动选择%f或%E（大写字母E用于科学计数法）  |
| %% |            字面量的%字符             |
| %c | 字符（接受整数，然后将其视为ASCII码，并打印对应的字符） |

##### 案例

```python
# 使用%s格式化字符串  
name = "Alice"  
age = 30  
greeting = "Hello, %s. You are %d years old." % (name, age)  
print(greeting)  # 输出: Hello, Alice. You are 30 years old.  
  
# 使用%d格式化十进制整数  
number = 42  
print("The answer is %d." % number)  # 输出: The answer is 42.  
  
# 使用%f格式化浮点数  
pi = 3.1415926  
print("Pi is approximately %.2f." % pi)  # 输出: Pi is approximately 3.14.  
# 注意：%.2f表示保留两位小数  
  
# 使用%x格式化十六进制数  
hex_num = 255  
print("Hexadecimal of 255 is %x." % hex_num)  # 输出: Hexadecimal of 255 is ff.  
  
# 使用%%打印字面量的%字符  
percent = 100  
print("The sale is %d%% off." % percent)  # 输出: The sale is 100% off.  
  
# 使用%c根据ASCII码打印字符  
ascii_code = 65  
print("The ASCII character for %d is %c." % (ascii_code, ascii_code))  # 输出: The ASCII character for 65 is A.
```

#### 使用str.format()方法

Python中的str.format()方法是一种非常强大且灵活的字符串格式化机制。它允许你通过花括号{}来指定占位符，并在format()方法中提供对应的值来替换这些占位符。此外，str.format()还支持多种格式化选项，如字段宽度、对齐方式、填充字符、精度等。

##### 语法

```python
# 定义一个带有占位符的字符串  
template = "Hello, {}! You are {} years old."  
# 使用format()方法替换占位符  
greeting = template.format("Alice", 30)  
# 打印结果  
print(greeting)  # 输出: Hello, Alice! You are 30 years old.
```

##### 关键字参数

你也可以通过关键字参数来指定占位符的值，这样可以使代码更加清晰。

```python
template = "Hello, {name}! You are {age} years old."  
greeting = template.format(name="Alice", age=30)  
print(greeting)  # 输出: Hello, Alice! You are 30 years old.
```

##### 索引和属性访问

format()方法还支持通过索引访问元组或列表中的元素，以及访问对象的属性。

```python
# 索引访问  
person = ("Alice", 30)  
greeting = "Hello, {0[0]}! You are {0[1]} years old.".format(person)  
print(greeting)  # 输出: Hello, Alice! You are 30 years old.  
  
# 属性访问  
class Person:  
    def __init__(self, name, age):  
        self.name = name  
        self.age = age  
  
person = Person("Bob", 25)  
greeting = "Hello, {p.name}! You are {p.age} years old.".format(p=person)  
print(greeting)  # 输出: Hello, Bob! You are 25 years old.
```

##### 格式化选项

format()方法允许你指定各种格式化选项，如字段宽度、对齐方式、填充字符等。

```python
# 字段宽度和对齐  
number = 123  
formatted_number = "{:>10}".format(number)  # 右对齐，宽度为10  
print(formatted_number)  # 输出: '      123'  
  
# 填充字符  
formatted_number = "{:*^10}".format(number)  # 使用'*'作为填充字符，右对齐，宽度为10  
print(formatted_number)  # 输出: '****123***'  
  
# 精度和小数点  
pi = 3.1415926  
formatted_pi = "{:.2f}".format(pi)  # 保留两位小数  
print(formatted_pi)  # 输出: '3.14'  
  
# 进制转换  
number = 255  
formatted_hex = "{:x}".format(number)  # 十六进制  
print(formatted_hex)  # 输出: 'ff'  
  
# 逗号作为千位分隔符  
large_number = 123456789  
formatted_large_number = "{:,}".format(large_number)  
print(formatted_large_number)  # 输出: '123,456,789'
```

##### 嵌套和组合

format()方法还支持嵌套和组合使用，以实现更复杂的格式化需求。

```python
# 嵌套使用  
name = "Alice"  
age = 30  
info = "Name: {name}, Age: {age}".format(name=name, age=age)  
greeting = "Hello, {}. {}".format(info, "Enjoy your day!")  
print(greeting)  # 输出: Hello, Name: Alice, Age: 30. Enjoy your day!  
  
# 组合使用  
formatted_string = "{0:>{width}} {1:<{width}}".format("Hello", "World", width=10)  
print(formatted_string)  # 输出: '     Hello World     '
```

#### f-string（Python 3.6+）

Python中的f-string（格式化字符串字面量）是从Python 3.6版本开始引入的一种非常简洁且强大的字符串格式化方法。f-string通过在字符串前加上f或F，并在字符串内部使用大括号{}包围变量或表达式来实现格式化。这种方式不仅代码更加简洁易读，而且执行效率也更高。

##### 语法

```python
# 定义变量  
name = "Alice"  
age = 30  
  
# 使用f-string格式化字符串  
greeting = f"Hello, {name}! You are {age} years old."  
  
# 打印结果  
print(greeting)  # 输出: Hello, Alice! You are 30 years old.
```

##### 表达式

f-string中的大括号{}内不仅可以包含变量，还可以包含任何有效的Python表达式。

```python
# 定义变量  
x = 10  
y = 20  
  
# 使用f-string包含表达式  
result = f"{x} + {y} = {x + y}"  
  
# 打印结果  
print(result)  # 输出: 10 + 20 = 30
```

##### 调用方法

你还可以在f-string中调用对象的方法。

```python
# 定义一个类  
class Person:  
    def __init__(self, name, age):  
        self.name = name  
        self.age = age  
  
    def greet(self):  
        return f"Hello, my name is {self.name} and I am {self.age} years old."  
  
# 创建Person对象  
person = Person("Bob", 25)  
  
# 使用f-string调用方法（注意：这里通常不直接在f-string中调用方法，但为了演示其可能性）  
# 更常见的做法是直接调用方法并赋值给变量，然后再在f-string中使用该变量  
greeting_method = person.greet()  
greeting_fstring = f"Person says: {greeting_method}"  
  
# 打印结果  
print(greeting_fstring)  # 输出: Person says: Hello, my name is Bob and I am 25 years old.
```

注意：虽然技术上可以在f-string中调用方法，但通常不推荐这样做，因为它可能会降低代码的可读性。更好的做法是先调用方法，然后将结果存储在变量中，再在f-string中使用该变量。

##### 格式化选项

f-string还支持与str.format()方法类似的格式化选项，尽管语法略有不同。

```python
# 定义变量  
pi = 3.1415926  
  
# 使用f-string的格式化选项  
formatted_pi = f"{pi:.2f}"  # 保留两位小数  
  
# 打印结果  
print(formatted_pi)  # 输出: 3.14  
  
# 字段宽度和对齐  
large_number = 123456789  
formatted_large_number = f"{large_number:>15,}"  # 右对齐，宽度为15，使用逗号作为千位分隔符  
  
# 打印结果  
print(formatted_large_number)  # 输出（可能因环境而异，但通常是这样）: '    123,456,789'
```

##### 注意事项

-   f-string在编译时就会进行格式化操作，因此如果在大括号{}中的表达式计算代价较大，可能会影响到程序的性能。
-   f-string不支持自动的国际化（i18n）和本地化（l10n）功能，因为它是在运行时直接插入字符串的。如果你需要支持多语言，可能需要考虑使用其他字符串格式化方法，如str.format()或%操作符，并结合适当的国际化库。

f-string以其简洁性和高效性成为了Python中字符串格式化的首选方法，尤其是在Python 3.6及更高版本中。

### 字符串编码转换(爬虫的时候需要使用)

> 在Python中，字符串的编码转换是一个重要的概念，特别是当你需要处理不同编码格式的文本数据时。Python 3中，字符串是以Unicode编码存储的，这意味着Python内部的字符串表示是统一的，但当你需要将字符串写入文件或通过网络发送时，通常需要将它们转换为特定的编码（如UTF-8、GBK、ASCII等）。

#### Unicode与编码

-   Unicode：是一个为了将世界上所有的系统和语言的文字纳入同一编码标准而设计的字符集。Unicode字符使用代码点（code point）来表示，范围从U+0000到U+10FFFF。
-   编码：是将字符集中的字符转换为字节序列的过程。常见的编码包括UTF-8、GBK、ASCII等。

#### Python中的字符串编码转换

在Python中，str类型代表Unicode字符串，而bytes类型代表字节序列。要进行编码转换，你通常需要在str和bytes之间进行转换。

-   将str转换为bytes（编码）：使用.encode()方法，并指定目标编码。
-   将bytes转换为str（解码）：使用.decode()方法，并指定源编码。

#### 案例

```python
# 定义一个Unicode字符串  
unicode_string = "你好，世界！"  
  
# 将Unicode字符串编码为UTF-8字节序列  
utf8_bytes = unicode_string.encode('utf-8')  
print(f"UTF-8编码的字节序列: {utf8_bytes}")  # 执行结果依赖于环境，但通常是b'\xe4\xbd\xa0\xe5\xa5\xbd\xef\xbc\x8c\xe4\xb8\x96\xe7\x95\x8c\xef\xbc\x81'  
  
# 将UTF-8字节序列解码为Unicode字符串  
decoded_string = utf8_bytes.decode('utf-8')  
print(f"解码后的Unicode字符串: {decoded_string}")  # 输出: 你好，世界！  
  
# 尝试使用不同的编码（如GBK）进行编码和解码  
# 注意：如果原始字符串包含无法用指定编码表示的字符，将会抛出UnicodeEncodeError或UnicodeDecodeError  
try:  
    gbk_bytes = unicode_string.encode('gbk')  
    print(f"GBK编码的字节序列: {gbk_bytes}")  # 执行结果依赖于环境，但通常是b'\xc4\xe3\xba\xc3\xef\xbc\x8c\xca\xc0\xbc\xe1\x21'  
      
    # 解码GBK字节序列  
    decoded_string_gbk = gbk_bytes.decode('gbk')  
    print(f"从GBK解码的Unicode字符串: {decoded_string_gbk}")  # 输出: 你好，世界！  
except UnicodeEncodeError as e:  
    print(f"编码错误: {e}")  
except UnicodeDecodeError as e:  
    print(f"解码错误: {e}")  
  
# 尝试使用ASCII编码进行编码（通常会导致错误，因为ASCII不支持中文字符）  
try:  
    ascii_bytes = unicode_string.encode('ascii')  
except UnicodeEncodeError as e:  
    print(f"使用ASCII编码时发生错误: {e}")  # 输出: 使用ASCII编码时发生错误: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
```

#### 注意事项

-   编码兼容性：不是所有的编码都支持所有的字符。例如，ASCII编码只支持英文和一些特殊字符，不支持中文字符。
-   错误处理：在编码或解码过程中，如果遇到无法处理的字符，Python会抛出UnicodeEncodeError或UnicodeDecodeError。你可以通过指定errors参数来处理这些错误，例如encode(‘utf-8’, ‘ignore’)会忽略无法编码的字符。
-   默认编码：Python的默认编码取决于系统和环境配置，但Python 3中的字符串总是以Unicode形式存在，与底层编码无关。
