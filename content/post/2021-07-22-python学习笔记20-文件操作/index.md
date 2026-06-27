---
title: python学习笔记20-文件操作
description: ""
slug: 2021-07-22-python学习笔记20-文件操作
date: 2021-07-22
image: python.webp
categories:
  - 编程语言
tags:
  - python
  - 文件
---

## 1. 文件是什么？

文件是以计算机硬盘为载体存储在计算机上的信息集合，文件可以是文本文档、图片、程序等等。计算机文件基本上分为两种：
- 二进制文件：没有统一字符编码（图片/音频/视频/可执行文件等）
- 纯文本文件：有统一编码（可视为存储在磁盘上的长字符串）

常见纯文本编码：ASCII、ISO-8859-1、GB2312、GBK、UTF-8、UTF-16 等。实践中处理文本文件通常选 `UTF-8`。

典型 I/O 流程：打开 → 读/写 → 刷新/移动指针 → 关闭  

## 2. open() 打开文件与关闭文件
### 2.1 open 基本语法与参数

```python
file_object = open(
    file_path,
    mode='r',
    encoding=None,
    errors=None,
    newline=None,
    closefd=True,
    opener=None
)
```

- `file_path`：文件路径（相对/绝对）
- `mode`：读写模式（见下一节）
- `encoding`：文本编码（建议显式指定 `utf-8`）
- `errors`：编码/解码错误处理策略
- `newline`：换行符处理（写 CSV 常用 `newline=''`）
- `closefd`/`opener`：高级参数，通常不需要

### 2.2 推荐用法：with 自动关闭

```python
file_path = 'example.txt'
with open(file_path, 'r', encoding='utf-8') as file:
    file_content = file.read()
    print(file_content)
```

### 2.3 手动关闭：close()

```python
file_path = 'example.txt'
file = open(file_path, 'r', encoding='utf-8')
try:
    file_content = file.read()
    print(file_content)
finally:
    file.close()
```

## 3. 文件读写模式

| 模式 | 含义 |
|-----|------|
| r | 只读（默认，文件必须存在，不存在抛异常） |
| w | 只写，写之前清空文件内容；文件不存在则创建 |
| a | 追加写，文件指针在末尾；文件不存在则创建 |
| r+ | 可读可写（文件必须存在） |
| w+ | 读写（覆盖）；文件不存在则创建 |
| a+ | 读写（追加）；文件不存在则创建 |
| rb | 二进制只读 |
| wb | 二进制只写（覆盖） |
| ab | 二进制追加 |
| rb+ | 二进制可读可写 |
| wb+ | 二进制读写（覆盖） |
| ab+ | 二进制读写（追加） |

## 4. 文件对象常用方法

| 分类 | 方法 | 说明 | 示例 |
|---|---|---|---|
| 打开文件 | `open(file, mode='r', ...)` | 返回文件对象 | `file = open('example.txt', 'r')` |
| 读取 | `read(size=-1)` | 读取整个文件或指定大小 | `content = file.read()` |
| 读取 | `readline(size=-1)` | 读取一行 | `line = file.readline()` |
| 读取 | `readlines(hint=-1)` | 读取全部行 → 列表 | `lines = file.readlines()` |
| 写入 | `write(string)` | 写入字符串/bytes | `file.write('Hello\\n')` |
| 写入 | `writelines(lines)` | 写入多行（不自动加换行） | `file.writelines(['a\\n','b\\n'])` |
| 指针 | `seek(offset, whence=0)` | 移动文件指针 | `file.seek(0)` |
| 指针 | `tell()` | 返回当前指针 | `pos = file.tell()` |
| 缓冲 | `flush()` | 刷新缓冲区 | `file.flush()` |
| 关闭 | `close()` | 关闭文件 | `file.close()` |
| 上下文 | `with open(...) as f:` | 自动管理打开/关闭 | `with open(...) as f: ...` |
| 二进制 | `open(file, 'rb')` | 二进制读 | `with open('a.png','rb') as f:` |

## 5. 读取文件（read / readline / readlines）

### 5.1 read：读取全部内容

```python
file_path = 'example.txt'
with open(file_path, 'r', encoding='utf-8') as file:
    data = file.read()
    print(data)
```

### 5.2 readline：逐行读取

```python
with open('file.txt', 'r', encoding='utf-8') as file:
    line = file.readline()
```

逐行读取整个文件：

```python
with open('file.txt', 'r', encoding='utf-8') as file:
    line = file.readline()
    while line != '':
        print(line.strip())
        line = file.readline()
```

### 5.3 readlines：一次性读取全部行

```python
with open('file.txt', 'r', encoding='utf-8') as file:
    lines = file.readlines()
print(lines)
print(lines[0].strip())
```

