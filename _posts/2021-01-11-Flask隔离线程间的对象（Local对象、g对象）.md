# Flask隔离线程间的对象（Local对象、g对象）

标签（空格分隔）： flask  隔离线程  local对象  g对象

---

### Local对象隔离线程间的对象_即ThreadLocal变量

#### Local对象：
在Flask中，类似于`request`对象，其实是绑定到了一个`werkzeug.local.Local`对象上。
这样，即使是同一个对象，那么在多个线程中都是隔离的。类似的对象还有`session`以及`g`对象。
```
from werkzeug.local import Local
# flask=werkzeug + sqlalchemy + jinja2
```       
![浏览器错误请求](threadlocal1.png)
![浏览器正确请求](threadlocal2.png) 

 
#### ThreadLocal变量：
Python提供了ThreadLocal变量，它本身是一个全局变量，但是每个线程却可以利用它来保存属于自己的私有数据，这些私有数据对其他线程也是不可见的。

#### 总结：
只要满足绑定到"local"或"Local"对象上的属性，在每个线程中都是隔离的，那么他就叫做`ThreadLocal`对象,也叫'ThreadLocal'变量。

#### 代码演示：
```
from threading import Thread,local
local =local()
local.request = '具体用户的请求对象'
class MyThread(Thread):
    def run(self):
        local.request = 'haha'
        print('子线程：',local.request)

mythread = MyThread()
mythread.start()
mythread.join()
print('主线程：',local.request)


from werkzeug.local import Local
local = Local()
local.request = '具体用户的请求对象'
class MyThread(Thread):
    def run(self):
        local.request = 'tantan'
        print('子线程：',local.request)

mythread = MyThread()
mythread.start()
mythread.join()

print('主线程：',local.request)
```    


### Flask_线程隔离的g对象：

#### 保存为全局对象g对象的好处：

g对象是在整个Flask应用运行期间都是可以使用的。并且也跟request一样，是线程隔离。
这个对象是专门用来存储开发者自己定义的一些数据，方便在整个Flask程序中都可以使用。一般使用就是，将一些经常会用到的数据绑定到上面，以后就直接从g上面取就可以了，而不需要通过传参的形式，这样更加方便。

#### g对象使用场景：
有一个工具类utils.py 和 用户办理业务：
```
def funa(uname):
    print('funa  %s' % uname)

def funb(uname):
    print('funb %s' % uname)

def func(uname):
    print('func %s' % uname)
```

用户办理业务：
```
from flask import Flask,request
from  utils import  funa,funb,func
app = Flask(__name__)

# Flask_线程隔离的g对象使用详解
@app.route("/profile/")
def my_profile():
    #从url中取参
    uname = request.args.get('uname')
    #调用功能函数办理业务
    funa(uname)
    funb(uname)
    func(uname)
    #每次都得传参 麻烦，引入g对象进行优化
    return "办理业务成功"

if __name__ == '__main__':
    app.run(debug=True)
```

优化工具类utils.py：
```
from flask import g

def funa():
    print('funa  %s' % g.uname)

def funb():
    print('funb %s' % g.uname)

def func():
    print('func %s' % g.uname)
```

 优化用户办理业务：
```
from flask import Flask,request,g
from utils import  funa,funb,func
app = Flask(__name__)

#Flask_线程隔离的g对象使用详解
@app.route("/profile/")
def my_profile():
    #从url中取参
    uname = request.args.get('uname')
    #调用功能函数办理业务
    # funa(uname)
    # funb(uname)
    # func(uname)
    #每次都得传参 麻烦，引入g对象进行优化
    g.uname = uname
    funa()
    funb()
    func()
    return "办理业务成功"

if __name__ == '__main__':
    app.run(debug=True)
```




