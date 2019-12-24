# 项目准备

## 创建虚拟环境

新建一个虚拟环境py3\_toutiao

```
mkvirtualenv -p python3 py3_toutiao
```

安装Flask

```
pip install Flask
pip install flask-sqlalchemy
pip install redis
pip install flask-session
pip install flask-script
pip install flask_migrate
```

## 创建项目

* 新建项目（例如 toutiao），创建`manage.py`文件

```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return 'index'

if __name__ == '__main__':
    app.run()
```

![](/assets/创建项目.png)

