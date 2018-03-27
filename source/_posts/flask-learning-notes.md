---
title: Flask基础简介
date: 2016-04-07 16:51:01
tags: [flask, route]
---

![flask](http://docs.jinkan.org/docs/flask/_images/logo-full.png)

<!-- more -->

### 1.Flask Intor

```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```

### 2.Redirect

```
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))
    
@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
```

### 3.Session

```
from flask import Flask, session, redirect, url_for, escape, request

app = Flask(__name__)

@app.route('/')
def index():
    if'username'in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'
    
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form action="" method="post">
        <p><input type=text name=username>
        <p><input type=submit value=Login>
        </form>
        '''
        
@app.route('/logout')
def logout():
    # 如果用户名存在，则从会话中移除该用户名
    session.pop('username', None)
    return redirect(url_for('index'))

# 设置密钥，保证会话安全
app.secret_key = '0Zr98j/3yX R~XHH!jmN]LWX/,?R'
```

### 4.Router

#### (1) route装饰器

可以使用Flask应用实例的route装饰器将一个URL规则绑定到 一个视图函数上。

例如，下面的示例将URL规则/test绑定到视图函数test()上：

```
@app.route('/test')
def test():
    return'this is response'
```

在Flask中，转换器/converter用来对从URL中提取的变量进行预处理，这个过程 发生在调用视图函数之前。

![converter](https://raw.githubusercontent.com/Keleir/IMGDB/master/Blog/flask_convert.png)

Flask预置了四种转换器：

- string - 匹配不包含/的字符串，这是默认的转换器

- path - 匹配包含/的字符串

- int - 只有当URL中的变量是整型值时才匹配，并将变量转换为整型

- float - 只有当URL中的变量是浮点值时才匹配，并将变量转换为浮点型

#### (2) add_url_rule() 

另一种等价的写法是使用Flask应用实例的add_url_rule()方法。 
下面的示例注册了一个与前例相同的路由：

```
def test():
    return 'this is response'
    
app.add_url_rule('/test',view_func=test)
```

为路由指定请求方法：

```
@app.route('/user',methods=['POST','GET'])
def v_users():
    if request.method == 'GET':
        return ... # 返回用户列表
    if request.method == 'POST'
        return ... #创建新用户
```

##### Litte Demo:

```
# -*- coding:utf-8-*-
from flask import Flask

app = Flask(__name__)

@app.route('/')
def v_index():
    return '''
        <form action="/auth" method="POST">
        <input type="text" name="uid">
        <input type="password" name="pwd">
        <input type="submit" value="submit">
        </form>
    '''
    
@app.route('/auth',methods=['GET', 'POST'])
def v_auth():
    if request.method == 'GET':
        return 'GET REQ'
    if request.method == 'POST':
        return 'POST REQ'
        
app.run(host='0.0.0.0',port=80)
```

#### (3) 访问点/endpoint

在 Flask框架中，请求任务的分发并不是直接从用户请求的URL一步定位到视图函数， 两者之间隔着一个访问点/endpoint。

![endpoint](https://github.com/Keleir/IMGDB/blob/master/Blog/flask_endpoint.png?raw=true)

在Flask内部使用两张表维护路由：

- url_map ：维护URL规则和endpoint的映射
 
- view_functions ：维护endpoint和视图函数的映射

以用户访问URL/home为例，Flask将首先利用url_map找到所请求URL对应的 endpoint，即访问点home，然后再利用view_functions表查找home这个访问点 对应的视图函数，最终匹配到函数home()：

```
@app.route('/home')
def home():
    pass
```

#### (4)静态目录路由

当创建应用实例时，Flask将自动添加一条静态目录路由，其访问点 始终被设置为static，URL规则默认被设置为/static，本地路径默认被 设置为应用文件夹下的static子文件夹;

##### 改变默认的本地路径：

可以在创建应用对象时使用关键字参数 `static_folder` 改变 默认的静态文件夹。

例如，你的静态文件都存放在应用下的assets目录下， 那么可以按如下的方式创建应用对象：
    
```
app = Flask(__name__, static_folder='assets')
```

也可以使用一个绝对路径：
    
```
app =Flask(__name__,static_folder='/var/www/static')
```

改变默认的本地路径并不会对路由表产生影响。

##### 改变默认的URL规则 ： 

如果不喜欢静态目录URL/static，也可以在创建应用 对象时使用关键字参数 `static_url_path` 换一个别的名字。

下面的示例中，将应用下的assets文件夹注册为静态目录/assets：
  
```
app =Flask(__name__,static_folder='assets',static_url_path='/assets')
```

#### (5) url_for()

##### 添加URL变量： 
如果指定访问点对应的视图函数接收参数，那么关键字参数将生成对应的参数URL。

下面的示例将生成 /contact/Julia?format=html：

```
@app.route('/')
def v_index():
    print url_for('v_contact',name='Julia',format='html')
    return ''

@app.route('/contact/<name>')
def v_contact(name):
    pass
</name>
```

##### 添加锚点 ：
使用_anchor关键字可以为生成的URL添加锚点。

下面的示例将生成URL /contact#part2

```
@app.route('/')
def v_index():
    print url_for('v_contacts',_anchor='part2')
    
@app.route('/contact')
def v_contacts():
    pass
```

##### 外部URL： 
默认情况下，url_for()生成站内URL，可以设置关键字参数  `_external` 为True，生成包含站点地址的外部URL。

下面的示例将生成URL http://<x.y.z>/contacts:

```
@app.route('/')
def v_index():
    print url_for('v_contacts', _external=True)
    
@app.route('/contact')
def v_contacts():
    pass
```

### 5.Request & request

- form - 记录请求中的表单数据。类型：MultiDict
- args - 记录请求中的查询参数。类型：MultiDict
- cookies - 记录请求中的cookie。类型：Dict
- headers - 记录请求中的报文头。类型：EnvironHeaders
- method - 记录请求使用的HTTP方法：GET/POST/PUT....。类型：string
- environ - 记录WSGI服务器转发的环境变量。类型：Dict
- url - 记录请求的URL地址。类型：string

### 6.Global Varible

#### (1) g.user

在login view方法中我们通过检查 `g.user` 来判断一个用户是否登录了,
为了实现这个我们将使用Flask提供的 `before_request` 事件。

任何一个被`before_request`装饰器装饰的方法将会在每次request请求被收到时提前与view方法执行。
所以在这儿来设置我们的g.user变量(app/views.py)：

```
@app.before_request
def before_request():
    g.user = current_user
```

这就是它要做的一切，`current_user` 全局变量是被 `Flask-Login` 设定的，
所以我们只需要把它拷贝到更容易被访问的 `g` 变量就OK了。

这样，所有的请求都能访问这个登录的用户，甚至于内部的模板。


### References

- [欢迎进入Flask大型教程项目](http://www.pythondoc.com/flask-mega-tutorial/index.html)

- [Flask学习之用户登录](http://www.tuicool.com/articles/bEJNjai)
