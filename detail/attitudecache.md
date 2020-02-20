# 态度缓存实现

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



