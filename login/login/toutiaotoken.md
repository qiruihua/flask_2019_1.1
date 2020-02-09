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

```
def generate_tokens(self, user_id, with_refresh_token=True):
        """
        生成token 和refresh_token
        :param user_id: 用户id
        :return: token, refresh_token
        """
        # 颁发JWT
        now = datetime.utcnow()
        expiry = now + timedelta(hours=current_app.config['JWT_EXPIRY_HOURS'])
        token = generate_jwt({'user_id': user_id, 'refresh': False}, expiry)
        refresh_token = None
        if with_refresh_token:
            refresh_expiry = now + timedelta(days=current_app.config['JWT_REFRESH_DAYS'])
            refresh_token = generate_jwt({'user_id': user_id, 'refresh': True}, refresh_expiry)
        return token, refresh_token
```



