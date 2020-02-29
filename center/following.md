# 关注用户列表

## 接口分析

**请求方式**：GET /app/v1_0/user/followings?page=xxx&per\_page=xxx_

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token\(**header**\) | str | 是 | 用户Token |
| page\(**query**\) | str | 否 | 页码 |
| per\_page\(**query**\) | str | 否 | 每页多少条数据 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "total_count": 1,
        "page": 1,
        "per_page": 20,
        "results": [
            {
                "id": 1,
                "name": "黑马头条号",
                "photo": "Fph9hHDwU9UzMTUAqDwd6bFGaWbg",
                "fans_count": 1,
                "mutual_follow": false  # 是否为互相关注
            }
        ]
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| total\_count | int | 是 | 总记录数 |
| page | int | 是 | 当前页码 |
| per\_page | int | 是 | 每页多少条记录 |
| results | list | 是 | 记录数 |
| id | str | 否 | id |
| name | str | 否 | 名字 |
| photo | str | 否 | 头像路由 |
| fans\_count | int | 否 | 粉丝数量 |
| mutual\_follow | bool | 否 | 是否互相关注 |

## 后端实现

```
class FollowResource(Resource):

    method_decorators = [loginrequired]

    def get(self):
        """
        获取关注的用户列表
        """
        # 参数验证
        qs_parser = RequestParser()
        qs_parser.add_argument('page', type=inputs.positive, required=False, location='args')
        qs_parser.add_argument('per_page', type=inputs.int_range(constants.DEFAULT_USER_FOLLOWINGS_PER_PAGE_MIN,
                                                                 constants.DEFAULT_USER_FOLLOWINGS_PER_PAGE_MAX,
                                                                 'per_page'),
                               required=False, location='args')
        args = qs_parser.parse_args()
        page = 1 if args.page is None else args.page
        per_page = args.per_page if args.per_page else constants.DEFAULT_USER_FOLLOWINGS_PER_PAGE_MIN


        #查询结果
        followers = Relation.query.filter_by(user_id=g.user_id,
                       relation=Relation.RELATION.FOLLOW) \
            .order_by(Relation.utime.desc()).all()

        total_count = len(followers)

        #获取所有的关注人员id
        follower_ids = []
        for relation in followers:
            follower_ids.append(relation.target_user_id)
        #分页
        page_followings = follower_ids[(page - 1) * per_page:page * per_page]
        #遍历获取缓存信息
        results = []
        for following_user_id in page_followings:
            user = UserProfileCache(following_user_id).get()
            results.append({
                'id':id,
                'name':user.get('name'),
                'photo':user.get('photo'),
                'mutual_follow':False
            })

        return {'total_count': total_count, 'page': page, 'per_page': per_page, 'results': results}
```

在home蓝图的constants中添加常量

```
# 用户关注列表每页数据量
DEFAULT_USER_FOLLOWINGS_PER_PAGE_MIN = 10
# 用户关注列表每页数据量
DEFAULT_USER_FOLLOWINGS_PER_PAGE_MAX = 50
```

![](/assets/关注列表.png)

## 添加缓存

用户关注

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| user:{user\_id}:collection | zset | user\_id | \[{article\_id:update\_time}\] |

在common的cache包的user.py文件中添加缓存

```
from models.user import Relation
class UserFollowingCache(object):
    """
    用户关注缓存数据
    """
    def __init__(self, user_id):
        self.key = 'user:{}:following'.format(user_id)
        self.user_id = user_id

    def get(self):
        """
        获取用户的关注列表
        :return:
        """
        # 查询到缓存，则直接构造返回
        try:
            ret = current_app.redis_store.zrevrange(self.key, 0, -1)
        except RedisError as e:
            print(e)
            ret = None

        if ret:
            # 为了与数据库类型保持一致
            return [int(uid) for uid in ret]

        # 查询不到缓存，则从数据库中查询并构造返回

        ret = Relation.query.filter_by(user_id=self.user_id, relation=Relation.RELATION.FOLLOW) \
            .order_by(Relation.utime.desc()).all()

        followings = []
        cache = []
        for relation in ret:
            followings.append(relation.target_user_id)

            cache.append({
                relation.target_user_id: relation.utime.timestamp()
            })
        # 将数据存入缓存
        if cache:
            try:
                pl = current_app.redis_store.pipeline()
                for item in cache:
                    pl.zadd(self.key, item)
                pl.expire(self.key, constants.UserFollowingsCacheTTL.get_val())
                pl.execute()
            except RedisError as e:
                current_app.logger.error(e)

        return followings
```

缓存常量设置

```
class UserFollowingsCacheTTL(BaseCacheTTL):
    """
    用户关注列表缓存时间，秒
    """
    TTL = 30 * 60
```

## 修改视图使用缓存

```
class FollowResource(Resource):

    method_decorators = [loginrequired]

    def get(self):
        """
        获取关注的用户列表
        """
        # 参数验证
        qs_parser = RequestParser()
        qs_parser.add_argument('page', type=inputs.positive, required=False, location='args')
        qs_parser.add_argument('per_page', type=inputs.int_range(constants.DEFAULT_USER_FOLLOWINGS_PER_PAGE_MIN,
                                                                 constants.DEFAULT_USER_FOLLOWINGS_PER_PAGE_MAX,
                                                                 'per_page'),
                               required=False, location='args')
        args = qs_parser.parse_args()
        page = 1 if args.page is None else args.page
        per_page = args.per_page if args.per_page else constants.DEFAULT_USER_FOLLOWINGS_PER_PAGE_MIN


        #查询结果
        from cache.user import UserFollowingCache
        follower_ids = UserFollowingCache(g.user_id).get()
        total_count = len(follower_ids)
        #分页
        page_followings = follower_ids[(page - 1) * per_page:page * per_page]
        #遍历获取缓存信息
        results = []
        for following_user_id in page_followings:
            user = UserProfileCache(following_user_id).get()
            results.append(dict(
                id=following_user_id,
                name=user['name'],
                photo=user['photo'],
                mutual_follow=False
            ))

        return {'total_count': total_count, 'page': page, 'per_page': per_page, 'results': results}
```

## 判断用户是否关注作者

添加判断方法

```
class UserFollowingCache(object):
    """
    用户关注缓存数据
    """
    def __init__(self, user_id):
        self.key = 'user:{}:following'.format(user_id)
        self.user_id = user_id


    def user_follows_target(self, target_user_id):
        """
        判断用户是否关注了目标用户
        :param target_user_id: 被关注的用户id
        :return:
        """
        followings = self.get()

        return int(target_user_id) in followings
```

修改详情页面

```
from cache.user import UserFollowingCache
from cache.user import UserArticleCollectionsCache

if g.user_id:
    #是否收藏
    is_collected=UserArticleCollectionsCache(g.user_id).user_collect_target(article_id)
    #是否关注
    is_followed=UserFollowingCache(g.user_id).user_follows_target(article_dict.get('aut_id'))
```

### 



