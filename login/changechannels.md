# 修改关注频道

![](/assets/修改关注频道.png)

## 接口分析

**请求方式**：PUT /app/v1\_0/user/channels

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token（**Header**） | str | 是 | 用户token |
| channels\(**body**\) | list | 是 | 频道列表 |
| id\(**body**\) | str | 是 | 频道id |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "channels": [
            {
                "id": 11,
                "name": "html"
            },
            {
                "id": 7,
                "name": "数据库"
            }
        ]
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| channels | list | 是 | 频道列表 |
| id | str | 是 | 频道id |
| name | str | 是 | 频道名 |

## 后端实现

```
from models.news import UserChannel
from flask import g,request
from toutiao.utils.decorators import loginrequired
from toutiao import db

class UserChannelsResource(Resource):

    method_decorators = {
        'put':[loginrequired]
    }

    def put(self):
        """
        1.接收数据
        2.更新数据
        3.返回相应
        """
        user_id = g.user_id
        # 1.接收数据
        channel_list = request.json.get('channels')
        # 2.更新数据
        UserChannel.query.filter_by(user_id=user_id, is_deleted=False).update({'is_deleted': True})

        for channel in channel_list:
            data = {"sequence": channel['seq'], "is_deleted": False}
            flag = UserChannel.query.filter_by(user_id=user_id,
                                               channel_id=channel['id']).update(data)
            if flag == 0:
                user_channel = UserChannel()
                user_channel.user_id = user_id
                user_channel.sequence = channel.get('seq')
                user_channel.channel_id = channel.get('id')
                db.session.add(user_channel)

            db.session.commit()

        # 3.返回相应
        return {'channels': channel_list}, 201
```

> 修改用户关注频道之后，要清除用户关注的缓存

### 添加清除缓存功能

给UserChannelsCache添加clear函数

```
class UserChannelsCache(object):
    """
    用户频道缓存
    """
    def __init__(self, user_id):
        self.key = 'user:{}:ch'.format(user_id)
        self.user_id = user_id

    def clear(self):
        """
        清除
        """
        try:
            current_app.redis_store.delete(self.key)
        except RedisError as e:
            current_app.logger.error(e)
```

在UserChannelsResource的put方法中添加清除缓存功能

```
        from cache.channel import UserChannelsCache
        UserChannelsCache(user_id).clear()
        # 3.返回相应
        return {'channels': channel_list}, 201
```



