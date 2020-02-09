# 验证token

判断请求头的token信息

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



