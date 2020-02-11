# 验证token

我们该如何获取到用户的user\_id呢？当然是根据token信息

## 获取请求头的token信息，并解析

我们通过请求钩子来获取请求头中的token信息

在toutiao的_init_.py中添加token的获取和验证

```
#验证请求钩子
from flask import g,request
from auth.auth_jwt import verify_jwt
@app.before_request
def before_request():
    #初始化 user_id
    g.user_id=None
    #获取请求头的token
    authorization = request.headers.get('Authorization')
    # 对token进行解析验证，获取用户信息
    if authorization and authorization.startswith('Bearer '):
        token = authorization[7:]
        payload = verify_jwt(token)
        if payload:
            g.user_id = payload.get('user_id')
            g.is_refresh_token = payload.get('refresh')
```

## 验证token

```
def verify_jwt(token,secret=None):
    """
    检验jwt
    :param token: jwt
    :param secret: 密钥
    :return: dict: payload
    """
    if not secret:
        secret = current_app.config['JWT_SECRET']

    try:
        payload = jwt.decode(token, secret, algorithm=['HS256'])
    except jwt.PyJWTError:
        payload = None

    return payload
```

## 判断用户登录完善

```
#必须登录装饰器
from flask import g
def loginrequired(func):

    def wrapper(*args,**kwargs):

        if g.user_id is None:
            return {'message': 'User must be authorized.'}, 401
        elif g.is_refresh_token:
            return {'message': 'Do not use refresh token.'}, 403
        else:
            return func(*args,**kwargs)

    return wrapper
```



