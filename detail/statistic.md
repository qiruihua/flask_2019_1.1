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

```

## 在视图中调用实现

