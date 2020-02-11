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

在home子应用的\_\__init\_\_.py_设置返回响应的装饰器

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




