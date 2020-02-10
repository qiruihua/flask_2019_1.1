# 首页文章列表

## 接口分析

**请求方式**：GET /app/v1\__0/articles?channel\_id=xxx&page=xxx&per\_page=xxx_

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 否 | 用户token |
| channel\_id | str | 是 | 频道id |
| page | int | 否 | 页码 |
| per\_page | int | 否 | 每页条数 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "pre_page": 2,
        "results": [
            {
                "art_id": 140901,
                "title": "机器学习技术前沿与未来展望",
                "aut_id": 1,
                "pubdate": "2018-11-29T17:18:33",
                "aut_name": "黑马头条号",
                "comm_count": 0
            },
            {
                "art_id": 139940,
                "title": "Enscape 2.4新功能预览",
                "aut_id": 1,
                "pubdate": "2018-11-29T17:17:46",
                "aut_name": "黑马头条号",
                "comm_count": 0
            }
        ]
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| per\_page | int | 否 | 文章数 |
| results | dict | 否 | 文章列表 |
| art\_id | str | 否 | 文章id |
| title | str | 否 | 文章标题 |
| aut\_id | str | 否 | 作者id |
| pubdate | str | 否 | 发布日期 |
| aut\_name | str | 否 | 作者名字 |
| comm\_count | int | 否 | 评论数量 |

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
class IndexResource(Resource):

    def get(self):
        """
        1.获取参数
        2.根据参数查询数据
        3.分页处理
        4.将对象列表数据转换为字典，返回相应
        :return:
        """
        # 1.获取参数
        channel_id=request.args.get('channel_id',0)
        page=request.args.get('page',1)
        per_page=request.args.get('per_page',10)

        try:
            page=int(page)
            per_page=int(per_page)
        except Exception:
            page=1
            per_page=10

        if int(channel_id) == 0:
            channel_id=1
        # 2.根据参数查询数据
        # 3.分页处理
        page_articles=Article.query.filter_by(channel_id=channel_id,
                                   status=Article.STATUS.APPROVED).paginate(page=page,
                                                                            per_page=per_page)

        # 4.将对象列表数据转换为字典，返回相应
        results = []
        for item in page_articles.items:
            results.append({
                "art_id": item.id,
                "title": item.title,
                "aut_id": item.user.id,
                "pubdate": item.ctime.strftime('%Y-%m-%d %H:%M:%S'),
                "aut_name": item.user.name,
                "comm_count": item.comment_count
            })

        return {'per_page': per_page, 'results': results}
```

## 缓存实现



