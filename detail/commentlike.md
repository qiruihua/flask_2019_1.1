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

## 缓存用户评论点赞数据

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| user:{user\_id}:comm:liking | set | - | \[1,2,3\] |

在cache的user.p文件中，实现评论点赞缓存

```
from models.news import CommentLiking
class UserCommentLikingCache(object):
    """
    用户评论点赞缓存数据
    """

    def __init__(self, user_id):
        self.key = 'user:{}:comm:liking'.format(user_id)
        self.user_id = user_id

    def get(self):
        """
        获取用户文章评论点赞数据
        :return:
        """

        try:
            ret = current_app.redis_store.smembers(self.key)
        except RedisError as e:
            current_app.logger.error(e)
            ret = None

        if ret:
            return set([int(cid) for cid in ret])

        ret = CommentLiking.query.filter(CommentLiking.user_id == self.user_id,
                    CommentLiking.is_deleted == False).all()

        cids = [com.comment_id for com in ret]
        pl = current_app.redis_store.pipeline()

        try:

            pl.sadd(self.key, *cids)
            pl.expire(self.key, constants.UserCommentLikingCacheTTL.get_val())
            pl.execute()

        except RedisError as e:
            current_app.logger.error(e)

        return set(cids)

    def user_liking_comment(self, comment_id):
        """
        判断是否对文章点赞
        :param comment_id:
        :return:
        """
        liking_comments = self.get()

        return comment_id in liking_comments

    def clear(self):
        """
        清除
        """
        current_app.redis_store.delete(self.key)
```

在constants.py文件中定义缓存时间常量

```
class UserCommentLikingCacheTTL(BaseCacheTTL):
    """
    用户文章评论点赞缓存时间，秒
    """
    TTL = 10 * 60
```

## 添加是否点赞评论判断

```
            from cache.user import UserCommentLikingCache
            is_liking=UserCommentLikingCache(comment.user_id).user_liking_comment(comment.id)
            comment_dict = {
                'com_id': comment.id,
                'aut_id': comment.user.id,
                'aut_name': comment.user.name,
                'aut_photo': comment.user.profile_photo,
                'pubdate': comment.ctime.strftime('%Y-%m-%d %H:%M:%S'),
                'content': comment.content,
                'is_top': comment.is_top,
                'is_liking': is_liking,
                'reply_count': 0
            }
```

## 在点赞和取消点赞评论视图处添加清除缓存功能

```
        from cache.user import UserCommentLikingCache
        UserCommentLikingCache(g.user_id).clear()
```



