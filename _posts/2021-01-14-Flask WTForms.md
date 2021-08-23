# Flask WTForms

标签： flask WTForms

---

### WTForms介绍和基本使用：
#### WTForms介绍：
这个插件库主要有两个作用。
第一个是做表单验证，将用户提交上来的数据进行验证是否符合系统要求。
第二个是做模版渲染。 （了解即可）
官网：https://wtforms.readthedocs.io/en/latest/index.html

Flask-WTF是简化了WTForms操作的一个第三方库。WTForms表单的两个主要功能是验证用户提交数据的合法性以及渲染模板。而Flask-WTF还包括一些其他的功能：CSRF保护，文件上传等。
安装Flask-WTF默认也会安装WTForms，因此使用以下命令来安装Flask-WTF和WTForms:
```
pip install flask-wtf
```

####WTForms做表单验证的基本使用：
- 自定义一个表单类，继承自wtforms.Form类。
- 定义好需要验证的字段，字段的名字必须和模版中那些需要验证的input标签的name属性值保持一致。
- 在需要验证的字段上，需要指定好具体的数据类型。
- 在相关的字段上，指定验证器。
- 以后在视图函数中，只需要使用这个表单类的对象，并且把需要验证的数据，也就是request.form传给这个表单类，再调用表单类对象.validate()方法进行，如果返回True，那么代表用户输入的数据都是符合格式要求的，Flase则代表用户输入的数据是有问题的。如果验证失败了，那么可以通过表单类对象.errors来获取具体的错误信息。

如注册页面register.html：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>某系统注册页面</title>
</head>
<body>
<form action="/register/" method="post">
    <table>
            <tr>
                <th>用户名：</th>
                <td><input type="text" name="uname"></td>
            </tr>
            <tr>
                <th>密码：</th>
                <td><input type="password" name="pwd"></td>
            </tr>
            <tr>
                <th>确认密码：</th>
                <td><input type="password" name="pwd2"></td>
            </tr>
            <tr>
                <td></td>
                <td><input type="submit" value="注册"></td>
            </tr>
    </table>
</form>
</body>
</html>
```
如app.py文件：
```
from flask import Flask,request,render_template
from wtforms import Form,StringField
from wtforms.validators import Length,EqualTo
app = Flask(__name__)

class RegisterForm(Form):
    uname =StringField(validators=[Length(min=2,max=15,message='用户名长度必须在2-15之间')])
    pwd = StringField(validators=[Length(min=6,max=12)])
    pwd2 = StringField(validators=[Length(min=6,max=12),EqualTo("pwd")])

@app.route('/')
def hello_world():
    return render_template("register.html")

@app.route('/register/',methods=['GET','POST'])
def register():
    if request.method == 'GET':
        return render_template('register.html')
    else:
        form = RegisterForm(request.form)
        if form.validate(): #验证   要么ok  要么no
            return "验证通过"
        else:
            print(form.errors)
            return "数据验证通不过"


if __name__ == '__main__':
    app.run(debug=True)
```



### WTForms常用验证器：
页面把数据提交上来，需要经过表单验证，进而需要借助验证器来进行验证，以下是常用的内置验证器：

1. Length：字符串长度限制，有min和max两个值进行限制。
2. EqualTo：验证数据是否和另外一个字段相等，常用的就是密码和确认密码两个字段是否相等。
3. Email：验证上传的数据是否为邮箱数据格式  如：223333@qq.com。
4. InputRequired：验证该项数据为必填项，即要求该项非空。
5. NumberRange：数值的区间，有min和max两个值限制，如果处在这两个数字之间则满足。
6. Regexp：定义正则表达式进行验证，如验证手机号码。
7. URL：必须是URL的形式 如http://www.bjsxt.com。
8. UUID：验证数据是UUID类型。

注意：
  数据项的类型，一般常用的有
```
from wtforms import Form,StringField,IntegerField,RadioField,BooleanField,SelectField,TextAreaField,DateField
class RegisterForm2(Form):
    uname = StringField(validators=[InputRequired()])
    age = IntegerField(validators=[NumberRange(18,40)])
```

代码演示：
register2.html文件
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册页面2</title>
</head>
<body>
  <form action="/register2/" method="post">
    <table>
            <tr>
                <th>邮箱：</th>
                <td><input type="email" name="email"></td>
            </tr>
            <tr>
                <th>用户名：</th>
                <td><input type="text" name="uname"></td>
            </tr>
            <tr>
                <th>年龄：</th>
                <td><input type="number" name="age"></td>
            </tr>
            <tr>
                <th>手机号码：</th>
                <td><input type="text" name="phone"></td>
            </tr>
            <tr>
                <th>个人主页：</th>
                <td><input type="text" name="phomepage"></td>
            </tr>
            <tr>
                <th>uuid：</th>
                <td><input type="text" name="uuid"></td>
            </tr>
            <tr>
                <th></th>
                <td><input type="submit" value="注册"></td>
            </tr>
    </table>
</form>
</body>
</html>
```

