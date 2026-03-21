---
title: 爬虫学习笔记2-requests+BeautifulSoup
description: ""
slug: 2022-03-05-爬虫学习笔记1-基础知识
date: 2022-03-05
image: crawlSpider.jpg
categories:
  - AI&数据
tags:
  - python
  - 爬虫
---


## Request

“Requests是一个针对人类的优雅而简单的Python HTTP库。” Request 库不仅小巧，能够轻松实现对 HTML 页面的自动爬取和自动网络请求提交，是 python 网络爬虫必须学习的内容。  

### get()方法

最常用的就是 get()方法 ，语法为：

```python
requests.get(url, params=None, **kwargs)
```


|     参数      |          说明           |
|-------------|-----------------------|
|     url     |      拟获取页面的url链接      |
|   params    | url中的额外参数，字典或字节流格式，可选 |
|   **kwargs    |      12个控制访问的参数       |


```python
import requests  
r = requests.get("http://www.baidu.com")  
print(r.status_code)  
print(r.text)
```

运行结果：  
![](assets/file-20260321143154728.png)

### Request 对象

> 当我们使用 get() 方法时，就会构造一个向服务器请求资源的 Request 对象。Request对象的作用是与客户端交互，收集客户端的 Form、Cookies、超链接，或者收集服务器端的环境变量。Request 对象是从客户端向服务器发出请求，包括用户提交的信息以及客户端的一些信息。客户端可通过 HTML 表单或在网页地址后面提供参数的方法提交数据，然后服务器通过 Request 对象的相关方法来获取这些数据。Request 的各种方法主要用来处理客户端浏览器提交的请求中的各项参数和选项。

```python
requests.get("https://www.baidu.com")
```


### Response 对象

> 当我们使用 get() 方法时，将会返回一个包含服务器资源的 Response 对象，Response 对象用于动态响应客户端请示，控制发送给用户的信息，并将动态生成响应。Response 对象只提供了一个数据集合 cookie，它用于在客户端写入 cookie 值。若指定的 cookie 不存在，则创建它。若存在，则将自动进行更新，结果返回给客户端浏览器。


#### response 对象的属性

| 属性                  | 说明                            |
| ------------------- | ----------------------------- |
| r.status_code       | HTTP请求的返回状态，200表示连接成功，404表示失败 |
| r.text              | HTTP响应内容的字符串形式，即，url对应的页面内容   |
| r.encoding          | 从HTTP header中猜测的响应内容编码方式      |
| r.apparent_encoding | 从内容分析出的响应内容编码方式（备选编码方式）       |
| r.content           | HTTP响应内容的二进制形式                |




### request()方法的参数

#### 基本格式

```python
requests.request(method, url, **kwargs)
```

#### method

也就是请求方式，有 'GET'、'HEAD'、'POST'、'PUT'、'PATCH'、'DELETE'、'OPTIONS' 7 种，分别对应 HTTP 协议的 7 种操作。


####  \*\*kwargs

控制访问的参数，均为可选项

##### params：字典或字节序列，作为参数增加到url中

例如构造一个字典，通过 .request() 方法向网页提交：

```python
kv = {'key1': 'value1', 'key2': 'value2'}
r = requests.request('GET', 'https://www.baidu.com', params=kv)
print(r.url)
```

![](assets/file-20260321143614949.png)


##### data：字典、字节序列或文件对象，作为Request的对象

例如构造一个字典和字符串，通过 .request() 方法向网页提交：

```python
  
kv = {'key1': 'value1', 'key2': 'value2'}  
r = requests.request('POST', 'https://www.baidu.com', data=kv)  

body = '测试'  
r = requests.request('POST', 'https://www.baidu.com', data=body)  
```

##### json：JSON格式的数据，作为Request的内容

```python
kv = {'key1': 'value1'}
r = requests.request('POST', 'https://www.baidu.com', json=kv)
```

##### headers：字典，HTTP定制头

```python
hd = {'user-agent': 'Chrome/10'}
r = requests.request('POST', 'https://www.baidu.com', headers=hd)
```

