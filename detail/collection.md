# 喜欢文章

## 接口分析

**请求方式**：POST /app/v1\_0/article/collections

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token**\(headers\)** | str | 是 | 用户token |
| target**\(body\)** | str | 是 | 收藏的文章id |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "target": 1
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| target | str | 是 | 收藏的文章id |

## 模型类

```
class Collection(db.Model):
    """
    用户收藏表
    """
    __tablename__ = 'news_collection'

    id = db.Column('collection_id', db.Integer, primary_key=True, doc='主键ID')
    user_id = db.Column(db.Integer, doc='用户ID')
    article_id = db.Column(db.Integer, doc='文章ID')
    ctime = db.Column('create_time', db.DateTime, default=datetime.now, doc='创建时间')
    is_deleted = db.Column(db.Boolean, default=False, doc='是否删除')
    utime = db.Column('update_time', db.DateTime, default=datetime.now, onupdate=datetime.now, doc='更新时间')

```

## 后端实现

```
from project.utils.decorators import loginrequired
from flask_restful.reqparse import RequestParser
from models.news import Attitude

class DislikeResource(Resource):
    """
    用户不喜欢
    """
    method_decorators = [loginrequired]

    def post(self):
        """
        不喜欢
        """
        json_parser = RequestParser()
        json_parser.add_argument('target', type=int, required=True, location='json')
        args = json_parser.parse_args()
        target = args.target

        atti = Attitude.query.filter_by(user_id=g.user_id, article_id=target).first()
        if atti is None:
            atti = Attitude(user_id=g.user_id, article_id=target, attitude=Attitude.ATTITUDE.DISLIKE)
        else:
            atti.attitude = Attitude.ATTITUDE.DISLIKE

        db.session.add(atti)
        db.session.commit()

        return {'target': target}, 201
```

## 缓存关注信息



