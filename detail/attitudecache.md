# 态度缓存实现

###  {#2-搜索历史}

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| user:{user\_id}:art:attitude | hash | 用户的所有态度数据 | {art\_id:attitude} |

在cache包中的user.py文件添加实现代码

```
from models.news import Attitude
class UserArticleAttitudeCache(object):
    """
    用户文章态度缓存数据
    """

    def __init__(self, user_id):
        self.key = 'user:{}:art:attitude'.format(user_id)
        self.user_id = user_id

    def get_all(self):
        """
        获取用户文章态度数据
        :return:
        """
        try:
            ret = current_app.redis_store.hgetall(self.key)
        except RedisError as e:
            current_app.logger.error(e)
            ret = None

        if ret:
            return {int(aid): int(attitude) for aid, attitude in ret.items()}

        ret = Attitude.query.filter(Attitude.user_id == self.user_id,
                                    Attitude.attitude != None).all()

        attitudes = {}
        for atti in ret:
            attitudes[atti.article_id] = atti.attitude

        pl = current_app.redis_store.pipeline()
        try:
            if attitudes:
                pl.hmset(self.key, attitudes)
                pl.expire(self.key, constants.UserArticleAttitudeCacheTTL.get_val())
            else:
                pl.hmset(self.key, {-1: -1})
                pl.expire(self.key, constants.UserArticleAttitudeNotExistsCacheTTL.get_val())
            results = pl.execute()
            if results[0] and not results[1]:
                current_app.redis_store.delete(self.key)
        except RedisError as e:
            current_app.logger.error(e)

        return attitudes

    def get_article_attitude(self, article_id):
        """
        获取指定文章态度
        :param article_id:
        :return:
        """

        attitudes = self.get_all()

        return attitudes.get(int(article_id), -1)

    def user_liking_article(self, article_id):
        """
        判断是否对文章点赞
        :param article_id:
        :return:
        """
        return self.get_article_attitude(article_id) == Attitude.ATTITUDE.LIKING

    def clear(self):
        """
        清除
        """
        current_app.redis_store.delete(self.key)
```

定义缓存时间常量

```
class UserArticleAttitudeCacheTTL(BaseCacheTTL):
    """
    用户文章态度缓存时间，秒
    """
    TTL = 30 * 60


class UserArticleAttitudeNotExistsCacheTTL(BaseCacheTTL):
    """
    用户文章态度不存在数据缓存时间，秒
    """
    TTL = 5 * 60
```

## 修改视图态度判断

```
         # 判断是否关注
        is_followed=False
        # 判断是否喜欢
        attitude=None
        # 判断是否收藏
        is_collected = False

        if g.user_id:
            # 查询登录用户对文章的态度（点赞or不喜欢）
            from cache.user import UserArticleAttitudeCache
            try:
                attitude = UserArticleAttitudeCache(g.user_id).get_article_attitude(article_id)
            except Exception as e:
                current_app.logger.error(e)
                attitude = -1
        article_dict['is_followed']=is_followed
        article_dict['attitude']=attitude
        article_dict['is_collected']=is_collected
```



