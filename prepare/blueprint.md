# 蓝图

项目的蓝图模块可以按以下方式来分：

* 按功能模块来分，比如：用户模块、订单模块
* 按接口版本来分，某个版本的接口放一个文件夹下面

* 在整个项目文件夹中，除了启动文件`manage.py`和配置文件`setings.py`放在根目录，其他具体业务逻辑文件都放在一个单独的文件包内，与`manage.py`同级

* 创建`apps`Package，与`manage.py`同级

![](/assets/project.png)

> apps 存放当前项目所有的模块

## Home蓝图

在`apps`文件夹下创建`home`包， 并在此文件夹下创建`views.py`文件

![](/assets/home蓝图.png)

### \_\__init_\_\_.py代码

```
from flask import Blueprint
from flask_restful import Api
#创建蓝图
home_blueprint=Blueprint('home',__name__,url_prefix='/')
#Api接管蓝图
home_api=Api(home_blueprint)

#记住要导入过来
from . import views

home_api.add_resource(views.IndexResource, '/')
```

### view.py代码

```
from toutiao.apps.home import home_api
from flask_restful import Resource

class IndexResource(Resource):

    def get(self):
        return {
                "index": "index"
        }
```

### project包的\_\__init_\_.py中注册路由\_

```
# 注册蓝图
from project.apps.home import home_blueprint
app.register_blueprint(home_blueprint)
```

> 测试访问首页

## User蓝图

### \_\__init_\_\_.py代码

```
from flask import Blueprint
from flask_restful import Api
#创建蓝图
user_blueprint=Blueprint('user',__name__,url_prefix='/app/v1_0')
#Api接管蓝图
user_api=Api(user_blueprint)
#记住 导入
from . import views

user_api.add_resource(views.LoginResource,'/login')
```

### view.py代码

```
from flask_restful import Resource
from project.apps.user import user_api

class LoginResource(Resource):

    def get(self):
        return {
              "login": "login"
            }
```

### project包的\_\__init_\_.py中注册路由\_

```
# 注册蓝图

from project.apps.user import user_blueprint
app.register_blueprint(user_blueprint)
```

> 测试访问



