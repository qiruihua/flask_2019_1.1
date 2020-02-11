# 首页文章列表

## 接口分析

**请求方式**：GET /app/v1\__0/articles?channel\_id=xxx&timestamp_=xxx&_with\_top_=xxx

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 否 | 用户token |
| channel\_id（**Query**） | str | 是 | 频道id |
| timestamp\(**Query**\) | int | 是 | 时间戳，请求新的推荐数据传当前的时间戳，请求历史推荐传指定的时间戳 |
| with\_top\(**Query**\) | bool | 是 | 是否包含置顶。进入页面第一次请求要包含指定文章，1表示包含 0表示不包含 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "pre_timestamp": 2,
        "results": [
            {
                "art_id": 140901,
                "title": "机器学习技术前沿与未来展望",
                "aut_id": 1,
                "pubdate": "2018-11-29T17:18:33",
                "aut_name": "黑马头条号",
                "comm_count": 0,
                "is_top":0,
                "cover":[{"type": 0, "images": []}]
            },
            {
                "art_id": 139940,
                "title": "Enscape 2.4新功能预览",
                "aut_id": 1,
                "pubdate": "2018-11-29T17:17:46",
                "aut_name": "黑马头条号",
                "comm_count": 0,
                "is_top":0,
                "cover":[{"type": 0, "images": []}]
            }
        ]
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| per\_timestamp | int | 是 | 请求前一页数据的时间戳 |
| results | list | 是 | 文章列表 |
| art\_id | str | 是 | 文章id |
| title | str | 是 | 文章标题 |
| aut\_id | str | 是 | 作者id |
| pubdate | str | 是 | 发布日期 |
| aut\_name | str | 是 | 作者名字 |
| comm\_count | int | 是 | 评论数量 |
| is\_top | int | 是 | 是否置顶 0不置顶1置顶 |
| cover | list | 是 | 封面 |
| type | int | 是 | 0无封面 1一张 3 三张 |

## 模型类

```
class Article(db.Model):
    """
    文章基本信息表
    """
    __tablename__ = 'news_article_basic'

    # 只有APPROVED状态的文章可以被用户查看
    class STATUS:
        DRAFT = 0  # 草稿
        UNREVIEWED = 1  # 待审核
        APPROVED = 2  # 审核通过
        FAILED = 3  # 审核失败
        DELETED = 4  # 已删除
        BANNED = 5  # 封禁

    STATUS_ENUM = [0, 1, 2, 3]

    id = db.Column('article_id', db.Integer, primary_key=True,  doc='文章ID')
    user_id = db.Column(db.Integer, db.ForeignKey('user_basic.user_id'), doc='用户ID')
    channel_id = db.Column(db.Integer, db.ForeignKey('news_channel.channel_id'), doc='频道ID')
    title = db.Column(db.String, doc='标题')
    cover = db.Column(db.JSON, doc='封面')
    is_advertising = db.Column(db.Boolean, default=False, doc='是否投放广告')
    ctime = db.Column('create_time', db.DateTime, default=datetime.now, doc='创建时间')
    status = db.Column(db.Integer, default=0, doc='帖文状态')
    reviewer_id = db.Column(db.Integer, doc='审核人员ID')
    review_time = db.Column(db.DateTime, doc='审核时间')
    delete_time = db.Column(db.DateTime, doc='删除时间')
    comment_count = db.Column(db.Integer, default=0, doc='评论数')
    allow_comment = db.Column(db.Boolean, default=True, doc='是否允许评论')
    reject_reason = db.Column(db.String, doc='驳回原因')
    utime = db.Column('update_time', db.DateTime, default=datetime.now, doc='更新时间')

    content = db.relationship('ArticleContent', uselist=False)
    user = db.relationship('User', uselist=False)
    # statistic = db.relationship('ArticleStatistic', uselist=False)
    channel = db.relationship('Channel', uselist=False)


class ArticleContent(db.Model):
    """
    文章内容表
    """
    __tablename__ = 'news_article_content'

    id = db.Column('article_id', db.Integer, db.ForeignKey('news_article_basic.article_id'), primary_key=True, doc='文章ID')
    content = db.Column(db.Text, doc='帖文内容')
```

## 后端实现

```
from flask_restful.reqparse import RequestParser
from flask_restful import inputs
from . import constants
from models.news import Article

class IndexResource(Resource):

    def get(self):
        """
        1.接收参数，并验证参数
        2.查询数据
        3.将数据转换为字典
        4.返回数据
        :return:
        """
        # 1.验证参数
        qs_parser = RequestParser()
        qs_parser.add_argument('channel_id', type=int, required=True, location='args')
        qs_parser.add_argument('timestamp', type=inputs.positive, required=True, location='args')
        qs_parser.add_argument('with_top', type=inputs.boolean, required=True, location='args')
        args = qs_parser.parse_args()
        channel_id = args.channel_id
        per_page = constants.DEFAULT_ARTICLE_PER_PAGE_MIN

        #不为0会一直获取数据 我们就获取一次数据
        pre_timestamp = 0
        #偏移量
        offset = 0

        # 获取文章列表
        articles=Article.query.filter_by(channel_id=channel_id,
                                status=Article.STATUS.APPROVED).\
            order_by(Article.id.desc()).\
            offset(offset).limit(per_page).all()
        # 4.将对象列表数据转换为字典，返回相应
        results = []
        for item in articles:
            results.append({
                "art_id": item.id,
                "title": item.title,
                "aut_id": item.user.id,
                "pubdate": item.ctime.strftime('%Y-%m-%d %H:%M:%S'),
                "aut_name": item.user.name,
                "comm_count": item.comment_count,
                "is_top":False,
                'cover':item.cover,
            })

        return {'pre_timestamp': pre_timestamp, 'results': results}

```



