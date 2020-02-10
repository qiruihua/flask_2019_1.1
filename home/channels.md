# 获取所有频道

## 接口分析

**请求方式**：GET /app/v1\_0/channels

**请求参数**：无

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
from flask_restful import fields
#将频道列表对象转换
channels_fields = {
    'id': fields.Integer,
    'name': fields.String,
}
class ChannelsResource(Resource):

    def get(self):
        """
        1.获取所有频道
        2.将频道列表对象转换
        3.返回相应
        :return:
        """
        # 1.获取所有频道
        channels = Channel.query.all()
        # 3.返回数据
        return marshal(channels, channels_fields,envelope='channels')
```

## 定义缓存类

```
class AllChannelsCache(object):
    """
    全部频道缓存
    """
    key = 'ch:all'

    @classmethod
    def get(cls):
        """
        获取
        :return: [{'name': 'python', 'id': '123'}, {}]
        """

        # 先从缓存取数据，能取到缓存数据就直接返回
        ret = current_app.redis_store.get(cls.key)
        if ret:
            results = json.loads(ret)
            return results

        # 取不到缓存数据，则进行数据库查询
        results = []

        channels = Channel.query.options(load_only(Channel.id, Channel.name)) \
            .filter(Channel.is_visible==True).order_by(Channel.sequence, Channel.id).all()
        if not channels:
            return results

        for channel in channels:
            results.append({
                'id': channel.id,
                'name': channel.name
            })

        # 返回前，先将数据缓存起来，这样下次就可以直接在缓存中取数据了
        try:
            current_app.redis_store.setex(cls.key, constants.ALL_CHANNELS_CACHE_TTL, json.dumps(results))
        except RedisError as e:
            print(e)

        return results
```

修改视图

```
from cache.channel import AllChannelsCache
class ChannelsResource(Resource):

    def get(self):
        channels=AllChannelsCache.get()
        return {'channels':channels}
```



