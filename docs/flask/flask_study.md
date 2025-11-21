# 基本web框架的学习

## 1.1 flask工程文件结构

一个中等体量的flask代码文件夹应该包括

<pre>
my_flask_app/
├── app/
│   ├── __init__.py
│   ├── routes.py
│   └── models.py
├── config.py
├── requirements.txt
└── run.py
</pre>

- app/：包含 Flask 应用的主要代码。
- \___init\__.py：初始化 Flask 应用和配置扩展。
- routes.py：定义应用的路由和视图函数。
- models.py：定义应用的数据模型。
- config.py：配置文件，包含应用的配置信息。
- requirements.txt：列出项目的依赖库。
- run.py：用于启动 Flask 应用。

运用到了“应用工厂”这样一个函数，即create_app,

```py
#__init__.py
from flask import Flask

def create_app():
    app = Flask(__name__)

    from . import routes
    app.register_blueprint(routes.bp)

    return app

```
```py
# routes.py
from flask import Blueprint, request

bp = Blueprint('main',__name__)

@bp.route('/')
def home():
    return "hello world"
```

```py
# run.py
from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True)
```


## 1.2 路由
Flask 路由是 Web 应用程序中将 URL 映射到 Python 函数的机制。

就是@app.route('/')

#### 路由参数 ：
以get方式请求，在url体现，但是可以传递给视图函数的参数

```py
@app.route('/greet/<name>')
def greet(name):
    return f'Hello, {name}!'
```

当然可以添加如有规则，只需要在路由参数括号里，名称前面加上类型限制，包括：

int:整数

float：浮点数

path：路径

<b>如果提交的不符合要求，那么就会爆404错误</b>

当然可以在后续的视图函数里面设置一个默认的参数，比如None、default_name 之类的，不过还有一种高级一点的方法：

```py
@bp.route('/user',defaults={'name': 'hajimi'})
```

#### 请求方式
限制请求方式，GET POST等

```py
@app.route('/submit', methods=['POST'])
def submit():
    return 'Form submitted!'
```

使用非法的请求方法就会爆出 405 （Method Not Allowed）错误

#### 路由函数返回
字符串/json/html/响应

####  静态文件和模板
模板文件则通过 templates 文件夹组织，用于渲染 HTML 页面。
templates文件夹需要放在app.py相同目录

```py
from flask import render_template

@app.route('/hello/<name>')
def hello(name):
    return render_template('hello.html', name=name)
```

这里成功把hello.html的{{name}}参数替换了


## 1.3 视图函数

前面提到，视图函数可以接受路由提供的参数，也可以通过POST的方式接受参数
```py
@app.route('/submit',method=['POST'])

def submit():
    username = request.form.get('username')

```

request.form.get('username')：获取 POST 请求中表单数据的 username 字段。

也可以使用request.args.get进行参数查询，比如一个格式这样的查询参数：

/query?query=

也可以设置default参数和默认type
```py
@bp.route('/search')
def search():
    query = request.args.get('query',default="hello",type=str)
    return f'Search results for: {query}'
```

#### 处理请求头信息

```py
@bp.route('/info')

def info():
    user_agent = request.headers.get('User-Agent')
    return f'Your User-Agent is: {user_agent}'

```

#### 创造自定义相应

使用make_response('') , 括号内为响应头信息

#### 处理错误

```py
@bp.route('/calculate/<int:num1>/<int:num2>')
def calculate(num1, num2):
    try:
        result = num1 / num2
        return f'The result of {num1} divided by {num2} is {result}'
    except ZeroDivisionError:
        return 'Error: Division by zero is not allowed.', 400
```

哇还有自定义状态码

还可以处理全局错误：

```py
    @app.errorhandler(404)
    def not_found(error):
        return render_template('404.html'), 404

```
要注意的点：这个属于全局错误，不能放在蓝图，即routes.py里面，因为错误的url并不能匹配任何路由，因此根本不会进入routes.py

所以这段代码必须放到主程序里面，即app.py

## 2.1 Flask 模板渲染

#### 模板继承
```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Website{% endblock %}</title>
</head>
<body>
    <header>
        <h1>My Website</h1>
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer>
        <p>Footer content</p>
    </footer>
</body>
</html>

```
把它设置为基础模板base.html
后续的html文件只需要基于这个模板进行替换这里的两个块，即{% block title %}My Website{% endblock %} 和 {% block content %}{% endblock %}

替换的格式：

```html

{% extends "base.html" %} 
<!--继承基础模板-->

{% block title %}Home Page{% endblock %}

{% block content %}
<h2>Welcome to the Home Page!</h2>
<p>Content goes here.</p>
{% endblock %}
```

## 表单处理

最基础的POST方式表单处理方法：
```py
@bp.route('/form')
def form():
    return render_template('form.html')

@bp.route('/submit', methods=['POST'])
def submit():
    name = request.form.get('name')
    email = request.form.get('email')
    return f"name: {name}, email: {email}"

```

这里浏览器居然会自动提示email需要@符号，好神奇

#### WTF

#### 文件上传
