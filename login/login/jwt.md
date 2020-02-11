# JWT Python库

* itsdangerous

  * JSONWebSignatureSerializer
  * TimedJSONWebSignatureSerializer （可设置有效期）

* pyjwt

  [https://pyjwt.readthedocs.io/en/latest/](https://pyjwt.readthedocs.io/en/latest/)

  ### 安装

  ```
    $ pip install pyjwt
  ```

  ### 用例

  ```
    >>> import jwt

    >>> encoded_jwt = jwt.encode({'some': 'payload'}, 'secret', algorithm='HS256')
    >>> encoded_jwt
    'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzb21lIjoicGF5bG9hZCJ9.4twFt5NiznN84AWoo1d7KO1T_yoc0Z6XOpOVswacPZg'

    >>> jwt.decode(encoded_jwt, 'secret', algorithms=['HS256'])
    {'some': 'payload'}
  ```

## 项目中抽取使用

在common目录下定义auth包，并创建auth\_jwt文件

![](/assets/common_auth.png)

auth\_jwt主要实现加密和解密的封装代码

```
import jwt
from flask import current_app


def generate_jwt(payload, expiry, secret=None):
    """
    生成jwt
    :param payload: dict 载荷
    :param expiry: datetime 有效期
    :param secret: 密钥
    :return: jwt
    """
    _payload = {'exp': expiry}
    _payload.update(payload)

    if not secret:
        secret = current_app.config['JWT_SECRET']

    token = jwt.encode(_payload, secret, algorithm='HS256')
    return token.decode()

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