##### files：字典类型，传输文件

```python
fs = {'file': open('data.xls','rb')}
r = requests.request('POST', 'https://www.baidu.com', files=fs)
```

##### timeout：设定超时时间，秒为单位

```python
r = requests.request('GET', 'https://www.baidu.com', timeout=10)
```

##### proxies：字典类型，设置访问代理服务器，可以增加登录认证

```python
proxies = {'http': 'http://user:pass@10.10.10.10:1234',
           'https': 'https://10.10.10.10:1234'}
r = requests.request('GET', 'https://www.baidu.com', proxies=proxies)
```

其他参数

|       参数        |               说明                |
|-----------------|---------------------------------|
|     cookies     | 字典或 CookieJar，Request 中的 cookie |
|      auth       |          元组，支持HTTP认证功能          |
| allow_redirects |    True/False，默认为True，重定向开关     |
|     stream      |  True/False，默认为True，获取内容立即下载开关  |
|     verify      |  True/False，默认为True，认证SSL证书开关   |
|      cert       |            本地SSL证书路径            |


### Request 库方法

除了上面提到的 get() 和 request() 还有其他的一些方法：

|         方法         |                   说明                   |
|--------------------|----------------------------------------|
| requests.request() |          构造一个请求，支撑一下各方法的基础方法           |
|   requests.get()   |     获取 HTML 网页的主要方法，对应于 HTTP 的 GET     |
|  requests.head()   |    获取 HTML 网页头信息的方法，对应于 HTTP 的 HEAD    |
|  requests.post()   | 向 HTML 网页提交 POST 请求的方法，对应于 HTTP 的 POST |
|   requests.put()   |  向 HTML 网页提交 PUT 请求的方法，对应于 HTTP 的 PUT  |
|  requests.patch()  |   向 HTML 网页提交局部修改请求，对应于 HTTP 的 PATCH   |
| requests.delete()  |   向 HTML 页面提交删除请求，对应于 HTTP 的 DELETE    |

#### head(): 获取 HTML 网页头信息

```python
requests.head(url, **kwargs)
```

|   参数   |     说明      |
|--------|-------------|
|  url   | 拟获取页面的url链接 |
| **kwargs | 13个控制访问的参数  |

#### post(): 向 HTML 网页提交 POST 请求

```python
requests.post(url, data=None, json=None, **kwargs)
```

|   参数   |          说明           |
|--------|-----------------------|
|  url   |      拟获取页面的url链接      |
|  data  | 字典、字节序列或文件，Request的内容 |
|  json  | JSON格式的数据，Request的内容  |
| **kwargs |      11个控制访问的参数       |

#### put(): 向 HTML 网页提交 PUT 请求

```python
requests.put(url, data=None, **kwargs)
```

|   参数   |          说明           |
|--------|-----------------------|
|  url   |      拟获取页面的url链接      |
|  data  | 字典、字节序列或文件，Request的内容 |
| **kwargs |      12个控制访问的参数       |

#### patch(): 向 HTML 网页提交局部修改请求

```python
requests.patch(url, data=None, **kwargs)
```

|   参数   |          说明           |
|--------|-----------------------|
|  url   |      拟获取页面的url链接      |
|  data  | 字典、字节序列或文件，Request的内容 |
| **kwargs |      12个控制访问的参数       |

#### delete(): 向 HTML 页面提交删除请求

```python
requests.delete(url, **kwargs)
```

|   参数   |     说明      |
|--------|-------------|
|  url   | 拟获取页面的url链接 |
| **kwargs | 12个控制访问的参数  |


### Request 库异常

|                                       异常                                       |               说明               |
|--------------------------------------------------------------------------------|--------------------------------|
|                            requests.ConnectionError                            |    网络连接错误异常，如DNS查询失败、拒绝连接等     |
|                               requests.HTTPError                               |            HTTP错误异常            |
|                              requests.URLRequired                              |            URL缺失异常             |
|                           requests.TooManyRedirects                            |       超过最大重定向次数，产生重定向异常        |
|                            requests.ConnectTimeout                             |          连接远程服务器超时异常           |
|                                requests.Timeout                                |         请求URL超时，产生超时异常         |
|                               r.raise_for_status()                               | 如果不是200，产生异常requests.HTTPError |
| 爬取信息的时候是有可能遇到异常的，因此在读取之前，应该先用 r.status_code 检查一下，若值为 200 则为成功爬取，若为 404 或其他则失败。 |                                |



