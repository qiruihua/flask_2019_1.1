# 用户关注频道

## 接口分析

**请求方式**：GET /app/v1\_0/user/channels

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "channels": [
            {
                "id": 0,
                "name": "推荐"
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
| name | str | 是 | 频道名字 |

## 模型类

```
class UserChannel(db.Model):
    """
    用户关注频道表
    """
    __tablename__ = 'news_user_channel'

    id = db.Column('user_channel_id', db.Integer, primary_key=True, doc='主键ID')
    user_id = db.Column(db.Integer, doc='用户ID')
    channel_id = db.Column(db.Integer, db.ForeignKey('news_channel.channel_id'), doc='频道ID')
    ctime = db.Column('create_time', db.DateTime, default=datetime.now, doc='创建时间')
    is_deleted = db.Column(db.Boolean, default=False, doc='是否删除')
    utime = db.Column('update_time', db.DateTime, default=datetime.now, onupdate=datetime.now, doc='更新时间')
    sequence = db.Column(db.Integer, default=0, doc='序号')

    channel = db.relationship('Channel', uselist=False)
```

## 后端实现

```
from models.news import UserChannel
class UserChannelsResource(Resource):
    

    def get(self):
        """
        1.获取用户id
        2.根据用户进行数据查询
        3.返回数据
        :return:
        """
        # 1.获取用户id
        user_id=current_app.user_id
        # 2.根据用户进行数据查询
        channels=UserChannel.query.filter_by(user_id=user_id,is_deleted=False).all()
        # 3.返回数据
        data_list=[{
            'id':0,
            'name':'推荐'
        }]
        for channel in channels:
            data_list.append({
                'id':channel.id,
                'name':channel.channel.name
            })
        return {"channels":data_list}
```

## 添加缓存



