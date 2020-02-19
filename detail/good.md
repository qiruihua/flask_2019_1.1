# 点赞文章

## 接口分析

**请求方式**：POST /app/v1\_0/article/likings

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token**\(headers\)** | str | 是 | 用户token |
| target**\(body\)** | str | 是 | 点赞用户id |

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
| target | str | 是 | 点赞id |

## 模型类

```
class Attitude(db.Model):
    """
    用户文章态度表
    """
    __tablename__ = 'news_attitude'

    class ATTITUDE:
        DISLIKE = 0  # 不喜欢
        LIKING = 1  # 点赞

    id = db.Column('attitude_id', db.Integer, primary_key=True, doc='主键ID')
    user_id = db.Column(db.Integer, doc='用户ID')
    article_id = db.Column(db.Integer, db.ForeignKey('news_article_basic.article_id'), doc='文章ID')
    attitude = db.Column(db.Integer, doc='态度')
    ctime = db.Column('create_time', db.DateTime, default=datetime.now, doc='创建时间')
    utime = db.Column('update_time', db.DateTime, default=datetime.now, onupdate=datetime.now, doc='更新时间')

    article = db.relationship('Article', uselist=False)
```

## 后端实现

```
from toutiao.utils.decorators import loginrequired
from flask_restful.reqparse import RequestParser
from models.news import Attitude

class ArticleLikeResource(Resource):
    """
    文章点赞
    """
    method_decorators = [loginrequired]

    def post(self):
        """
        文章点赞
        """
        # 入参检验
        json_parser = RequestParser()
        json_parser.add_argument('target', type=int, required=True, location='json')
        args = json_parser.parse_args()
        target = args.target

        # 更新或创建数据
        atti = Attitude.query.filter_by(user_id=g.user_id, article_id=target).first()
        if atti is None:
            atti = Attitude(user_id=g.user_id, article_id=target, attitude=Attitude.ATTITUDE.LIKING)
        db.session.add(atti)
        db.session.commit()

        return {'target': target}, 201
```

![](/assets/点赞.png)

