# 头条项目实施方案 {#头条项目实施方案}

### 需求 {#需求}

设置有效期，但有效期不宜过长，需要刷新。

如何解决刷新问题？

* 手机号+验证码（或帐号+密码）验证后颁发接口调用token与refresh\_token（刷新token）

* Token 有效期为2小时，在调用接口时携带，每2小时刷新一次

* 提供refresh\_token，refresh\_token 有效期14天

* 在接口调用token过期后凭借refresh\_token 获取新token

* 未携带token 、错误的token或接口调用token过期，返回401状态码

* refresh\_token 过期返回403状态码，前端在使用refresh\_token请求新token时遇到403状态码则进入用户登录界面从新认证。

* token的携带方式是在请求头中使用如下格式：

  ```
  Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzb21lIjoicGF5bG9hZCJ9.4twFt5NiznN84AWoo1d7KO1T_yoc0Z6XOpOVswacPZg
  ```

  注意：Bearer前缀与token中间有一个空格

## 接口修改

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
    "token": "xxxxxxxx",
    "refresh_token":"xxxxxxx"
  }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| token | str | 是 | 用户token |
| refresh\_token | str | 是 | 刷新token |

### 修改token.py文件中get\_user\_token方法实现

```
from datetime import datetime, timedelta
from flask import current_app
from auth.auth_jwt import generate_jwt

def get_user_token(user_id):
    """
    生成token 和refresh_token
    :param user_id: 用户id
    :return: token, refresh_token
    """
    # 颁发JWT
    now = datetime.utcnow()

    expiry = now + timedelta(hours=current_app.config['JWT_EXPIRY_HOURS'])
    token = generate_jwt({'user_id': user_id, 'refresh': False}, expiry)

    refresh_expiry = now + timedelta(days=current_app.config['JWT_REFRESH_DAYS'])
    refresh_token = generate_jwt({'user_id': user_id, 'refresh': True}, refresh_expiry)
    return token, refresh_token
```

### 修改登录视图的返回数据

```
    # 4.生成token
    token,refresh_token = get_user_token(user.id)
    # 5.返回相应
    return {'token': token,'refresh_token':refresh_token}
```

![](/assets/登录成功响应.png)