## Beautiful Soup

"Beautiful Soup 是一个优秀的 HTML 页面解析的第三方库，能够对爬取来的数据进行解析、遍历、维护，是 python 网络爬虫的必学第三方库之一。

> Beautiful Soup提供一些简单的、python式的函数用来处理导航、搜索、修改分析树等功能。它是一个工具箱，通过解析文档为用户提供需要抓取的数据，因为简单，所以不需要多少代码就可以写出一个完整的应用程序。

### 说明文档和参考资料

#### 英文说明文档

[Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/)  
[Beautiful Soup Documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)

#### 中文说明文档

[Beautiful Soup 4.4.0 文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)  
[Beautiful Soup 4.4.0 文档](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id18)  

### HTML 简介

HTML语言是超文本标记语言(HyperText Markup Language)的缩写，是Web上的专用表述语言。HTML 运行在浏览器上，由浏览器来解析，它包括一系列标签．通过这些标签可以将网络上的文档格式统一，使分散的Internet资源连接为一个逻辑整体。HTML可以规定网页中信息陈列的格式，指定需要显示的图片，嵌入其他浏览器支持的描述型语言，以及指定超文本链接对象。  
HTML语言的源文件是纯文本文件，所以，可以使用任何文本编辑器进行编辑，例如我们常用的记事本。需要注意的是，HTML不是程序设计语言，而是一种标识语言，需要学习的内容是各种标记的用法。

#### 标记码

HTML语言使用的描述性标记符被称为标记码，用来指明文档的不同内容。通过标记码，能够把HTML文档划分成不同的逻辑部分或结构，如段落、标题等。可以类比于Markdown语法，标记码描述了文档的结构，向浏览器提供该文档的格式化信息，以传送文档的外观特性。

##### 标记码的格式要求

1.  任何标记码皆由“<”及“>”所围住；
2.  标记名与“<”和“>”号之间不能留有空白字符；
3.  如果标记需要加上参数，参数只可加于起始标记中；
4.  在起始标记之标记名前加上符号“／”表示为其终结标记；
5.  标记字母不区分大小写。

#### 标记码分类

标记码按形态分为围堵标记与空标记。

##### 围堵标记

也称为双标记或双标签，以起始标记及终结标记将文字围住，令其达到预期显示效果。例如：加粗

```html
<b>内容</b>
```

##### 空标记

是指标记单独出现，只有起始标记没有终结标记。例如：换行

```html
内容<br>内容
```

#### 标记码解析

|          符号          |            功能             |
|----------------------|---------------------------|
|  < !DOCTYPE html >   |       声明为 HTML5 文档        |
|  < html >…< /html >  | 是 HTML 页面的根元素，定义HTML文档的起始 |
|  < head >…< head >   |     文件头，代码区间包含文档的元数据      |
| < title >…< /title > |        代码区间定义文档的标题        |
|  < body >…< body >   |    包含了可见的页面内容，定义文档主体信息    |
|    < h1 >…< h1 >     |        代码区间定义一个大标题        |
|     < p >…< p >      |        代码区间定义一个段落         |

-   HTML5是HTML最新的修订版本，2014年10月由万维网联盟（W3C）完成标准制定。HTML5的设计目的是为了在移动设备上支持多媒体。

### Beautiful Soup 库

```python
from bs4 import BeautifulSoup
```

#### 解析器

Beautiful Soup支持Python标准库中的HTML解析器,还支持一些第三方的解析器,主要解释器如下：

