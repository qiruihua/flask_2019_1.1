# 项目准备

## 创建虚拟环境

新建一个虚拟环境py3\_toutiao

```
mkvirtualenv -p python3 py3_toutiao
```

安装依赖包

```
pip install -r requirements.txt
```

## 创建工程

新建目录（例如 toutiaoproject）

![](/assets/toutiaoproject.png)

## 创建头条目录

* 新建目录（例如 toutiao），并创建`manage.py`文件

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

