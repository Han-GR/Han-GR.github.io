# Flask_Restful

标签： Restful接口规范  参数验证  蓝图

---

### Restful接口规范介绍
- REST：Representational State Transfer，
- REST 指的是一组架构约束条件和原则。满足这些约束条件和原则的应用程序或设计就是 RESTful。
- 是一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件。
- 它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次。
- restful接口规范是用于在前端与后台进行通信的一套规范。使用这个规范可以让前后端开发变得更加轻松。

####适用场景：
一个系统的数据库数据，展现的平台有PC端、移动端、app端、ios端。
前端工程师：都遵循RESTful编程规范
后端工程师：都遵循RESTful编程规范
最终结果：开发效率高，便于管理

####协议：
用http或者https协议。

#### 数据传输格式：
数据传输的格式应该都用json格式。

#### url链接规则：
url链接中，不能有动词，只能有名词。并且对于一些名词，如果出现复数，那么应该在后面加s。比如：获取新闻列表，应该使用`/news/`，而不应该使用/get_news/

#### HTTP请求方式：
GET：从服务器上获取资源。
POST：在服务器上新增或者修改一个资源。
PUT：在服务器上更新资源。（客户端提供所有改变后的数据）
PATCH：在服务器上更新资源。（客户端只提供需要改变的属性）
DELETE：从服务器上删除资源。

#### 状态码：

- 200:	    
OK	            服务器成功响应客户端的请求。
- 400:
INVALID REQUEST	用户发出的请求有错误，服务器没有进行新建或修改数据的操作
- 401:
Unauthorized	用户没有权限访问这个请求
- 403:
Forbidden	    因为某些原因禁止访问这个请求
- 404:
NOT FOUND	    用户请求的url不存在
- 406:
NOT Acceptable  用户请求不被服务器接收（服务器期望客户端发送某个字段，但是没有发送）。
- 500:
Internal server error	服务器内部错误，比如遇到bug



### Flask_RESTful功能之参数验证：

#### 参数验证：也叫参数解析
Flask-Restful插件提供了类似WTForms来验证提交的数据是否合法的包，叫做reqparse。

#### 基本用法：(借助于测试工程师 常用的接口测试工具postman来检验)

```
from flask import Flask,url_for,render_template
from flask_restful import Api,Resource,reqparse,inputs
app = Flask(__name__)
api = Api(app)

class RegisterView(Resource):
    def post(self):
        #验证用户名
        #1.创建解析器对象
        parser = reqparse.RequestParser()
        #2.利用解析器对象添加 需要验证的参数
        parser.add_argument('uname',type=str,help='用户名验证错误！',required=True,trim=True)
                #3.利用解析器对象进行验证，若正确，直接返回验证后合格的参数值，若错误，抛异常信息给客户端
        args = parser.parse_args()
        print(args)
        return {"tips":"注册成功"}

api.add_resource(RegisterView,'/register/')

@app.route('/')
def hello_world():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(debug=True)
```

#### 其中add_argument方法使用详解

add_argument方法可以指定这个字段的名字，这个字段的数据类型等，验证错误提示信息等，具体如下：

- default：默认值，如果这个参数没有值，那么将使用这个参数指定的默认值。
- required：是否必须。默认为False，如果设置为True，那么这个参数就必须提交上来。
- type：这个参数的数据类型，如果指定，那么将使用指定的数据类型来强制转换提交上来的值。
- choices：固定选项。提交上来的值只有满足这个选项中的值才符合验证通过，否则验证不通过。
- help：错误信息。如果验证失败后，将会使用这个参数指定的值作为错误信息。
- trim：是否要去掉前后的空格。

其中的type，可以使用python自带的一些数据类型(如str或者int)，也可以使用flask_restful.inputs下的一些特定的数据类型来强制转换。比如一些常用的：
url：会判断这个参数的值是否是一个url，如果不是，那么就会抛出异常。
regex：正则表达式。
date：将这个字符串转换为datetime.date数据类型。如果转换不成功，则会抛出一个异常。

代码案例：
```
from flask import Flask,url_for,render_template
from flask_restful import Api,Resource,reqparse,inputs
app = Flask(__name__)
api = Api(app)

#Flask_RESTful功能之参数验证
class RegisterView(Resource):
    def post(self):
        #用户名   密码   年龄 性别  出生日期   号码  个人主页
        # 1.创建解析器对象
        parser = reqparse.RequestParser()
        #2.利用解析器对象添加 需要验证的参数
        parser.add_argument('uname',type=str,help='用户名验证错误！',required=True,trim=True)
        parser.add_argument('pwd', type=str, help='密码验证错误！',default="123456")
        parser.add_argument('age',type=int,help='年龄验证错误！')
        parser.add_argument('gender',type=str,choices=['男','女','双性'])
        parser.add_argument('birthday',type=inputs.date,help='生日字段验证错误！')
        parser.add_argument('phone',type=inputs.regex(r'1[3578]\d{9}'))
        parser.add_argument('phomepage',type=inputs.url,help='个人中心链接验证错误！')
        #3.利用解析器对象进行验证，若正确，直接返回验证后合格的参数值，若错误，抛异常信息给客户端
        args = parser.parse_args()
        print(args)
        return {"tips":"注册成功"}

api.add_resource(RegisterView,'/register/')

@app.route('/')
def hello_world():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(debug=True)
```


