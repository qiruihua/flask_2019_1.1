```
用户粉丝列表
```

## 接口分析

**请求方式**：GET /app/v1\_0/user/followers?_page=xxx&per\_page=xxx_

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户Token |
| page | str | 否 | 页码 |
| per\_page | str | 否 | 每页多少条数据 |

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
| mutual\_follow | bool | 否 | 是否互相关注 |

## 后端实现

```
class FansResource(Resource):

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
        followers = Relation.query.filter_by(target_user_id=g.user_id,
                       relation=Relation.RELATION.FOLLOW) \
            .order_by(Relation.utime.desc()).all()

        total_count = len(followers)

        #获取所有的关注人员id
        follower_ids = []
        for relation in followers:
            follower_ids.append(relation.user_id)
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

## 添加缓存

关注和粉丝的业务逻辑是相似的。

```
class UserFansCache(object):
    """
    用户粉丝缓存数据
    """

    def __init__(self, user_id):
        self.key = 'user:{}:fans'.format(user_id)
        self.user_id = user_id

    def update(self, target_user_id, timestamp, increment=1):
        """
        更新用户的粉丝缓存数据
        :param target_user_id: 被粉丝的目标用户
        :param timestamp: 关注时间戳
        :param increment: 增量 1表示增加 0表示删除
        :return:
        """

        try:
            if increment > 0:
                current_app.redis_store.zadd(self.key, {target_user_id: timestamp})
            else:
                current_app.redis_store.zrem(self.key, target_user_id)
        except RedisError as e:
            current_app.logger.error(e)

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

        ret = Relation.query.filter_by(target_user_id=self.user_id, relation=Relation.RELATION.FOLLOW) \
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

添加缓存时间常量

```
class UserFanssCacheTTL(BaseCacheTTL):
    """
    用户关注列表缓存时间，秒
    """
    TTL = 30 * 60
```

## 修改视图缓存实现

```
class FansResource(Resource):

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

        from cache.user import UserFansCache
        follower_ids=UserFansCache(g.user_id).get()
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