|      解析器      |                                 样例                                  |                  优点                   |       缺点       |
|---------------|---------------------------------------------------------------------|---------------------------------------|----------------|
| bs4的HTML 解析器  |                BeautifulSoup(markup, "html.parser")                 |   Python的内置标准库<br>执行速度适中<br>文档容错能力强   |  部分版本中文档容错能力差  |
| lxml HTML 解析器 |                    BeautifulSoup(markup, "lxml")                    |            速度快<br>文档容错能力强             |    需要安装C语言库    |
| lxml XML 解析器  | BeautifulSoup(markup, ["lxml-xml"])<br>BeautifulSoup(markup, "xml") |          速度快<br>唯一支持XML的解析器           |    需要安装C语言库    |
| html5lib 解析器  |                  BeautifulSoup(markup, "html5lib")                  | 最好的容错性<br>以浏览器的方式解析文档<br>生成HTML5格式的文档 | 速度慢<br>不依赖外部扩展 |

-   推荐使用lxml作为解析器,因为效率更高。

#### 基本元素

BeautifulSoup 类有 5 个基本元素：

|      基本元素       |                 说明                 |
|-----------------|------------------------------------|
|       Tag       |   标签，最基本的信息组织单元，分别用<>和</>标明开头和结尾   |
|      Name       | 标签的名字，< p >…< /p >的名字是'p'，格式：.name |
|   Attributes    |       标签的属性，字典形式组织，格式：.attrs       |
| NavigableString |  标签内非属性字符串，<>…</>中字符串，格式：.string   |
|     Comment     |     标签内字符串的注释部分，一种特殊的Comment类型     |

##### Tag

```python
url = 'https://www.baidu.com/'  # 要爬取的网页，以百度为例  
headers = {  
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'}  
  
response = requests.get(url, headers=headers)  
if response.status_code == 200:  
    soup = BeautifulSoup(response.text, 'lxml')  
  
    title = soup.title  
    print(title)  
  
    tag = soup.a  
    print(tag)
```

![](assets/file-20260321155005344.png)

-   任何 HTML 语法中的标签可以使用 soup. 进行获取，当 HTML 文档中存在多个相同 对应内容时，soup. 仅返回第一个标签。

##### Name

```python
import requests  
from bs4 import BeautifulSoup  
from lxml import html  
  
url = 'https://www.baidu.com/'  # 要爬取的网页，以百度为例  
headers = {  
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'}  
  
response = requests.get(url, headers=headers)  
if response.status_code == 200:  
    soup = BeautifulSoup(response.text, 'lxml')  
  
    name = soup.a.name  
    print(name)  
  
    name2 = soup.a.parent.name  
    print(name2)  
  
    name3 = soup.a.parent.parent.name  
    print(name3)
```

![](assets/file-20260321155312054.png)

- Tag 对象与 XML 或 HTML 原生文档中的 tag 相同。  
- 每个 tag 都有自己的名字,通过 .name 来获取，如果改变了 tag 的 name,那将影响所有通过当前 Beautiful Soup 对象生成的HTML文档。

##### Attributes

```python
import requests  
from bs4 import BeautifulSoup  
from lxml import html  
  
url = 'https://www.baidu.com/'  # 要爬取的网页，以百度为例  
headers = {  
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'}  
  
response = requests.get(url, headers=headers)  
if response.status_code == 200:  
    soup = BeautifulSoup(response.text, 'lxml')  
  
    tag = soup.a  
    print(tag)  

    tag_attrs = tag.attrs  
    print(tag_attrs)  
  
    tag_class = tag_attrs['class']  
    print(tag_class)  
  
    tag_href = tag_attrs['href']  
    print(tag_href)
```

![](assets/file-20260321155758711.png)

- 一个 tag 可能有很多个属性，tag 的属性的操作方法与字典相同

##### NavigableString

```python
import requests  
from bs4 import BeautifulSoup  
from lxml import html  
  
url = 'https://www.baidu.com/'  # 要爬取的网页，以百度为例  
headers = {  
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'}  
  
response = requests.get(url, headers=headers)  
if response.status_code == 200:  
    soup = BeautifulSoup(response.text, 'lxml')  
  
    a = soup.a  
    print(a)  
  
    a_s = soup.a.string  
    print(a_s)  
  
    p = soup.p  
    print(p)  
  
    p_s = soup.p.string  
    print(p_s)
```