### Flask_RESTful返回标准化参数：

对于一个类视图，你可以指定好一些字段作标准化用于返回。以后使用ORM模型或者自定义模型的时候，他会自动的获取模型中的相应的字段，生成json格式数据，然后再返回给客户端。这需要导入flask_restful.marshal_with装饰器。还需要写一个字典变量，来指定需要返回的标准化字段，以及该字段的数据类型。在get方法中，返回自定义对象的时候，flask_restful会自动的读取对象模型上的所有属性。组装成一个符合标准化参数的json格式字符串返回给客户端。


代码演示：
```
from flask import Flask
from flask_restful import Api,Resource,fields,marshal_with
app = Flask(__name__)
api = Api(app)

#flask_restful返回标准化参数
class News(object):
    def __init__(self,title,content):
        self.title =title
        self.content =content

news = News('能力强的体现','能屈能伸')

class NewsView(Resource):
    resource_fields ={
        'title': fields.String,
        'content':fields.String
    }
    @marshal_with(resource_fields)
    def get(self):
        # restful规范中，要求，定义好了返回的参数个数 和 内容
        # return {'title':"世界太大",'content':"可钱包太小"}
        #好处1：体现规范化，即使content这个参数没有值，也应该返回，返回一个null回去
        # return {'title':"世界太大"}
        #好处2：体现规范化，还可以返回一个对象模型回去
        return news

api.add_resource(NewsView,'/news/')


if __name__ == '__main__':
    app.run(debug=True)
```


### Flask_RESTful返回标准化参数强化：

#### 重命名属性和默认值：
需求1：
有的时候你对外给出的属性名和模型内部的属性名不相同时，可使用attribute可以配置这种映射。比如想要返回模型对象user.username的值，但是在返回给外面的时候，想以uname返回去。

需求2：
在返回某些字段的时候，有时候可能没有值，但想给一个值用以提示，那么这时候可以在指定fields的时候使用default指定默认值

```
from flask import Flask,url_for,render_template
from flask_restful import Api,Resource,reqparse,inputs,fields,marshal_with
app = Flask(__name__)
api = Api(app)

#flask_restful返回标准化参数强化
class User():
    def __init__(self,username,age):
        self.username=username
        self.age=age
        self.signature=None

class UserView(Resource):
    resource_fields={
        #1.重命名属性
        'uname':fields.String(attribute='username'),
        'age':fields.Integer,
        #2.默认值
        'signature':fields.String(default='此人很懒，什么也没写')
    }

    @marshal_with(resource_fields)
    def get(self):
        user = User('莫莫',18)
        return user

api.add_resource(UserView,'/user/')

if __name__ == '__main__':
    app.run(debug=True)
```


#### 复杂的参数结构：
大型的互联网项目中，返回的数据格式，是比较复杂的结构。
如：小米商城，
https://www.mi.com/
https://a.huodong.mi.com/flashsale/getslideshow

总结，无非就是key对应的value又是一个json，或者key对应的是一个列表，列表中的每项都是json那么可以使用一些特殊的字段来实现。
如在一个字段中放置一个列表，那么可以使用`fields.List`，如在一个字段下面又是一个字典，那么可以使用`fields.Nested`。
```
from flask import Flask,url_for,render_template
from flask_restful import Api,Resource,reqparse,inputs,fields,marshal_with
app = Flask(__name__)
api = Api(app)

#flask_restful返回标准化参数强化
#复杂的参数结构
#实体间关系有   1：1   1:n    n:n(转为2个1:n)
#新闻系统后台  用户  新闻   新闻标签
class User():
    def __init__(self,id,uname,age):
        self.id = id
        self.uname = uname
        self.age = age
    def __repr__(self):
        return "<User:{id},{uname},{age}>".format(id=self.id, uname=self.uname,age=self.age)

class News():
    def __init__(self,id,title,content):
        self.id=id
        self.title=title
        self.content=content
        #关系映射
        self.author=None
        self.tags=[]
    def __repr__(self):
        return "<News:{id},{title},{content},{author},{tags}>"\
            .format(id=self.id, title=self.title,content=self.content,author=self.author,tags=self.tags)

class NewsTag():
    def __init__(self,id,name):
        self.id=id
        self.name=name
    def __repr__(self):
        return '<NewsTage:{id},{name}>'.format(id =self.id , name=self.name)

def createData():
    user = User(110,'莫莫',30)
    tag1 = NewsTag(200,"要闻")
    tag2 = NewsTag(210,"娱乐")
    news =News(300,'吴京征服了世界上海拔最高的派出所','4月23日中午11点，吴京发了一条微博，配文“世界上海拔最高的派出所”，并@了另外一名演员张译。微博中有两张图片，第一张是吴京和张译两人坐在地上的合照，背后几个大字“中国边防”。第二张则是两人与派出所民警们的合照。 ')
    news.author = user   #绑定新闻作者
    news.tags.append(tag1) #绑定新闻标签1
    news.tags.append(tag2) #绑定新闻标签2
    print(news)
    return news

class NewsView2(Resource):
    resource_fields={
        'id':fields.Integer,
        'title': fields.String,
        'content': fields.String,
        #如在一个字段下面又是一个字典，那么可以使用fields.Nested({...})
        'author': fields.Nested({
            'id': fields.Integer,
            'uname': fields.String,
            'age':fields.Integer
        }),
        #如要在一个字段中放置一个列表，那么可以使用fields.List(fields.Nested({...}))
        'tags': fields.List(fields.Nested({
            'id': fields.Integer,
            'name': fields.String
        }))
    }

    @marshal_with(resource_fields)
    def get(self):
        news = createData()
        return news

api.add_resource(NewsView2,'/news2/')


if __name__ == '__main__':
    app.run(debug=True)
```


