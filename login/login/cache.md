# 用户信息缓存

## 用户缓存信息分析

用户资料

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| user:{user\_id}:profile | string | 缓存手机号，用户名等信息 | 字典数据进行编码 |

用户状态

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| user:{user\_id}:status | string | 缓存用户是否可用 | 1可用 0不可用 |

## 定义缓存类

在common目录中新建cache包，并新建一个user.py文件

![](/assets/common_cache.png)

user.py实现缓存

```
from flask import current_app
from redis import RedisError
from models.user import User
from . import constants
import json

class UserProfileCache(object):
    """
    用户信息缓存
    """
    def __init__(self, user_id):
        self.key = 'user:{}:profile'.format(user_id)
        self.user_id = user_id

    def save(self,user=None):
        """
        user 如果传递过来就可以不用再次查询数据库

        保存缓存
        1.判断缓存是否存在
        2.如果不存在则添加缓存
            查询数据库
            将对象数据转换为字典
            将字典进行编码
        """
        try:
            ret = current_app.redis_store.get(self.key)
        except RedisError as e:
            current_app.logger.error(e)
            exists = False
        else:
            if ret is not None:
                exists = True
            else:
                exists = False

        # 用户缓存不存在
        if not exists:
            if user is None:
                user = User.query.filter_by(id=self.user_id).first()
                #判断一下 是否有用户信息
                if user is None:
                    return None
            user_data = {
                'mobile': user.mobile,
                'name': user.name,
                'photo': user.profile_photo or '',
                'is_media': user.is_media,
                'intro': user.introduction or '',
                'certi': user.certificate or '',
            }

            try:
                current_app.redis_store.setex(self.key, constants.UserProfileCacheTTL.get_val(), json.dumps(user_data))
            except RedisError as e:
                current_app.logger.error(e)

            return user_data


class UserStatusCache(object):
    """
    用户状态缓存
    """
    def __init__(self, user_id):
        self.key = 'user:{}:status'.format(user_id)
        self.user_id = user_id

    def save(self, status):
        """
        设置用户状态缓存
        :param status:
        """
        try:
            current_app.redis_store.setex(self.key, constants.UserStatusCacheTTL.get_val(), status)
        except RedisError as e:
            current_app.logger.error(e)
```

## 定义缓存时间

在cache包中定义constants.py文件，用于存放缓存时间

```
class BaseCacheTTL(object):
    """
    由子类设置
    """
    TTL = 0

    @classmethod
    def get_val(cls):
        return cls.TTL


class UserProfileCacheTTL(BaseCacheTTL):
    """
    用户资料数据缓存时间, 秒
    """
    TTL = 30 * 60


class UserStatusCacheTTL(BaseCacheTTL):
    """
    用户状态缓存时间，秒
    """
    TTL = 60 * 60
```

## 登录缓存实现

```
        # 4.生成token
        token,fresh_token = get_user_token(user.id)

        #添加缓存
        from cache.user import UserProfileCache,UserStatusCache
        # UserProfileCache(user.id).save()
        UserProfileCache(user.id).save(user)
        UserStatusCache(user.id).save(user.status)

        # 5.返回相应
        return {'token': token,'fresh_token':fresh_token}
```



