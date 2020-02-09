# 登录注册

## 接口分析

**请求方式**：POST /app/v1\_0/authorizations

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| mobile | str | 是 | 手机号 |
| code | str | 是 | 验证码 |

**返回数据**： JSON

```
{
  "message": "ok",
  "data": {
    "token": "xxxxxxxx"
  }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| token | str | 是 | 用户token |

## 后端实现

定义验证手机号方法

```
import re

#验证手机号规格
def check_mobile(mobile_str):
    """
    检验手机号格式
    :param mobile_str: str 被检验字符串
    :return: mobile_str
    """
    if re.match(r'^1[3-9]\d{9}$', mobile_str):
        return mobile_str
    else:
        raise ValueError('{} is not a valid mobile'.format(mobile_str))
```

定义生成token的方法

```
from itsdangerous import TimedJSONWebSignatureSerializer
from settings import Config
#生成token
def get_user_login_token(user_id):

    serializer=TimedJSONWebSignatureSerializer(secret_key=Config.SECRET_KEY,expires_in=3600)

    data = {
        'user_id':user_id
    }

    return serializer.dumps(data).decode()
```

登录视图实现

```
from flask import request
from flask_restful import reqparse
from models.user import User
from project import db
from project.apps.user.utils import get_user_login_token
from project.utils.parsers import check_mobile

class LoginResource(Resource):

    def post(self):

        """
        1.接收数据
        2.验证数据
        3.根据手机号进行数据库查询，如果不存在则保存数据
        4.生成token
        5.返回相应
        """
        # 1.接收数据
        data = request.json
        # 2.验证数据
        parser = reqparse.RequestParser()
        parser.add_argument('mobile', type=check_mobile, location='json', required=True, )
        parser.add_argument('code', type=str, location='json', required=True)
        args = parser.parse_args()
        mobile=args.get('mobile')
        sms_code=args.get('code')

        # 验证短信码省略
        # 从redis中获取验证码
        key = 'app:code:{}'.format(mobile)
        redis_code = current_app.redis_store.get(key)

        if not redis_code or redis_code.decode() != sms_code:
            return {'message': 'Invalid code.'}, 400

        # 3.根据手机号进行数据库查询，如果不存在则保存数据
        user = None
        try:
            user = User.query.filter_by(mobile=args.get('mobile')).first()
        except Exception as e:
            current_app.logger.error(e)

        if user is None:
            user = User()
            user.mobile = args.get('mobile')
            user.name = args.get('mobile')
            db.session.add(user)
            db.session.commit()
        else:
            from datetime import datetime
            user.last_login = datetime.now()
            db.session.commit()
        # 4.生成token
        token = get_user_login_token(user.id)
        # 5.返回相应
        return {'token': token}
```