![](assets/file-20260321160439642.png)



### HTML 信息遍历
由于 HTML 通过了标记码划分了所属关系，是一个树结构。通过数据结构中的知识可知，树结构按照一定的规则可以对其进行遍历，我们可以使用内置的方法实现。  

#### 下行遍历
|      属性      |                说明                |
|--------------|----------------------------------|
|  .contents   |        子节点的列表，将所有儿子节点存入列表        |
|  .children   | 子节点的迭代类型，与.contents类似，用于循环遍历儿子节点 |
| .descendants |    子孙节点的迭代类型，包含所有子孙节点，用于循环遍历     |

```python
import requests  
from bs4 import BeautifulSoup  
from lxml import html  
  
url = 'https://www.baidu.com/'  # 要爬取的网页，以百度为例  
headers = {  
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'}  
  
response = requests.get(url, headers=headers)  
if response.status_code == 200:  
    soup = BeautifulSoup(response.text, 'lxml')  
  
    head = soup.head  
    print(head)  
  
    head_contents = soup.head.contents  
    print(head_contents)  
  
    body_contents = soup.body.contents  
    print(body_contents)  
  
    for child in soup.body.contents:  
        print(child)  
  
    for child in soup.body.children:  
        print(child)  
  
    for child in soup.body.descendants:  
        print(child)
```

- 结果太长了，就不放了，知道怎么用就行

#### 上行遍历

|      属性      |           说明           |
|--------------|------------------------|
|   .parent    |        节点的父亲标签         |
|   .parents   | 节点先辈标签的迭代类型，用于循环遍历先辈节点 |

```python
import requests  
from bs4 import BeautifulSoup  
from lxml import html  
  
url = 'https://www.baidu.com/'  # 要爬取的网页，以百度为例  
headers = {  
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'}  
  
response = requests.get(url, headers=headers)  
if response.status_code == 200:  
    soup = BeautifulSoup(response.text, 'lxml')  

    for parent in soup.a.parents:  
        if parent is None:  
            print(parent)  
        else:  
            print(parent.name)
```

![](assets/file-20260321161549903.png)

#### 平行遍历

|               属性                |              说明              |
|---------------------------------|------------------------------|
|          .next_sibling          |    返回按照HTML文本顺序的下一个平行节点标签    |
|        .previous_sibling        |    返回按照HTML文本顺序的上一个平行节点标签    |
|         .next_siblings          | 迭代类型，返回按照HTML文本顺序的后续所有平行节点标签 |
|       .previous_siblings        | 迭代类型，返回按照HTML文本顺序的前续所有平行节点标签 |

```python
import requests  
from bs4 import BeautifulSoup  
from lxml import html  
  
url = 'https://www.baidu.com/'  # 要爬取的网页，以百度为例  
headers = {  
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'}  
  
response = requests.get(url, headers=headers)  
if response.status_code == 200:  
    soup = BeautifulSoup(response.text, 'lxml')  
  
    for sibling in soup.a.next_sibling:  
        print(sibling)  
  
    for sibling in soup.a.next_siblings:  
        print(sibling)  
  
    for sibling in soup.a.previous_sibling:  
        print(sibling)  
  
    for sibling in soup.a.previous_siblings:  
        print(sibling)
```

![](assets/file-20260321162045720.png)

### 信息提取

### find 方法

方法将返回一个列表类型，存储查找的结果。

```python
soup.find_all(name, attrs, recursive, string, **kwargs)
```

|    参数     |          说明          |
|-----------|----------------------|
|   name    |     对标签名称的检索字符串      |
|   attrs   | 对标签属性值的检索字符串，可标注属性检索 |
| recursive |   是否对子孙全部检索，默认True   |
|  string   |  <>…</>中字符串区域的检索字符串  |

