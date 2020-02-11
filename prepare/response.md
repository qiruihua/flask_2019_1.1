# 响应数据

## 关于接口通用返回格式

我们项目接口采用**RESTful**风格定义，接口通用返回数据格式如下：

```
{
    "message": "接口信息",
    "data": {/*实际返回的数据*/}    // 接口数据
}
```

## 单个蓝图设置

在home子应用的\_\__init\_\__.py\_设置返回响应的装饰器

```
from flask import make_response, current_app
from flask_restful.utils import PY3
from json import dumps

@home_api.representation('application/json')
def output_json(data, code, headers=None):
    """Makes a Flask response with a JSON encoded body"""

    # 此处为自己添加***************
    if 'message' not in data:
        data = {
            'message': 'OK',
            'data': data
        }
    # **************************

    settings = current_app.config.get('RESTFUL_JSON', {})

    # If we're in debug mode, and the indent is not set, we set it to a
    # reasonable value here.  Note that this won't override any existing value
    # that was set.  We also set the "sort_keys" value.
    if current_app.debug:
        settings.setdefault('indent', 4)
        settings.setdefault('sort_keys', not PY3)

    # always end the json dumps with a new line
    # see https://github.com/mitsuhiko/flask/pull/1262
    dumped = dumps(data, **settings) + "\n"

    resp = make_response(dumped, code)
    resp.headers.extend(headers or {})
    return resp
```

## 所有蓝图设置

### 创建common包

common包中主要存放一些抽取的公共的类，这些类在其他系统中也可以用到

![](/assets/project_common.png)

### 再在common包中继续创建toutiaoapi包，并创建restful.py文件

![](/assets/common_toutiaoapi.png)

```
from flask_restful import Api
from flask import make_response, current_app
from flask_restful.utils import PY3
from json import dumps


def output_json(data, code, headers=None):
    """Makes a Flask response with a JSON encoded body"""
    if 'message' not in data:
        data = {
            'message': 'OK',
            'data': data
        }

    settings = current_app.config.get('RESTFUL_JSON', {})

    # If we're in debug mode, and the indent is not set, we set it to a
    # reasonable value here.  Note that this won't override any existing value
    # that was set.  We also set the "sort_keys" value.
    if current_app.debug:
        settings.setdefault('indent', 4)
        settings.setdefault('sort_keys', not PY3)

    # always end the json dumps with a new line
    # see https://github.com/mitsuhiko/flask/pull/1262
    dumped = dumps(data, **settings) + "\n"

    resp = make_response(dumped, code)
    resp.headers.extend(headers or {})
    return resp

class BaseApi(Api):

    def __init__(self, app=None, prefix='',
                 default_mediatype='application/json', decorators=None,
                 catch_all_404s=False, serve_challenge_on_401=False,
                 url_part_order='bae', errors=None):
        super().__init__(app=app, prefix=prefix, default_mediatype=default_mediatype, decorators=decorators,
                         catch_all_404s=catch_all_404s, serve_challenge_on_401=serve_challenge_on_401,
                         url_part_order=url_part_order, errors=errors)
        self.representations = {'application/json': output_json}
```

### 我们可以进一步讲output\_json函数抽取到另外一个文件中

![](/assets/common_output.png)

output文件代码如下

```
from flask import make_response, current_app
from flask_restful.utils import PY3
from json import dumps


def output_json(data, code, headers=None):
    """Makes a Flask response with a JSON encoded body"""
    if 'message' not in data:
        data = {
            'message': 'OK',
            'data': data
        }

    settings = current_app.config.get('RESTFUL_JSON', {})

    # If we're in debug mode, and the indent is not set, we set it to a
    # reasonable value here.  Note that this won't override any existing value
    # that was set.  We also set the "sort_keys" value.
    if current_app.debug:
        settings.setdefault('indent', 4)
        settings.setdefault('sort_keys', not PY3)

    # always end the json dumps with a new line
    # see https://github.com/mitsuhiko/flask/pull/1262
    dumped = dumps(data, **settings) + "\n"

    resp = make_response(dumped, code)
    resp.headers.extend(headers or {})
    return resp
```

### 修改restful.py文件

```
from flask_restful import Api
from .output import output_json

class BaseApi(Api):

    def __init__(self, app=None, prefix='',
                 default_mediatype='application/json', decorators=None,
                 catch_all_404s=False, serve_challenge_on_401=False,
                 url_part_order='bae', errors=None):
        super().__init__(app=app, prefix=prefix, default_mediatype=default_mediatype, decorators=decorators,
                         catch_all_404s=catch_all_404s, serve_challenge_on_401=serve_challenge_on_401,
                         url_part_order=url_part_order, errors=errors)
        self.representations = {'application/json': output_json}
```

### 修改蓝图中的Api为BaseApi

在setting文件中添加common路径

```
import sys
sys.path.insert(0,os.path.join(BASE_DIR, 'common'))
```

修改蓝图api

```
from toutiaoapi.restful import BaseApi
#创建蓝图
home_blueprint=Blueprint('home',__name__,url_prefix='/')
#Api接管蓝图
home_api=BaseApi(home_blueprint)
```