### 5.4 readlines vs readline
- `readlines()`：一次性读取所有行 → 列表，适合小文件
- `readline()`：一次读取一行，适合大文件逐行处理，内存占用更小

### 5.5 大文件读取建议

```python
with open('big.txt', 'r', encoding='utf-8') as f:
    for line in f:
        process(line)

with open('big.bin', 'rb') as f:
    for chunk in iter(lambda: f.read(1024 * 1024), b''):
        process(chunk)
```

## 6. 写入文件（write）

### 6.1 写入文本文件

```python
file_path = 'example.txt'
with open(file_path, 'w', encoding='utf-8') as file:
    file.write("Hello, this is some data.")
```

### 6.2 写入 CSV 文件

```python
import csv

csv_file_path = 'example.csv'
data = [
    ['Name', 'Age', 'Occupation'],
    ['John Doe', 30, 'Engineer'],
    ['Jane Smith', 25, 'Designer']
]

with open(csv_file_path, 'w', newline='', encoding='utf-8') as csvfile:
    csv_writer = csv.writer(csvfile)
    csv_writer.writerows(data)
```

### 6.3 写入 JSON 文件

```python
import json

json_file_path = 'example.json'
data = {"name": "John Doe", "age": 30, "occupation": "Engineer"}

with open(json_file_path, 'w', encoding='utf-8') as jsonfile:
    json.dump(data, jsonfile, ensure_ascii=False, indent=2)
```

### 6.4 写入数据库（以 sqlite3 为例）

```python
import sqlite3

conn = sqlite3.connect('example.db')
cursor = conn.cursor()
cursor.execute('CREATE TABLE IF NOT EXISTS users (name TEXT, age INTEGER, occupation TEXT)')
cursor.execute(
    "INSERT INTO users (name, age, occupation) VALUES (?, ?, ?)",
    ('John Doe', 30, 'Engineer')
)
conn.commit()
conn.close()
```

## 7. 从文件/数据库读取数据（含示例）

### 7.1 读取 CSV

```python
import csv

csv_file_path = 'example.csv'
with open(csv_file_path, 'r', encoding='utf-8') as csvfile:
    csv_reader = csv.reader(csvfile)
    for row in csv_reader:
        print(row)
```

### 7.2 读取 JSON

```python
import json

json_file_path = 'example.json'
with open(json_file_path, 'r', encoding='utf-8') as jsonfile:
    data = json.load(jsonfile)
    print(data)
```

### 7.3 从数据库读取（sqlite3）

```python
import sqlite3

conn = sqlite3.connect('example.db')
cursor = conn.cursor()
cursor.execute("SELECT * FROM users")
rows = cursor.fetchall()
for row in rows:
    print(row)
conn.close()
```

## 8. 二进制文件操作

```python
with open('example.bin', 'wb') as file:
    file.write(b'\x00\x01\x02\x03\x04')

with open('example.bin', 'rb') as file:
    content = file.read()
    print(content)
```

## 9. 文件指针与缓冲（seek / tell / flush）

```python
with open('example.txt', 'r+', encoding='utf-8') as file:
    content = file.read(5)
    print(content)
    file.seek(0)
    file.write('Hi ')
    file.flush()
    position = file.tell()
    print(position)
```

- `seek(offset, whence=0)`：改变指针位置，`whence=0/1/2` 对应 开头/当前位置/末尾
- `tell()`：返回指针当前位置
- `flush()`：把缓冲区数据立即写入磁盘

## 10. 使用 os 与 pathlib 进行文件/目录操作

### 10.1 os 常用

```python
import os

os.makedirs('new_directory', exist_ok=True)
os.remove('example.txt')
os.rename('old_name.txt', 'new_name.txt')
print(os.listdir('.'))
```

### 10.2 pathlib 常用

```python
from pathlib import Path

path = Path('example.txt')
if path.exists():
    print('File exists.')

Path('new_directory').mkdir(parents=True, exist_ok=True)
content = path.read_text(encoding='utf-8')
print(content)
```

## 11. 文件相关管理操作（重命名/删除/创建/当前目录）

### 11.1 文件重命名（含异常处理）

```python
import os

directory = 'path_to_directory'
files = os.listdir(directory)

try:
    for file_name in files:
        full_path = os.path.join(directory, file_name)
        if os.path.isfile(full_path):
            new_filename = 'new_name'
            os.rename(full_path, os.path.join(directory, new_filename))
except OSError as e:
    print(f'Error occurred: {e}')
```

高级：正则批量生成新名称：