formscheck.py表单验证工具类文件
```
from wtforms import Form,StringField,IntegerField
from wtforms.validators import Length,EqualTo,Email,InputRequired,NumberRange,Regexp,URL,UUID

class RegisterForm(Form):
    uname =StringField(validators=[Length(min=2,max=15,message='用户名长度必须在2-15之间')])
    pwd = StringField(validators=[Length(min=6,max=12)])
    pwd2 = StringField(validators=[Length(min=6,max=12),EqualTo("pwd")])


class RegisterForm2(Form):
    email = StringField(validators=[Email()])
    uname = StringField(validators=[InputRequired()])
    age = IntegerField(validators=[NumberRange(18,40)])
    phone = StringField(validators=[Regexp(r'1[34578]\d{9}')])
    phomepage = StringField(validators=[URL()])
    uuid = StringField(validators=[UUID()])
```

app.py文件
```
from flask import Flask,request,render_template
from formscheck import  RegisterForm,RegisterForm2
app = Flask(__name__)

@app.route('/')
def hello_world():
    return render_template("register.html")

#基本使用
@app.route('/register/',methods=['GET','POST'])
def register():
    if request.method == 'GET':
        return render_template('register.html')
    else:
        form = RegisterForm(request.form)
        if form.validate(): #验证   要么ok  要么no
            return "验证通过"
        else:
            print(form.errors)
            return "数据验证通不过"

#常用验证器使用
@app.route('/register2/',methods=['GET','POST'])
def login():
    if request.method == 'GET':
        return render_template("register2.html")
    else:
        form = RegisterForm2(request.form)
        if form.validate():
            return "验证OK"
        else:
            print(form.errors)
            return "验证失败"
import uuid
print(uuid.uuid4())#edbfa1d6-28f2-4f40-9111-528141a8e77d

if __name__ == '__main__':
    app.run(debug=True)
```



### WTForms自定义验证器：
只有当WTForms内置的验证器不够使的时候，才需要使用自定义验证器。
如果想要对表单中的某个字段进行更细化的验证，那么可以针对这个字段进行单独的验证。

自定义验证器步骤如下：
1. 定义一个方法，方法的名字规则是：`validate_字段名(self,field)`。
2. 在方法中，使用`field.data`可以获取到这个字段的具体的值。
3. 验证时，如果数据满足条件，那么可以什么都不做。如果验证失败，那么应该抛出一个`wtforms.validators.ValidationError`的异常，并且把验证失败的信息传到这个异常类中。

场景：验证码实现
关键代码演示：(实现验证码 验证)
```
from  flask import session
from wtforms import Form,StringField,IntegerField
from wtforms.validators import Length,EqualTo,Email,InputRequired,NumberRange,Regexp,URL,UUID,ValidationError

class RegisterForm2(Form):
    email = StringField(validators=[Email()])
    uname = StringField(validators=[InputRequired()])
    age = IntegerField(validators=[NumberRange(18,40)])
    phone = StringField(validators=[Regexp(r'1[34578]\d{9}')])
    phomepage = StringField(validators=[URL()])
    uuid = StringField(validators=[UUID()])
    code = StringField(validators=[Length(4,4)])
    #取到的值 和服务器上 session上存储的值对比
    def validate_code(self,field):
        print(field.data,session.get('code'))
        if field.data != session.get('code'):
            raise ValidationError('验证码不一致！')
```


###WTForms渲染模版：
渲染模版是WTForms的第二个作用，不过，我们只需要了解即可，不需要花太多精力和
除非那一天你能达到真正的全栈，才有可能考虑使用它。

代码：
formscheck.py文件
```
from wtforms import Form,StringField,IntegerField,BooleanField,SelectField,DateField
from wtforms.validators import Length,EqualTo,Email,InputRequired,NumberRange,Regexp,URL,UUID,ValidationError

 #渲染模版
class CreateForm(Form):
    uname = StringField("用户名：",validators = [InputRequired()])
    age = IntegerField("年龄：",validators = [NumberRange(18,40)])
    remember = BooleanField("记住我：")
    addr = SelectField('地址：',choices=[('bj',"北京"),('sj','上海'),('tj','天津')])
```

app.py文件
```
from flask import Flask,render_template
from formscheck import CreateForm

#WTForms渲染模版  了解
@app.route('/createform/')
def createform():
        form = CreateForm()
        return render_template('create_form.html',form=form)


if __name__ == '__main__':
    app.run(debug=True)
```

 create_form.html文件
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
        <style>
        .hot{
            background: red;
        }
    </style>
</head>
<body>
     <h2>WTForms创建表单</h2>
     <form action="#" method="post">
        <table>
                <tr>
                    <td>{{ form.uname.label }}</td>
                    <td>{{ form.uname(class='hot') }}</td>
                </tr>
                <tr>
                    <td>{{ form.age.label }}</td>
                    <td>{{ form.age() }}</td>
                </tr>
                <tr>
                    <td>{{ form.remember.label }}</td>
                    <td>{{ form.remember() }}</td>
                </tr>
                <tr>
                    <td>{{ form.addr.label }}</td>
                    <td>{{ form.addr() }}</td>
                </tr>
                <tr>
                    <td></td>
                    <td><input type="submit" value="提交"></td>
                </tr>
        </table>
     </form>

</body>
</html>
```

代码结果：
![WTForms创建表单](WTForms.png)

若感兴趣，可看官网了解更多：
https://wtforms.readthedocs.io/en/latest/index.html
https://wtforms.readthedocs.io/en/latest/fields.html
https://wtforms.readthedocs.io/en/latest/fields.html#field-definitions
