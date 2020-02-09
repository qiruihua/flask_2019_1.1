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
| mobile | str | 是 | 手机号 |

## 后端实现

### 定义类视图

```
from flask import current_app
import random
from string import digits
from project.apps.user import constants

class SmsCodeResource(Resource):

    def get(self,mobile):

        #发送短信验证码
        code = ''.join([random.choice(digits) for _ in range(6)])
        current_app.redis_store.setex('app:code:{}'.format(mobile), constants.SMS_VERIFICATION_CODE_EXPIRES, code)

        #返回相应
        return {
            'mobile':mobile
        }


user_api.add_resource(SmsCodeResource,'/sms/codes/<mobile>/')
```

将系统常量提取出来集中管理\(**/user/constants.py**\)：

```
# 短信验证码有效期, 秒

SMS_VERIFICATION_CODE_EXPIRES = 5 * 60
```

### 定义路由 {#23-定义路由}

我们从路由中传入手机号，为此我们先定义一个路由转换器**MobileConverter**\(**utils/converters.py**\)

![](/assets/project_utils.png)

```
from werkzeug.routing import BaseConverter


class MobileConverter(BaseConverter):
    """
    手机号格式
    """
    regex = r'1[3-9]\d{9}'


def register_converters(app):
    """
    向Flask app中注册转换器
    :param app: Flask app对象
    :return:
    """
    app.url_map.converters['mobile'] = MobileConverter
```

在`project`的`__init__.py`文件中注册转换器

```
# 注册url转换器
from project.utils.converters import register_converters
register_converters(app)
```

> **注意**：在注册蓝图前添加转换器

修改路由

```
user_api.add_resource(SmsCodeResource,'/sms/codes/<mobile:mobile>/')
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



