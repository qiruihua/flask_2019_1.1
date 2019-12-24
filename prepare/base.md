# 代码抽取

目标：将特定逻辑代码抽取到指定的类中，各司其职，放便后续项目维护

## 配置文件 {#配置文件}

* 在与`manage.py`同级目录下创建`setting.py`文件，用作于项目的配置文件

```
import redis

class Config(object):
    """工程配置信息"""
    DEBUG = True

    # 数据库的配置信息
    SQLALCHEMY_DATABASE_URI = "mysql://root:mysql@127.0.0.1:3306/toutiao"
    SQLALCHEMY_TRACK_MODIFICATIONS = False

    # redis配置
    REDIS_HOST = "127.0.0.1"
    REDIS_PORT = 6379


    # flask_session的配置信息
    SECRET_KEY = "EjpNVSNQTyGi1VvWECj9TvC/+kq3oujee2kTfQUs8yCM6xX9Yjq52v54g+HVoknA"
    SESSION_TYPE = "redis"  # 指定 session 保存到 redis 中
    SESSION_USE_SIGNER = True  # 让 cookie 中的 session_id 被加密签名处理
    SESSION_REDIS = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT)  # 使用 redis 的实例
    PERMANENT_SESSION_LIFETIME = 86400  # session 的有效期，单位是秒
```

在`manager.py`中引入`Config`类，直接使用

```
from settings import Config

app = Flask(__name__)

# 配置
app.config.from_object(Config)
```

> 运行测试

## 业务逻辑独立

在整个项目文件夹中，除了启动文件`manage.py`和配置文件`setings.py`放在根目录，其他具体业务逻辑文件都放在一个单独的文件夹内，与`manage.py`同级

* 创建`apps`Package，与`manage.py`同级

![](/assets/apps.png)

* `manage.py`只做最基本的启动工作，将`app`的创建操作移动到`apps`的`__init__.py`文件中

```
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
import redis
from flask_session import Session
from settings import Config

app = Flask(__name__)

#加载配置文件
app.config.from_object(Config)
# 数据库
db = SQLAlchemy(app)
# redis设置
redis_store = redis.StrictRedis(host=Config.REDIS_HOST, port=Config.REDIS_PORT)
#session设置
Session(app)


@app.route('/')
def index():
    return 'index'

```

* `manage.py`的代码为

```
from apps import app,db
from flask_script import Manager
from flask_migrate import Migrate,MigrateCommand

#Manager
manager=Manager(app)
#数据模型管理
Migrate(app, db)
manager.add_command('db', MigrateCommand)


if __name__ == '__main__':
    manager.run()
```



