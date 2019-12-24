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



