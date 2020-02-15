# 关注用户列表

## 接口分析

**请求方式**：GET /app/v1_0/user/followings?page=xxx&per\_page=xxx_

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

        # 构造返回
        #查询结果
        followers = Relation.query.filter_by(user_id=g.user_id,
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

### 添加缓存

用户状态

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| user:{user\_id}:art:collection | zset | user\_id | \[{article\_id,update\_time}\] |

在common的cache包的user.py文件中添加缓存

```
#用户收藏缓存
from models.news import Collection


```



