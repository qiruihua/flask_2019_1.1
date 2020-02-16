# 代码抽取

目标：将特定逻辑代码抽取到指定的类中，各司其职，放便后续项目维护

## 配置文件 {#配置文件}

![](/assets/setting文件.png)

* 在与`manage.py`同级目录下创建`settings.py`文件，用作于项目的配置文件

```
class Config(object):
    """工程配置信息"""
    DEBUG = True

    # 数据库的配置信息
    SQLALCHEMY_DATABASE_URI = "mysql://root:mysql@127.0.0.1:3306/toutiao"
    SQLALCHEMY_TRACK_MODIFICATIONS = False

    # redis配置
    REDIS_HOST = "127.0.0.1"
    REDIS_PORT = 6379
```

在`manage.py`中引入`Config`类，直接使用

```
from toutiao.settings import Config

app = Flask(__name__)

# 配置
app.config.from_object(Config)
```

> 运行测试

## manage脚本管理

在toutiao项目文件夹中，除了启动文件`manage.py`主要实现脚本管理，`setings.py`主要是配置文件，flask，db等实例创建可以抽取到\_\__init_\_.py文件中

### \_\__init_\_\_.py文件代码

```
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
import redis
from flask_cors import CORS
from toutiao.settings import Config


app = Flask(__name__)


#加载配置文件
app.config.from_object(Config)
#创建db
db = SQLAlchemy(app)
#创建redis客户端
app.redis_store = redis.StrictRedis(host=Config.REDIS_HOST, port=Config.REDIS_PORT)
#添加CORS
CORS(app)

@app.route('/')
def index():
    return 'index'
```

### `manage.py`文件代码

```
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand
from toutiao import app,db

# 把 Manager 类和应用程序实例进行关联
manager = Manager(app)
#使用脚本管理模型迁移
Migrate(app, db)
manager.add_command('db', MigrateCommand)

if __name__ == '__main__':
    manager.run()
```

> 测试运行

## 项目多种配置

[文档连接](http://docs.jinkan.org/docs/flask/config.html#id7)

一个web程序在开发阶段可能与生产阶段所需要的配置信息可能不一样，所以为了实现此功能，可以给不同情况创建不同的配置类，比如开发阶段使用的配置类名为`DevelopementConfig`，生产阶段使用的配置类名为`ProdutionConfig`

* 修改`settings.py`文件的配置文件如下

```
class Config(object):
    ...

# 线上/生成
class ProductionConfig(Config):
    DEBUG = False
# 线下/开发
class DevelopmentConfig(Config):
    DEBUG = True
# 测试环境
class TestingConfig(Config):
    TESTING = True

config_dict = {
    'prod':ProductionConfig,
    'dev':DevelopmentConfig,
    'test':TestingConfig
}
```

`toutiao`的`__init__.py`文件修改后的代码

```
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
import redis
from settings import config_dict

app = Flask(__name__)
#获取配置类
Config=config_dict['dev']
#加载配置文件
app.config.from_object(Config)
db = SQLAlchemy(app)
app.redis_store = redis.StrictRedis(host=Config.REDIS_HOST, port=Config.REDIS_PORT)

@app.route('/')
def index():
    return 'index'
```



