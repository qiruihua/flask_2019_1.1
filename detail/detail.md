# 详情页面

## 接口分析

**请求方式**：GET /app/v1\_0/articles/&lt;article\_id&gt;/

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| article\_id | str | 是 | 文章id |

**返回数据**： JSON

```
{
    "message": "ok",
    "data": {
        "art_id": 138154,
        "title": "PMBOK(第六版) PMP笔记——《九》第九章（项目资源管理）",
        "pubdate": "2018-11-29T17:16:16",
        "aut_id": 2,
        "aut_name": "18310820688",
        "aut_photo": "",
        "content": "xxxxxx"
        "is_followed": false,
        "attitude": -1,  # 不喜欢0 喜欢1 无态度-1  
        "is_collected": false
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| art\_id | str | 是 | 文章id |
| title | str | 是 | 文章标题 |
| pubdate | str | 是 | 发布日期 |
| aut\_id | str | 是 | 作者id |
| aut\_name | str | 是 | 作者name |
| content | str | 是 | 内容 |
| is\_followed | bool | 是 | 是否关注 |
| attitude | int | 是 | 0不喜欢 1喜欢 -1无态度 |
| is\_collected | bool | 是 | 是否喜欢 |

## 后端实现

```
from flask_restful import fields
from flask import current_app,abort
from sqlalchemy.orm import load_only
from cache.user import UserProfileCache

article_fields = {
    'art_id': fields.Integer(attribute='id'),
    'title': fields.String(attribute='title'),
    'pubdate': fields.DateTime(attribute='ctime', dt_format='iso8601'),
    'content': fields.String(attribute='content.content'),
    'aut_id': fields.Integer(attribute='user_id'),
    'ch_id': fields.Integer(attribute='channel_id'),
}


class DetailResource(Resource):

    def get(self,article_id):
        """
        1.根据文章id查询文章详情
        2.返回相应
        :param article_id:
        :return:
        """
        # 1.根据文章id查询文章详情
        article=None
        try:
            article = Article.query.filter_by(id=article_id, status=Article.STATUS.APPROVED).first()
        except Exception as e:
            current_app.logger.error(e)
            abort(404)

        article_dict = marshal(article, article_fields)

        #获取用户信息
        user = UserProfileCache(article_dict['aut_id']).get()
        #追加作者信息
        article_dict['aut_name'] = user['name']
        article_dict['aut_photo'] = user['photo']

        ## 判断是否关注
        is_followed=False
        # 判断是否喜欢
        attitude=1
        # 判断是否收藏
        is_collected = True

        article_dict['is_followed']=is_followed
        article_dict['attitude']=attitude
        article_dict['is_collected']=is_collected

        # 2.返回相应
        return article_dict
```

## 设置缓存

在common的cache包中创建article.py文件

```
import json
from flask_restful import fields, marshal
from flask import current_app
from redis import RedisError
from sqlalchemy.orm import load_only
from cache import constants
from cache.user import UserProfileCache
from models.news import Article

article_fields = {
    'art_id': fields.Integer(attribute='id'),
    'title': fields.String(attribute='title'),
    'pubdate': fields.DateTime(attribute='ctime', dt_format='iso8601'),
    'content': fields.String(attribute='content.content'),
    'aut_id': fields.Integer(attribute='user_id'),
    'ch_id': fields.Integer(attribute='channel_id'),
}

class ArticleDetailCache(object):
    """
    文章详细内容缓存
    """
    def __init__(self, article_id):
        self.key = 'art:{}:detail'.format(article_id)
        self.article_id = article_id

    def get(self):
        """
        获取文章详情信息
        :return:
        """
        # 查询文章数据
        try:
            article_bytes = current_app.redis_store.get(self.key)
        except RedisError as e:
            print(e)
            article_bytes = None

        if article_bytes:
            # 使用缓存
            article_dict = json.loads(article_bytes.decode())
        else:
            # 查询数据库
            article = Article.query.filter_by(id=self.article_id, status=Article.STATUS.APPROVED).first()

            article_dict = marshal(article, article_fields)

            # 缓存
            article_cache = json.dumps(article_dict)
            try:
                current_app.redis_store.setex(self.key, constants.ArticleDetailCacheTTL.get_val(), article_cache)
            except RedisError:
                pass

        user = UserProfileCache(article_dict['aut_id']).get()

        article_dict['aut_name'] = user['name']
        article_dict['aut_photo'] = user['photo']

        return article_dict
```



