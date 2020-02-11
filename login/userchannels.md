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
from flask import g

class UserChannelsResource(Resource):

    def get(self):
        """
        1.获取用户id
        2.根据用户id进行判断
        3.登陆用户进行数据查询
        4.未登录用户返回默认频道数据
        5.返回数据
        :return:
        """
        # 1.获取用户id
        user_id=g.user_id
        # 2.根据用户id进行判断
        if user_id:
            # 3.登陆用户进行数据查询
            channels = UserChannel.query.filter(UserChannel.user_id==user_id,
                                                UserChannel.is_deleted==False).\
                order_by(UserChannel.sequence).all()
        else:
            # 4.未登录用户返回默认频道数据
            channels = Channel.query.filter(Channel.is_default == True,
                                        Channel.is_visible == True).\
            order_by(Channel.sequence, Channel.id).all()


        # 5.处理数据
        data_list=[{
            'id':0,
            'name':'推荐'
        }]
        for channel in channels:
            data_list.append({
                'id':channel.id,
                'name':channel.channel.name if user_id else channel.name
            })
        return {"channels":data_list}
```

## 添加缓存

###  {#3-article-cache}

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| ch:user:default | string | 默认用户频道 | key:value |
| user:{user\_id}:ch | string | 用户频道 | key:value |

###  {#4-announcement-cache}

定义默认用户频道缓存

定义登录用户频道缓存

### 添加缓存时间变量

```
# 默认用户频道缓存有效期，秒
DEFAULT_USER_CHANNELS_CACHE_TTL = 24 * 60 * 60
```

### 修改获取所有频道视图实现逻辑



