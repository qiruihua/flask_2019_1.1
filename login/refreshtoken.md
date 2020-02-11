# 刷新用户token

## 接口分析

**请求方式**：PUT /app/v1\_0/authorizations

**请求参数（Headers）**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "token": "xxxx",
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |

## 后端实现

在登录注册的视图中添加put请求

```
class LoginResource(Resource):

    def put(self):
        """
        刷新token
        """
        user_id = g.user_id
        if user_id and g.is_refresh_token:
            # 判断用户状态
            user_enable = UserStatusCache(g.user_id).get()
            if not user_enable:
                return {'message': 'User denied.'}, 403
            token, refresh_token = get_user_token(user_id)
            return {'token': token}, 201
        else:
            return {'message': 'Wrong refresh token.'}, 403
```

然后在**UserStatusCache**类中定义获取用户状态的方法**get**\(**common/cache/user.py**\)：

```
class UserStatusCache(object):
    """
    用户状态缓存
    """

    def get(self):
        """
        获取用户状态
        :return:
        """

        try:
            status = current_app.redis_store.get(self.key)
        except RedisError as e:
            print(e)
            status = None

        if status is not None:
            return status
        else:
            user = User.query.options(load_only(User.status)).filter_by(id=self.user_id).first()
            if user:
                self.save(user.status)
                return user.status
            else:
                return False
```



