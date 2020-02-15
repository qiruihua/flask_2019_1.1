# 用户收藏列表

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

```
from cache.article import ArticleDetailCache
from flask import g
class CollectionListResource(Resource):

    method_decorators = [loginrequired]

    def get(self):
        """
        获取用户的收藏历史
        """
        # 参数检验
        qs_parser = RequestParser()
        qs_parser.add_argument('page', type=inputs.positive, required=False, location='args')
        qs_parser.add_argument('per_page', type=inputs.int_range(constants.DEFAULT_ARTICLE_PER_PAGE_MIN,
                                                                 constants.DEFAULT_ARTICLE_PER_PAGE_MAX,
                                                                 'per_page'),
                               required=False, location='args')
        args = qs_parser.parse_args()
        page = 1 if args.page is None else args.page
        per_page = args.per_page if args.per_page else constants.DEFAULT_ARTICLE_PER_PAGE_MIN

        # 构造返回
        collections = Collection.query.filter_by(user_id=g.user_id,
        is_deleted=False) \
        .order_by(Collection.utime.desc()).all()

        total_count = len(collections)
        #获取收藏的所有文章id 为缓存做准备
        collection_ids = []

        for collection in collections:
        collection_ids.append(collection.article_id)
        #获取指定页数的数据
        page_articles = collection_ids[(page - 1) * per_page:page * per_page]


        results = []
        for article_id in page_articles:
            article = ArticleDetailCache(article_id).get()
            results.append(article)

        return {'total_count': total_count, 'page': page, 'per_page': per_page, 'results': results}
```

### 添加缓存

用户收藏

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| user:{user\_id}:art:collection | zset | user\_id | \[{article\_id,update\_time}\] |

在common的cache包的user.py文件中添加缓存

```
#用户收藏缓存
from models.news import Collection

class UserArticleCollectionsCache(object):
    """
    用户收藏文章缓存
    """
    def __init__(self, user_id):
        self.user_id = user_id
        self.key = 'user:{}:art:collection'.format(user_id)

    def get_page(self, page, per_page):
        """
        获取用户的文章列表
        :param page: 页数
        :param per_page: 每页数量
        :return: total_count, [article_id, ..]
        """
        try:
            pl = current_app.redis_store.pipeline()
            pl.zcard(self.key)
            pl.zrevrange(self.key, (page - 1) * per_page, page * per_page)
            total_count, ret = pl.execute()
        except RedisError as e:
            current_app.logger.error(e)
            total_count = 0
            ret = []

        if total_count > 0:
            # Cache exists.
            return total_count, [int(aid) for aid in ret]
        else:
            # No cache.
            # 构造返回
            collections = Collection.query.filter_by(user_id=self.user_id,
                                                     is_deleted=False) \
                .order_by(Collection.utime.desc()).all()

            total_count = len(collections)
            # 获取收藏的所有文章id 为缓存做准备
            collection_ids = []
            cache = []
            for collection in collections:
                collection_ids.append(collection.article_id)
                # 缓存的数据结构
                cache.append({
                    collection.article_id:collection.utime.timestamp()
                })

            # 获取指定页数的数据
            page_articles = collection_ids[(page - 1) * per_page:page * per_page]

            if cache:
                #重新更新缓存数据
                try:
                    pl = current_app.redis_store.pipeline()
                    for item in cache:
                        pl.zadd(self.key, item)
                    pl.expire(self.key, constants.UserArticleCollectionsCacheTTL.get_val())
                    results = pl.execute()

                except RedisError as e:
                    current_app.logger.error(e)

            return total_count, page_articles
```

### 缓存常量设置

```
class UserArticleCollectionsCacheTTL(BaseCacheTTL):
    """
    用户文章收藏缓存时间，秒
    """
    TTL = 10 * 60
```

### 判断用户是否收藏文章

#### 添加判断方法

```
class UserArticleCollectionsCache(object):
    """
    用户收藏文章缓存
    """
    def __init__(self, user_id):
        self.user_id = user_id
        self.key = 'user:{}:art:collection'.format(user_id)


    def user_collect_target(self, target):
        """
        判断用户是否收藏了指定文章
        :param target:
        :return:
        """
        total_count, collections = self.get_page(1, -1)
        return target in collections
```

#### 修改详情页面

```
is_collected = False
from cache.user import UserArticleCollectionsCache
if g.user_id:
    is_collected=UserArticleCollectionsCache(g.user_id).user_collect_target(article_id)
```

### 添加收藏或取消收藏清除缓存数据

#### 添加清除缓存方法

```
class UserArticleCollectionsCache(object):
    """
    用户收藏文章缓存
    """
    def __init__(self, user_id):
        self.user_id = user_id
        self.key = 'user:{}:art:collection'.format(user_id)


    def clear(self):
        current_app.redis_store.delete(self.key)
```

### 添加收藏或取消收藏调用

```
# 删除关注缓存
from cache.user import UserArticleCollectionsCache
UserArticleCollectionsCache(g.user_id).clear()
```



