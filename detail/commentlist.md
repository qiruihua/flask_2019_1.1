# 评论/回复评论列表

## 接口分析

**请求方式**：GET /app/v1\_0/comments?source=x&offset=xxx&limit=xxx

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| type\(**query**\) | str | 是 | 评论类型，a表示文章评论 c表示回复评论 |
| source\(**query**\) | int | 是 | 文章id或者评论id |
| offset\(**query**\) | int | 否 | 获取评论数据的偏移量 |
| limit\(**query**\) | int | 否 | 评论条数 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "total_count": 1,
        "end_id": 108,  # 最后一条评论的id, 前端用于判断是否剩余评论, 无值返回None
        "last_id": 108,  # 本次请求最后一条评论的id, 作为下次请求的offset, 无值返回None
        "results": [
            {
                "com_id": 108,
                "aut_id": 1155989075455377414,
                "aut_name": "18310820688",
                "aut_photo": "",
                "pubdate": "2019-08-07T08:53:01",
                "content": "你写的真好",
               "is_top": 0,
               "is_liking": 0
            }
        ]
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| total\_count | int | 是 | 评论总数或评论回复总数 |
| end\_id | int | 是 | 所有评论或回复的最后一个id（截止offset值，小于此值的offset可以不用发送请求获取评论数据，已经没有数据），若无评论或回复数据，此值为NULL |
| last\_id | int | 是 | 本次返回结果的最后一个评论id，作为请求下一页数据的offset参数，若本次无具体数据，此值为NULL |
| results | list | 是 | 数据列表 |
| com\_id | int | 是 | 评论或回复id |
| aut\_id | int | 是 | 评论或回复用户id |
| aut\_name | str | 是 | 用户昵称 |
| aut\_photo | str | 是 | 用户头像 |
| pubdate | str | 是 | 创建时间 |
| content | str | 是 | 评论或回复内容 |
| is\_top | int | 是 | 是否置顶，0表示不置顶 1表示置顶 |
| is\_liking | bool | 是 | 当前用户是否点赞 |

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
from toutiao.apps.home import constants
from flask_restful.inputs import positive, int_range
class CommentsResource(Resource):

    def get(self):
        qs_parser = RequestParser()
        qs_parser.add_argument('type', type=str, required=True, location='args')
        qs_parser.add_argument('source', type=positive, required=True, location='args')
        qs_parser.add_argument('offset', type=positive, required=False, location='args')
        qs_parser.add_argument('limit', type=int_range(constants.DEFAULT_COMMENT_PER_PAGE_MIN,
                                                       constants.DEFAULT_COMMENT_PER_PAGE_MAX,
                                                       argument='limit'), required=False, location='args')
        args = qs_parser.parse_args()

        limit = args.limit if args.limit is not None else constants.DEFAULT_COMMENT_PER_PAGE_MIN
        offset = args.offset

        if args.type == 'a':
            # 文章评论
            article_id = args.source
            comments = Comment.query.filter(Comment.article_id == article_id,
                                            Comment.parent_id == None,
                                            Comment.status == Comment.STATUS.APPROVED). \
                order_by(Comment.id.desc()).all()
        else:
            comment_id = args.source
            comments = Comment.query.filter(Comment.parent_id == comment_id,
                                            Comment.status == Comment.STATUS.APPROVED). \
                order_by(Comment.id.desc()).all()

        page_comments = []
        page_count = 0
        total_count = len(comments)
        page_last_comment = None

        for comment in comments:
            score = comment.ctime.timestamp()

            # 构造返回数据
            if ((offset is not None and score < offset) or offset is None) and page_count <= limit:
                page_comments.append({
                    'com_id': comment.id,
                    'aut_id': comment.user.id,
                    'aut_name': comment.user.name,
                    'aut_photo': comment.user.profile_photo,
                    'pubdate': comment.ctime.strftime('%Y-%m-%d %H:%M:%S'),
                    'content': comment.content,
                    'is_top': comment.is_top,
                    'is_liking': False,
                    'reply_count': 0
                })
                page_count += 1
                page_last_comment = comment
        
        end_id=0
        last_id=0
        if len(comments)>0:
            end_id = comments[-1].ctime.timestamp()
            last_id = page_last_comment.ctime.timestamp() if page_last_comment else None

        return {'total_count': total_count, 'end_id': end_id, 'last_id': last_id, 'results': page_comments}
```

在home的constant中定义常量

```
# 评论分页默认每页数量 下限
DEFAULT_COMMENT_PER_PAGE_MIN = 10

# 评论分页默认每页数量 上限
DEFAULT_COMMENT_PER_PAGE_MAX = 50
```

![](/assets/评论列表.png)