```python
import re  
  
import requests  
from bs4 import BeautifulSoup  
from lxml import html  
  
url = 'https://www.baidu.com/'  # 要爬取的网页，以百度为例  
headers = {  
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'}  
  
response = requests.get(url, headers=headers)  
if response.status_code == 200:  
    soup = BeautifulSoup(response.text, 'lxml')  
	
    a = soup.find_all(name='a', attrs={'href': re.compile('^https?://.*')}, recursive=True, string=re.compile('文'))  
    print(a)
```

代码解释：
- `name='a'`：只查找`<a>`（超链接）标签
- `attrs={'href': re.compile('^https?://.*')}`：要求标签的`href`（链接地址）属性满足正则`^https?://.*`：也就是必须是以`http://`或者`https://`开头的外部链接，排除百度站内的相对路径链接
- `recursive=True`：递归搜索，搜索当前节点下所有子层级的标签（这个默认就是True，这里显式写出来了）
- `string=re.compile('文')`：要求标签内部的文本，必须包含"文"这个字，用正则做模糊匹配，只要带这个字就会被匹配到

运行结果：
![](assets/file-20260321163103102.png)


#### 拓展方法
|            方法             |                说明                |
|---------------------------|----------------------------------|
|         <>.find()         |    搜索且只返回一个结果，同.find_all()参数     |
|     <>.find_parents()     |  在先辈节点中搜索，返回列表类型，同.find_all()参数  |
|     <>.find_parent()      |     在先辈节点中返回一个结果，同.find()参数      |
|   <>.find_next_siblings()   | 在后续平行节点中搜索，返回列表类型，同.find_all()参数 |
|   <>.find_next_sibling()    |    在后续平行节点中返回一个结果，同.find()参数     |
| <>.find_previous_siblings() | 在前序平行节点中搜索，返回列表类型，同.find_all()参数 |
| <>.find_previous_sibling()  |    在前序平行节点中返回一个结果，同.find()参数     |


## 实操：爬取中国大学排名

### 实例解析

从[中国最好大学排名](https://www.shanghairanking.cn/rankings/bcur/202111)提取大学排名信息并输出排名，大学名称，总分的信息，实现的是定向爬虫，即仅对输入 URL 进行爬取，不扩展爬取其他 URL。由于这个网页的信息是静态存放于网页源码中的，因此可以用 Requests 和 Beautiful Soup 库进行爬取。

### 代码
```python
import requests  
from bs4 import BeautifulSoup  
  
"""  
获取网页源码  
"""  
def get_html_response(url):  
    response = requests.get(url)  
    try:  
        if response.status_code == 200:  
            response.encoding = 'utf-8'  # 加这句是因为正常解析出来，中文会乱码  
            return response.text  
        else:  
            print(f"网页返回的状态码是{response.status_code}")  
    except Exception as e:  
        print(f"发生错误: {e}")  
  
"""  
使用BeautifulSoup解析网页  
"""  
def parse_html(resp):  
    soup = BeautifulSoup(resp, "lxml")  
  
    trs = soup.find_all(name='tr')  
  
    for tr in trs:  
        tds = tr.find_all(name='td')  
        if tds:  
  
            u_rank = tds[0].text.split()[0]  
  
            u_name_and_attr = tds[1].text.replace("\n", "").split()  
  
            u_cn_name = u_name_and_attr[0]  
  
            u_attr = u_name_and_attr[-1]  
  
            u_city = tds[2].text.split()[0]  
  
            u_type = tds[3].text.split()[0]  
  
            u_score = tds[4].text.split()[0]  
  
            print(f"{u_rank:<5}{u_cn_name:<20}{u_attr:<20}{u_city:<6}{u_type:<8}{u_score}")  
  
  
if __name__ == '__main__':  
    url = 'https://www.shanghairanking.cn/rankings/bcur/202111'  
    resp = get_html_response(url)  
  
    header = ["排名", "名称", "学校属性", "省市", "学校类型", "总分"]  
  
    print(f"{header[0]:<5}{header[1]:<20}{header[2]:<20}{header[3]:<6}{header[4]:<8}{header[5]}")  
  
    parse_html(resp)
```

### 运行结果

![](assets/file-20260321180208677.png)




