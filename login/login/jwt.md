# JWT Python库

* itsdangerous

  * JSONWebSignatureSerializer
  * TimedJSONWebSignatureSerializer （可设置有效期）

* pyjwt

  [https://pyjwt.readthedocs.io/en/latest/](https://pyjwt.readthedocs.io/en/latest/)

  ## 安装

  ```
    $ pip install pyjwt
  ```

  ## 用例

  ```
    >>> import jwt

    >>> encoded_jwt = jwt.encode({'some': 'payload'}, 'secret', algorithm='HS256')
    >>> encoded_jwt
    'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzb21lIjoicGF5bG9hZCJ9.4twFt5NiznN84AWoo1d7KO1T_yoc0Z6XOpOVswacPZg'

    >>> jwt.decode(encoded_jwt, 'secret', algorithms=['HS256'])
    {'some': 'payload'}
  ```

###  {#需求}



