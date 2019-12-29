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



