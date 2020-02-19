# 阅读历史列表

阅读历史进行持久存储，最多存储100条数据

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| user:{user\_id}:his:reading | zset |  | \[{article\_id, read\_time}\] |

在cache的user.py文件中定义实现阅读历史

```
import time
class UserReadingHistoryStorage(object):
    """
    用户阅读历史
    """
    def __init__(self, user_id):
        self.key = 'user:{}:his:reading'.format(user_id)
        self.user_id = user_id

    def save(self, article_id):
        """
        保存用户阅读历史
        :param article_id: 文章id
        :return:
        """
        try:
            pl = current_app.redis_store.pipeline()
            pl.zadd(self.key, {article_id:time.time()})
            pl.zremrangebyrank(self.key, 0, -1*(constants.READING_HISTORY_COUNT_PER_USER+1))
            pl.execute()
        except RedisError as e:
            current_app.logger.error(e)

    def get(self, page, per_page):
        """
        获取阅读历史
        """
        total_count = current_app.redis_store.zcard(self.key)

        article_ids = []
        if total_count > 0 and (page - 1) * per_page < total_count:

            article_ids = current_app.redis_store.zrevrange(self.key, (page - 1) * per_page, page * per_page - 1)

        return total_count, article_ids
```



