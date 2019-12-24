# 获取短信验证码

## 接口分析

**请求方式**：GET /app/v1\_0/sms/codes/&lt;mobile&gt;/

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| mobile | str | 是 | 手机号 |

**返回数据**： JSON

```
{
    "message": "ok",
    "data": {
        "mobile": "18310820688"
    }
}
```

| 返回值 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| mobile | dict | 是 | 手机号 |

## 后端实现

### 定义类视图

```
class SmsCodeResource(Resource):

    def get(self,mobile):
        
        #发送短信验证码
        
        return {
            'mobile':mobile
        }


user_api.add_resource(SmsCodeResource,'/sms/codes/<mobile>/')
```

### 设置返回响应的装饰器

```
from flask import make_response, current_app
from flask_restful.utils import PY3
from json import dumps

@user_api.representation('application/json')
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



