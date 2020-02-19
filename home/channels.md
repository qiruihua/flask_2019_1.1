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

## 模型类

```
class Channel(db.Model):
    """
    新闻频道
    """
    __tablename__ = 'news_channel'

    id = db.Column('channel_id', db.Integer, primary_key=True, doc='频道ID')
    name = db.Column('channel_name', db.String, doc='频道名称')
    ctime = db.Column('create_time', db.DateTime, default=datetime.now, doc='创建时间')
    utime = db.Column('update_time', db.DateTime, default=datetime.now, onupdate=datetime.now, doc='更新时间')
    sequence = db.Column(db.Integer, default=0, doc='序号')
    is_visible = db.Column(db.Boolean, default=False, doc='是否可见')
    is_default = db.Column(db.Boolean, default=False, doc='是否默认')
```

## 后端实现

```
from models.news import Channel
from flask_restful import fields,marshal

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
        channels = Channel.query.filter(Channel.is_visible == True).order_by(Channel.sequence, Channel.id).all()
        # 3.返回数据
        return marshal(channels, channels_fields,envelope='channels')
```

## 定义缓存类

在common目录的cache包中新建一个channel.py文件

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| ch:all | string | 所有频道 | key:value |

```
from flask import current_app
import json
from redis import RedisError
from models.news import Channel
from flask_restful import fields,marshal
from . import constants
#将频道列表对象转换
channels_fields = {
    'id': fields.Integer,
    'name': fields.String,
}

class AllChannelsCache(object):
    """
    全部频道缓存
    """
    key = 'ch:all'

    @classmethod
    def get(cls):
        """
        获取
        :return: {'channels':[{'name': 'python', 'id': '123'}, {}]}
        """

        # 先从缓存取数据，能取到缓存数据就直接返回
        ret = current_app.redis_store.get(cls.key)
        if ret:
            results = json.loads(ret.decode())
            return results

        channels = Channel.query.filter(Channel.is_visible == True).order_by(Channel.sequence, Channel.id).all()
        # 取不到缓存数据，则进行数据库查询
        results=marshal(channels, channels_fields, envelope='channels')

        # 返回前，先将数据缓存起来，这样下次就可以直接在缓存中取数据了
        try:
            current_app.redis_store.setex(cls.key, constants.ALLCHANNELSCACHETTL.get_val(), json.dumps(results))
        except RedisError as e:
            current_app.logger.error(e)

        return results
```

### 添加缓存时间变量

```
class ALLCHANNELSCACHETTL(BaseCacheTTL):
    """
    全部频道缓存有效期，秒
    """
    TTL = 24 * 60 * 60
```

### 修改获取所有频道视图实现逻辑

```
from cache.channel import AllChannelsCache
class ChannelsResource(Resource):

    def get(self):
        return AllChannelsCache.get()
```



