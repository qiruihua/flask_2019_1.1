# 验证token

## 判断请求头的token信息

```
#验证请求钩子   
from project.apps.user.utils import check_user_login_token
@app.before_request
def before_request():
    #初始化 user_id
    current_app.user_id=None
    #获取请求头的token
    token = request.headers.get('Authorization')
    # 对token进行解析验证，获取用户信息
    if token and token.startswith('Bearer '):
        data = token[7:]
        user_id = check_user_login_token(data)
        current_app.user_id=user_id
```

## 验证token

```
#验证token
from itsdangerous import TimedJSONWebSignatureSerializer,BadData
from settings import Config

def check_user_login_token(token):

    if token is None:
        return None

    serializer = TimedJSONWebSignatureSerializer(secret_key=Config.SECRET_KEY, expires_in=3600)
    try:
        result = serializer.loads(token)
    except BadData:
        return None
    else:
        return result.get('user_id')
```

## 判断用户登录完善

```
#必须登录装饰器
def loginrequired(func):

    def wrapper(*args,**kwargs):

        if current_app.user_id is None:
            return {
            "id": -1,
            "name": "",
            "photo": "",
            "intro": "",
            "art_count": 0,
            "follow_count": 0,
            "fans_count": 0
            }
        else:
            return func(*args,**kwargs)

    return wrapper

#定义字段
from flask_restful import fields
resource_fields = {
    'id': fields.Integer,
    'name': fields.String,
    'photo': fields.String,
    'intro': fields.String,
    'art_count': fields.Integer,
    'follow_count': fields.Integer,
    'fans_count': fields.Integer
}
#返回用户信息
class UserInfoResource(Resource):

    method_decorators = [loginrequired]

    def get(self):

        """
        1.根据用户id查询用户信息
        2.返回用户信息
        :return:
        """
        user_id=current_app.user_id
        user=User.query.get(user_id)
        if user is None:
            return {
            "id": -1,
            "name": "",
            "photo": "",
            "intro": "",
            "art_count": 0,
            "follow_count": 0,
            "fans_count": 0
            }
        else:

            return marshal(user, resource_fields)
```



