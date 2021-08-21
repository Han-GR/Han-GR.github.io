# Flask上下文（app上下文，request上下文）

标签（空格分隔）： flask  上下文 

---
### Flask_app上下文详解:

app上下文，也叫应用上下文：
是存放到一个`LocalStack`的栈中。和应用app相关的操作就必须要用到应用上下文，
比如通过`current_app`获取当前的这个`app`名字。

- 注意1：
在视图函数中，不用担心应用上下文的问题。因为视图函数要执行，那么肯定是通过访问url的方式执行的，那么这种情况下，Flask底层就已经自动的帮我们把应用上下文都推入到了相应的栈中。

- 注意2：
如果想要在视图函数外面执行相关的操作，比如获取当前的app名称，那么就必须要手动推入应用上下文：

第一种方式：便于理解的写法
```
from flask import Flask,current_app

app = Flask(__name__)

#app上下文
app_context = app.app_context()
app_context.push()
print(current_app.name)

@app.route('/')
def hello_world():
    print(current_app.name) #获取应用的名称
    return 'Hello World!'

if __name__ == '__main__':
    app.run(debug=True)
```

第二种方式：用with语句
```
from flask import Flask,current_app

app = Flask(__name__)

#app上下文
#换一种写法
with app.app_context():
   print(current_app.name)

@app.route('/')
def hello_world():
    print(current_app.name) #获取应用的名称
    return 'Hello World!'


if __name__ == '__main__':
    app.run(debug=True)
```

图解：

![app上下文](appcontext.png)



### Flask_request上下文详解：

request上下文，也叫请求上下文：
也是存放到一个`LocalStack`的栈中。和请求相关的操作就必须用到请求上下文，比如使用`url_for`反转视图函数。

- 注意1：
在视图函数中，不用担心请求上下文的问题。因为视图函数要执行，那么肯定是通过访问url的方式执行的，那么这种情况下，Flask底层就已经自动的帮我们把应用上下文和请求上下文都推入到了相应的栈中。

- 注意2：
如果想要在视图函数外面执行相关的操作，比如反转url，那么就必须要手动推入请求上下文：
底层代码执行说明：
111.推入请求上下文到栈中，会首先判断有没有应用上下文，
2.如果没有那么就会先推入应用上下文到栈中，
3.然后再推入请求上下文到栈中
```
from flask import Flask,url_for

app = Flask(__name__)

#request上下文
@app.route('/hi')
def hi():
    print(url_for('my_list')) #获取构建得到的url
    return 'Hello World!'
@app.route('/list/')
def my_list():
    return '返回列表'

# print(url_for('my_list')) #获取构建得到的url
# with app.app_context():
#     print(url_for('my_list'))
#查看报错源码

with app.test_request_context():
    #手动推入一个请求上下文到请求上下文栈中
    #如果当前app应用上下文栈中没有app应用上下文
    #那么会首先推入一个app应用上下文到栈中
    print(url_for('my_list'))

if __name__ == '__main__':
    app.run(debug=True)
```

图解：
 
![request上下文](requestcontext.png) 

 

### 总结：
#### 为什么上下文需要放在栈中？

1.应用上下文：Flask底层是基于werkzeug，werkzeug是可以包含多个app的，所以这时候用一个栈来保存。如果你在使用app1，那么app1应该是要在栈的顶部，如果用完了app1，那么app1应该从栈中删除。方便其他代码使用下面的app。

2.请求上下文：如果在写测试代码，或者离线脚本的时候，我们有时候可能需要创建多个请求上下文，这时候就需要存放到一个栈中了。使用哪个请求上下文的时候，就把对应的请求上下文放到栈的顶部，用完了就要把这个请求上下文从栈中移除掉。




 






