# 统计数据

## 持久存储设计

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| count:art:reading | zset | 文章阅读数量 | \[{article\_id, count}\] |
| count:art:collecting | zset | 文章收藏数量 | \[{article\_id, count}\] |
| count:art:liking | zset | 文章点赞数量 | \[{article\_id, count}\] |
| count:art:comm | zset | 文章评论数量 | \[{article\_id, count}\] |

## 定义存储基类

```
from redis.exceptions import RedisError
from flask import current_app

class CountStorageBase(object):
    """
    数据存储父类
    """
    key = ''

    @classmethod
    def get(cls, id_value):
        """
        获取
        """
        try:
            count = current_app.redis_store.zscore(cls.key, id_value)
        except RedisError as e:
            current_app.logger.error(e)
            count = 0

        count = 0 if count is None else int(count)

        return count

    @classmethod
    def incr(cls, id_value, increment=1):
        """
        增加
        """
        try:
            current_app.redis_store.zincrby(cls.key, id_value, increment)
        except RedisError as e:
            current_app.logge.error(e)
```

## 定义其他存储类

```
class ArticleReadingCountStorage(CountStorageBase):
    """
    文章阅读量
    """
    key = 'count:art:reading'

class ArticleCollectingCountStorage(CountStorageBase):
    """
    文章收藏数量
    """
    key = 'count:art:collecting'

class ArticleDislikeCountStorage(CountStorageBase):
    """
    文章不喜欢数据
    """
    key = 'count:art:dislike'

class ArticleLikingCountStorage(CountStorageBase):
    """
    文章点赞数据
    """
    key = 'count:art:liking'

class ArticleCommentCountStorage(CountStorageBase):
    """
    文章评论数量
    """
    key = 'count:art:comm'

class CommentReplyCountStorage(CountStorageBase):
    """
    评论回复数量
    """
    key = 'count:art:reply'
```

## 在视图中调用实现

### 记录回复评论数据

```
        if art_id:
            comment.article_id = art_id
            comment.parent_id = args.target
            from cache.statistic import CommentReplyCountStorage
            CommentReplyCountStorage.incr(args.target)
```



