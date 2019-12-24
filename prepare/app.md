# 项目基本配置 {#项目基本配置}

## Config类 {#config类}

* 先在当前类中定义配置的类，并从中加载配置

```
app = Flask(__name__)

class Config(object):
    """工程配置信息"""
    DEBUG = True

    SERVER_NAME = '127.0.0.1:5000'

#加载配置文件
app.config.from_object(Config)
```

> 运行测试

## SQLAlchemy {#sqlalchemy}

* 导入数据库扩展，并在配置中填写相关配置

```
from flask_sqlalchemy import SQLAlchemy

...

class Config(object):
    """工程配置信息"""
    DEBUG = True
    # 数据库的配置信息
    SQLALCHEMY_DATABASE_URI = "mysql://root:mysql@127.0.0.1:3306/toutiao"
    SQLALCHEMY_TRACK_MODIFICATIONS = False

app.config.from_object(Config)
db = SQLAlchemy(app)
```

* 在终端创建数据库

```
mysql > create database toutiao charset utf8;
```

> 运行测试

## Redis {#redis}

* 创建redis存储对象，并在配置中填写相关配置

```
import redis
...

class Config(object):
    """工程配置信息"""
    ...
    # redis配置
    REDIS_HOST = "127.0.0.1"
    REDIS_PORT = 6379

app.config.from_object(Config)
db = SQLAlchemy(app)
redis_store = redis.StrictRedis(host=Config.REDIS_HOST, port=Config.REDIS_PORT)
```

> 运行测试

## Session {#session}

* 利用 flask-session扩展，将 session 数据保存到 Redis 中

```
from flask_session import Session
...

class Config(object):
    """工程配置信息"""
    SECRET_KEY = "EjpNVSNQTyGi1VvWECj9TvC/+kq3oujee2kTfQUs8yCM6xX9Yjq52v54g+HVoknA"
    ...
    # flask_session的配置信息
    SESSION_TYPE = "redis" # 指定 session 保存到 redis 中
    SESSION_USE_SIGNER = True # 让 cookie 中的 session_id 被加密签名处理
    SESSION_REDIS = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT) # 使用 redis 的实例
    PERMANENT_SESSION_LIFETIME = 86400 # session 的有效期，单位是秒

app.config.from_object(Config)
...
Session(app)
```

> 运行测试
>
> 文档地址：[http://pythonhosted.org/Flask-Session/](http://pythonhosted.org/Flask-Session/)



