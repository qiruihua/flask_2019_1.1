# 阅读历史列表

## 接口分析

**请求方式**：GET /app/v1\_0/article/collections

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |
| page | int | 否 | 页数，默认是1 |
| per\_page | int | 否 | 每页数量 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "page": 1,
        "per_page": 10, 
        "total_count": 100,  
        "results": [
            {
                "art_id": 108,
                "title":"Python如何学好"
                "aut_id": 1155989075455377414,
                "aut_name": "18310820688",
                "aut_photo": "",
                "pubdate": "2019-08-07T08:53:01",
                "cover": "封面",
                  "type": 0,
                  "images":[]
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
| results | list | 是 | 数据列表 |
| com\_id | int | 是 | 评论或回复id |
| aut\_id | int | 是 | 评论或回复用户id |
| aut\_name | str | 是 | 用户昵称 |
| aut\_photo | str | 是 | 用户头像 |
| pubdate | str | 是 | 创建时间 |
| content | str | 是 | 评论或回复内容 |
| is\_top | int | 是 | 是否置顶，0表示不置顶 1表示置顶 |
| is\_liking | bool | 是 | 当前用户是否点赞 |

## 后端实现

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

在constants.py文件中定义常量

```
# 阅读历史每人保存数目
READING_HISTORY_COUNT_PER_USER = 100
```

添加阅读历史

```
from cache.user import UserReadingHistoryStorage
if g.user_id:
    #是否收藏
    is_collected = UserArticleCollectionsCache(g.user_id).user_collect_target(article_id)
    # 是否关注
    is_followed = UserFollowingCache(g.user_id).user_follows_target(article_dict.get('aut_id'))
    #添加阅读历史
    UserReadingHistoryStorage(g.user_id).save(article_id)
```

展示阅读历史

```
from cache.user import UserReadingHistoryStorage
from cache.article import ArticleDetailCache

class ReadingHistoryResource(Resource):
    """
    用户阅读历史
    """
    method_decorators = [loginrequired]

    def get(self):
        """
        获取用户阅读历史
        """
        qs_parser = RequestParser()
        qs_parser.add_argument('page', type=inputs.positive, required=False, location='args')
        qs_parser.add_argument('per_page', type=inputs.int_range(constants.DEFAULT_ARTICLE_PER_PAGE_MIN,
                                                                 constants.DEFAULT_ARTICLE_PER_PAGE_MAX,
                                                                 'per_page'),
                               required=False, location='args')
        args = qs_parser.parse_args()
        page = 1 if args.page is None else args.page
        per_page = args.per_page if args.per_page else constants.DEFAULT_ARTICLE_PER_PAGE_MIN

        user_id = g.user_id

        results = []
        total_count, article_ids = UserReadingHistoryStorage(user_id).get(page, per_page)

        for article_id in article_ids:
            article = ArticleDetailCache(int(article_id)).get()
            results.append(article)

        return {'total_count': total_count, 'page': page, 'per_page': per_page, 'results': results}
```

![](/assets/历史记录.png)

