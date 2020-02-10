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
from project.models.user import Relation
class FollowResource(Resource):

    method_decorators = [loginrequired]

    def post(self):
        """
        1.接收参数
        2.验证参数
        3.数据入库
        4.返回相应
        :return:
        """
        user_id=current_app.user_id
        # 1.接收参数
        # 2.验证参数
        parse=reqparse.RequestParser()
        parse.add_argument('target',location='json',required=True)
        args=parse.parse_args()
        user=None
        try:
            user=User.query.get(args.get('target'))
        except Exception as e:
            current_app.logger.error(e)

        if user is None:
            abort(404)
        # 3.数据入库
        relation=Relation()
        relation.user_id=user_id
        relation.target_user_id=args.get('target')
        relation.relation=Relation.RELATION.FOLLOW
        try:
            db.session.add(relation)
            db.session.commit()
        except Exception as e:
            current_app.logger.error(e)
            return {'message':'error','data':{}}
        # 4.返回相应
        return {'target':args.get('target')}
```

## 缓存关注信息