### Flask_RESTful结合蓝图使用和渲染模版：

#### Flask_RESTful结合蓝图使用
在蓝图中，如果使用Flask_RESTful，那么在创建Api对象的时候，使用蓝图对象，不再是使用`app`对象了

蓝图news.py文件
```
from flask import url_for,render_template,Blueprint,make_response,Response
from flask_restful import Api,Resource,reqparse,inputs,fields,marshal_with
import json
news_bp = Blueprint('news',__name__,url_prefix='/news')
api = Api(news_bp)
#1.flask-restful结合蓝图使用

#复杂的参数结构
#实体间关系有   1：1   1:n    n:n(转为2个1:n)
#新闻系统后台  用户  新闻   新闻标签
class User():
    def __init__(self,id,uname,age):
        self.id = id
        self.uname = uname
        self.age = age
    def __repr__(self):
        return "<User:{id},{uname},{age}>".format(id=self.id, uname=self.uname,age=self.age)

class News():
    def __init__(self,id,title,content):
        self.id=id
        self.title=title
        self.content=content
        #关系映射
        self.author=None
        self.tags=[]
    def __repr__(self):
        return "<News:{id},{title},{content},{author},{tags}>"\
            .format(id=self.id, title=self.title,content=self.content,author=self.author,tags=self.tags)

class NewsTag():
    def __init__(self,id,name):
        self.id=id
        self.name=name
    def __repr__(self):
        return '<NewsTage:{id},{name}>'.format(id =self.id , name=self.name)

def createData():
    user = User(110,'莫莫',30)
    tag1 = NewsTag(200,"要闻")
    tag2 = NewsTag(210,"娱乐")
    news =News(300,'吴京征服了世界上海拔最高的派出所','4月23日中午11点，吴京发了一条微博，配文“世界上海拔最高的派出所”，并@了另外一名演员张译。微博中有两张图片，第一张是吴京和张译两人坐在地上的合照，背后几个大字“中国边防”。第二张则是两人与派出所民警们的合照。 ')
    news.author = user   #绑定新闻作者
    news.tags.append(tag1) #绑定新闻标签1
    news.tags.append(tag2) #绑定新闻标签2
    print(news)
    return news

class NewsView2(Resource):
    resource_fields={
        'id':fields.Integer,
        'title': fields.String,
        'content': fields.String,
        #如在一个字段下面又是一个字典，那么可以使用fields.Nested({...})
        'author': fields.Nested({
            'id': fields.Integer,
            'uname': fields.String,
            'age':fields.Integer
        }),
        #如要在一个字段中放置一个列表，那么可以使用fields.List(fields.Nested({...}))
        'tags': fields.List(fields.Nested({
            'id': fields.Integer,
            'name': fields.String
        }))
    }

    @marshal_with(resource_fields)
    def get(self):
        news = createData()
        return news

api.add_resource(NewsView2,'/news2/')
```

app.py文件
```
from flask import Flask,url_for,render_template

from blueprints.news import news_bp
app = Flask(__name__)
app.register_blueprint(news_bp)

@app.route('/')
def hello_world():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(debug=True)
```

#### Flask_RESTful渲染模版
如果在Flask_RESTful的类视图中想要返回html片段代码，或者是整个html文件代码，即渲染模版的意思。那么就应该使用`api.representation`这个装饰器来定义一个函数，在这个函数中，应该对`html`代码进行一个封装，再返回。
```
from flask import url_for,render_template,Blueprint,make_response,Response
from flask_restful import Api,Resource,reqparse,inputs,fields,marshal_with
import json
news_bp = Blueprint('news',__name__,url_prefix='/news')
api = Api(news_bp)

#2. 使用flask-restful渲染模版
class ListView(Resource):
    def get(self):
        return render_template('index.html')
api.add_resource(ListView,'/list/')

# 渲染模版经过修改后，能支持html和json
@api.representation('text/html')
def output_html(data,code,headers):
    if isinstance(data,str):
        # 在representation装饰的函数中，必须返回一个Response对象
        # resp = make_response(data)
        resp =Response(data)
        return resp
    else:
        return Response(json.dumps(data),mimetype='application/json')
```


