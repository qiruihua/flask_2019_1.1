# 用户信息

## 接口分析

**请求方式**：GET /app/v1\_0/user

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "id": 1155,
        "name": "18310820688",
        "photo": "xxxxx",
        "intro": "xxx",
        "art_count": 0,
        "follow_count": 0,
        "fans_count": 0
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| id | str | 是 | 用户id |
| name | str | 是 | 昵称 |
| photo | str | 是 | 头像 |
| intro | str | 是 | 简介 |
| art\_count | int | 是 | 文章数量 |
| follow\_count | int | 是 | 关注数量 |
| fans\_count | int | 是 | 粉丝数量 |

## 后端实现

判断用户是否登录的装饰器

```
#必须登录装饰器
def loginrequired(func):

    def wrapper(*args,**kwargs):

        if g.user_id is None:
            return {'message': 'login'}, 401
        else:
            return func(*args,**kwargs)

    return wrapper

from project.utils.decorators import loginrequired
```

返回用户信息

```
class UserInfoResource(Resource):

    method_decorators = [loginrequired]

    def get(self):

        """
        1.根据用户id查询用户信息
        2.返回用户信息
        :return:
        """
        user_data=UserProfileCache(g.user_id).get()
        user_data['id']=g.user_id
        del user_data['mobile']
        return user_data
```

缓存类完善

```
class UserProfileCache(object):
    """
    用户信息缓存
    """

    def get(self):
        """
        获取用户数据
        :return:
        """
        try:
            ret = current_app.redis_store.get(self.key)
        except RedisError as e:
            current_app.logger.error(e)
            ret = None

        user_data = None
        if ret:
            user_data = json.loads(ret)

        if not user_data['photo']:
            user_data['photo'] = constants.DEFAULT_USER_PROFILE_PHOTO
        return user_data
```



