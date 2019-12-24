# 蓝图

项目的蓝图模块可以按以下方式来分：

* 按功能模块来分，比如：用户模块、订单模块
* 按接口版本来分，某个版本的接口放一个文件夹下面

* 在`project`目录下创建`apps`Package，创建完成如下：

![](/assets/子应用.png)

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
```

### view.py代码

```
from project.apps.home import home_api
from flask_restful import Resource

class IndexResource(Resource):

    def get(self):
        return {
                "message": "OK"
        }


home_api.add_resource(IndexResource, '/')
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
home_blueprint=Blueprint('home',__name__,url_prefix='/')
#Api接管蓝图
home_api=Api(home_blueprint)

#记住要导入过来
from . import views
```

### view.py代码

```
from project.apps.home import home_api
from flask_restful import Resource

class IndexResource(Resource):

    def get(self):
        return {
                "message": "OK"
        }


home_api.add_resource(IndexResource, '/')
```

### project包的\_\__init_\_.py中注册路由\_

```
# 注册蓝图
from project.apps.home import home_blueprint
app.register_blueprint(home_blueprint)
```

> 测试访问