```python
import os
import re

directory = 'path_to_directory'
files = os.listdir(directory)

for file_name in files:
    src = os.path.join(directory, file_name)
    if os.path.isfile(src):
        new_filename = re.sub(r'\d+', 'new_prefix', file_name)
        dst = os.path.join(directory, new_filename)
        try:
            os.rename(src, dst)
        except OSError as e:
            print(f'Error renaming {file_name}: {e}')
```

### 11.2 删除文件

```python
import os

file_path = 'path_to_file'
if os.path.isfile(file_path):
    try:
        os.remove(file_path)
        print(f'File {file_path} deleted successfully.')
    except OSError as e:
        print(f'Error occurred: {e}')
```

### 11.3 创建文件

```python
import os

file_path = 'path_to_file'
if not os.path.exists(file_path):
    try:
        with open(file_path, 'w', encoding='utf-8') as f:
            pass
    except IOError as e:
        print(f'Error occurred: {e}')
```

### 11.4 获取当前目录

```python
import os

current_directory = os.getcwd()
print(f'Current directory is: {current_directory}')
```

## 12. 示例脚本

### 12.1 目录.txt 自动清洗
规则：
- 二级标题行前空 4 个格子（一级标题不加）
- “章/节” 后加空格
- 页码前加 `=>`

```python
import os
import re

desktop_path = os.path.join(os.path.expanduser("~"), "Desktop")
file_path = os.path.join(desktop_path, "目录.txt")

with open(file_path, 'r', encoding='utf-8') as file:
    lines = file.readlines()

modified_lines = []
for line in lines:
    line = line.replace(" ", "")
    if len(line) == 1:
        continue

    line = re.sub(r'(章|节)(?![ ])', r'\1 ', line)
    line = re.sub(r'(\.\d)', r'\1 ', line)

    if '章' not in line:
        line = ' ' * 4 + line

    line = re.sub(r'\(([\d\s]+)\)', r'\1', line)
    line = re.sub(r'（([^）]+)）', r'\1', line)

    if '=>' not in line:
        line = re.sub(r'(\d+)$', r'=>\1', line)

    line = re.sub(r'([\u4e00-\u9fff]+)[^\w\s]+=>', r'\1=>', line)
    modified_lines.append(line)

with open(file_path, 'w', encoding='utf-8') as file:
    file.writelines(modified_lines)

with open(file_path, 'r', encoding='utf-8') as file:
    print(file.read())
```

### 12.2 批量修改文件夹下文件命名（补齐前缀）

```python
import os

directory_path = r'目标文件夹绝对路径'
prefix = '00000000000'

files = os.listdir(directory_path)
for file_name in files:
    file_path = os.path.join(directory_path, file_name)

    if file_name.lower().endswith(('.png', '.jpg', '.jpeg', '.gif')) and '_' in file_name:
        parts = file_name.split('_')
        if parts[0] != prefix:
            new_file_name = prefix + '_' + '_'.join(parts[1:])
            new_file_path = os.path.join(directory_path, new_file_name)
            os.rename(file_path, new_file_path)
            print(f'Renamed: {file_name} -> {new_file_name}')
```

### 12.3 检测同级目录下是否存在同名文件夹（前 5 位重复）

```python
import os

directory_path = r'目标路径'
folders = [
    folder
    for folder in os.listdir(directory_path)
    if os.path.isdir(os.path.join(directory_path, folder))
]

same_prefix_folders = {}
for folder in folders:
    prefix = folder[:5]
    if prefix in same_prefix_folders:
        same_prefix_folders[prefix].append(folder)
    else:
        same_prefix_folders[prefix] = [folder]

for prefix, items in same_prefix_folders.items():
    if len(items) > 1:
        print(f\"前5位为 '{prefix}' 的文件夹有以下重复命名：\")
        print(', '.join(items))
```

## 13. 安全与健壮性
- 文本文件尽量显式指定 `encoding='utf-8'`
- 读取/写入时根据需要设置 `errors='ignore'/'replace'`
- 写 CSV 建议 `newline=''`
- 原子写入：临时文件写完再替换，避免半成品

```python
from pathlib import Path
import os
import tempfile

data = 'content'
target = Path('safe.txt')
with tempfile.NamedTemporaryFile('w', delete=False, encoding='utf-8') as tmp:
    tmp.write(data)
    tmp_path = Path(tmp.name)
os.replace(tmp_path, target)
```

## 14. 小结
- 基础：`with open(path, 'r/w/a', encoding='utf-8')`
- 大文件：优先逐行/分块，不要一次 `read()` 全部
- CSV：写入 `newline=''`
- JSON：写入 `ensure_ascii=False` + `indent`
- 二进制：加 `b`（`rb/wb/ab`）
- 目录/路径：`pathlib.Path` 更顺手
