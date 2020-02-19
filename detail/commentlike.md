# 评论或评论回复点赞 {#评论或评论回复点赞取消点赞}

## 接口分析

**请求方式**：POST /app/v1\_0/comment/likings

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
class CommentLiking(db.Model):
    """
    评论点赞
    """
    __tablename__ = 'news_comment_liking'

    id = db.Column('liking_id', db.Integer, primary_key=True, doc='主键ID')
    user_id = db.Column(db.Integer, doc='用户ID')
    comment_id = db.Column(db.Integer, doc='评论ID')
    ctime = db.Column('create_time', db.DateTime, default=datetime.now, doc='创建时间')
    is_deleted = db.Column(db.Boolean, default=False, doc='是否删除')
    utime = db.Column('update_time', db.DateTime, default=datetime.now, onupdate=datetime.now, doc='更新时间')
```

## 后端实现

```
from toutiao.utils.decorators import loginrequired
from models.news import CommentLiking

class CommentLikingResource(Resource):
    """
    评论点赞
    """
    method_decorators = [loginrequired]

    def post(self):
        """
        评论点赞
        """
        # 参数解析
        json_parser = RequestParser()
        json_parser.add_argument('target', type=int, required=True, location='json')
        args = json_parser.parse_args()
        target = args.target
        ret = 1
        # 创建或更新数据
        try:
            comment_liking = CommentLiking(user_id=g.user_id, comment_id=target)
            db.session.add(comment_liking)
            db.session.commit()
        except IntegrityError:
            db.session.rollback()
            CommentLiking.query.filter_by(user_id=g.user_id, comment_id=target, is_deleted=True) \
                .update({'is_deleted': False})
            db.session.commit()

        return {'target': target}, 201
```



