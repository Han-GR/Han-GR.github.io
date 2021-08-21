# url_for 和自定义url转换器

标签（空格分隔）： url  flask

---

在此输入正文


url_for的使用：
一般我们通过一个 URL就可以执行到某一个函数。如果反过来，我们知道一个函数，怎么去获得这个 URL 呢？ url_for 函数就可以帮我们实现这个功能。
 
url_for()  ：函数接收两个及以上的参数，他接收函数名作为第一个参数，接收对应URL规则的命名参数，如果还出现其他的参数，则会添加到 URL  的后面作为查询参数。
如：
```
@app.route('/post/list/<page>/')
def my_list(page):
    return 'my list'

@app.route('/')
def hello_world():
# 构建出来的url：/post/my_list/2?num=8
return url_for('my_list',page=2,num=8)
# return "/post/list/2？num=8"
```
为什么选择`url_for` 而不选择直接在代码中拼 URL  的原因有两点：
1. 将来如果修改了 URL  ，但没有修改该 URL  对应的函数名，就不用到处去替换 URL  了。
2.  url_for()  函数会转义一些特殊字符和 unicode  字符串，这些事情 url_for  会自动的帮我们
搞定。
如：
```
@app.route('/login/')
def login():
    return 'login'

@app.route('/')
def hello_world():
return url_for('login', next='/')
# /login/?next=/
# 会自动的将/编码，不需要手动去处理。
# url=/login/?next=%2F
```

## 自定义url转换器##

自定义URL转换器的步骤：
转换器是一个类，且必须继承自`werkzeug.routing.BaseConverter`。
1. 实现一个类，继承自`BaseConverter`。
2. 在自定义的类中，重写`regex`，也就是这个变量的正则表达式。
3. 将自定义的类，映射到`app.url_map.converters`上。理解为加入字典`DEFAULT_CONVERTERS`中
比如：
```
app.url_map.converters['tel']=TelephoneConveter
```

注意：to_python 方法和 to_url 方法的作用
 1.在转换器类中，实现to_python(self,value)方法，这个方法的返回值，将会传递到 view函数中作为参数。
 2.在转换器类中，实现to_url(self,values)方法，这个方法的返回值，将会在调用url_for函数的时候生成符合要求的URL形式。

