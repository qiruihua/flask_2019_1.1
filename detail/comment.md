# 发布评论

## 接口分析

**请求方式**：POST /app/v1\_0/comments

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |
| target | str | 是 | 文章id |
| content | str | 是 | 评论内容 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "com_id": 108,
        "target": 138154
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| com\_id | str | 是 | 评论id |
| target | str | 是 | 文章id |

## 模型类

```
class Comment(db.Model):
    """
    文章评论
    """
    __tablename__ = 'news_comment'

    class STATUS:
        UNREVIEWED = 0  # 待审核
        APPROVED = 1  # 审核通过
        FAILED = 2  # 审核失败
        DELETED = 3  # 已删除

    id = db.Column('comment_id', db.Integer, primary_key=True, doc='评论ID')
    user_id = db.Column(db.Integer, db.ForeignKey('user_basic.user_id'), doc='用户ID')
    article_id = db.Column(db.Integer, db.ForeignKey('news_article_basic.article_id'), doc='文章ID')
    parent_id = db.Column(db.Integer, doc='被评论的评论id')
    like_count = db.Column(db.Integer, default=0, doc='点赞数')
    reply_count = db.Column(db.Integer, default=0, doc='回复数')
    content = db.Column(db.String, doc='评论内容')
    is_top = db.Column(db.Boolean, default=False, doc='是否置顶')
    status = db.Column(db.Integer, default=1, doc='评论状态')
    ctime = db.Column('create_time', db.DateTime, default=datetime.now, doc='创建时间')

    user = db.relationship('User', uselist=False)
    article = db.relationship('Article', uselist=False)

```

## 后端实现

```
from flask_restful import reqparse
from project.utils.user import loginrequired
class CommentsResource(Resource):
    method_decorators = {
        'post':[loginrequired]
    }

    def post(self):
        """
        1.接收数据
        2.验证数据
        3.数据入库
        4.返回相应
        :return:
        """
        user_id=current_app.user_id

        parse=reqparse.RequestParser()
        parse.add_argument('target',required=True)
        parse.add_argument('content',required=True)
        args=parse.parse_args()

        comment=Comment()
        comment.article_id=args.get('target')
        comment.content=args.get('content')
        comment.user_id=user_id
        try:
            db.session.add(comment)
            db.session.commit()
        except Exception:
            return {}

        return {
            "com_id": comment.id,
            "target": comment.article_id
        }
```



