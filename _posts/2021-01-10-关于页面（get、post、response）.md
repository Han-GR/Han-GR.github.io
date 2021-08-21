# 关于页面（get、post、response）

标签（空格分隔）： flask  页面   get  post  response

---

在局域网中让其他电脑访问我的网站：
如果想在同一个局域网下的其他电脑访问自己电脑上的Flask网站，那么可以设置`host='0.0.0.0'`才能访问得到。如：
```
app.run(host='0.0.0.0')
```
指定端口号：
Flask项目，默认使用`5000`端口。如果想更换端口，那么可以设置`port=9000`。如：
```
app.run(host='0.0.0.0',port=9000)
```

url唯一：
在定义url的时候，一定要记得在最后加一个斜杠。
1. 如果不加斜杠，那么在浏览器中访问这个url的时候，如果最后加了斜杠，那么就访问不到。这样用户体验不太好。
2. 搜索引擎会将不加斜杠的和加斜杠的视为两个不同的url。而其实加和不加斜杠的都是同一个url，那么就会给搜索引擎造成一个误解。加了斜杠，就不会出现没有斜杠的情况。

GET请求和POST请求
------------

在网络请求中有许多请求方式，比如：GET、POST、DELETE、PUT请求等。那么最常用的就是`GET`和`POST`请求了。
1. `GET`请求：只会在服务器上获取资源，不会更改服务器的状态。这种请求方式推荐使用`GET`请求。
2. `POST`请求：会给服务器提交一些数据或者文件。一般POST请求是会对服务器的状态产生影响，那么这种请求推荐使用POST请求。
3. 关于参数传递：
    `GET`请求：把参数放到`url`中，通过`?xx=xxx`的形式传递的。因为会把参数放到url中，一眼就能看到你传递给服务器的参数。这样不太安全。
    `POST`请求：把参数放到`Form Data`中。会把参数放到`Form Data`中，避免了被偷瞄的风险，但是如果别人想要偷看你的密码，那么其实可以通过抓包的形式。因为POST请求可以提交一些数据给服务器，比如可以发送文件，那么这就增加了很大的风险。所以POST请求，对于那些有经验的黑客来讲，其实是更不安全的。
4. 在`Flask`中，`route`方法，默认将只能使用`GET`的方式请求这个url，如果想要设置自己的请求方式，那么应该传递一个`methods`参数。
  如：
```
@app.route('/login/',methods=['GET','POST'])
def login():
    if request.method =='GET':
        return render_template('login.html')
    else:
        return "success"
```

页面跳转和重定向
--------

重定向分为永久性重定向和暂时性重定向，在页面上体现的操作就是浏览器会从一个页面自动跳转到另外一个页面。比如用户访问了一个需要权限的页面，但是该用户当前并没有登录，因此我们应该给他重定向到登录页面。

永久性重定向： `http`的状态码是`301`，多用于旧网址被废弃了要转到一个新的网址确保用户的访问，最经典的就是京东网站，你输入 www.jingdong.com  的时候，会被重定向到 www.jd.com  ，因为 jingdong.com，这个网址已经被废弃了，被改成jd.com，所以这种情况下应该用永久重定向。


暂时性重定向：`http`的状态码是`302`，表示页面的暂时性跳转。比如访问一个需要权限的网址，如果当前用户没有登录，应该重定向到登录页面，这种情况下，应该用暂时性重定向。

flask中重定向：重定向是通过 `redirect(location,code=302)`  这个函数来实现的， location  表示需要重定向到的 URL  ，应该配合之前讲的 url_for()  函数来使用， code  表示采用哪个重定向，默认是 302  也即 暂时性重定向  ，可以修改成 301  来实现永久性重定向。
例如：
```
from flask import Flask,request,url_for,redirect
app = Flask(__name__)
@app.route('/')
def hello_world():
    return 'Hello World!'

@app.route('/login/')
def login():
    return '这是登录页面'

#falsk中重定向
@app.route('/profile/')
def proflie():
    if request.args.get('name'):
        return '个人中心页面'
    else:
        # return redirect(url_for('login'))
        return redirect(url_for('login'),code=302)

if __name__ == '__main__':
    app.run(debug=True)
```


视图函数Response返回值
-----------------

关于响应（Response）
视图函数中可以返回以下类型的值：
1.Response 对象及其子类对象
2.字符串。其实 Flask 是根据返回的字符串类型，重新创建一个 `werkzeug.wrappers.Response` 对象， Response 将该字符串作为主体，状态码为200， MIME  类型为 text/html  ，然后返回该 Response  对象
3.元组。元组中格式是 (response,status,headers)  。 response  为一个字符串， status  值是状态码， headers是一些响应头。即(响应体,状态码,头部信息)，也不一定三个都要写，写两个也是可以的。
4.如果不是以上三种类型。即如果视图函数返回的数据，不是字符串，也不是元组，也不是Response对象，那么就会将返回值传给` Response.force_type(rv,request.environ) `，然后再将`force_type`的返回值返回给前端。

自定义的`Response`对象的步骤
1. 继承自`Response`类。
2. 实现方法`force_type(cls,rv,environ=None)`。
3. 指定`app.response_class`为你自定义的`Response`对象。

如：自定义的`Response`对象将视图中返回的字典，转换成json对象，然后返回
```
class JSONResponse(Response):
      @classmethod
      def force_type(cls, response, environ=None):
          """
          这个方法只有视图函数返回 非字符串  非Response对象  非元组时才会调用
          response：视图函数的返回值
          """
          if  isinstance(response,dict):
              response = jsonify(response)
          #调用父类方法
          return super(JSONResponse, cls).force_type(response,environ)

app.response_class = JSONResponse
```







