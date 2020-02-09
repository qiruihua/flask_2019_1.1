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
from project.models.user import Relation
class FollowResource(Resource):

    method_decorators = [loginrequired]

    def get(self):
        """
        1.获取用户id
        2.查询关注列表
        3.将对象列表转换为字典，返回相应
        :return:
        """
        page = request.args.get('page', 1)
        per_page = request.args.get('per_page', 10)

        try:
            page = int(page)
            per_page = int(per_page)
        except Exception:
            page = 1
            per_page = 10
        # 1.获取用户id
        user_id=current_app.user_id
        # 2.查询关注列表
        page_relations=Relation.query.filter_by(user_id=user_id,
                                          relation=Relation.RELATION.FOLLOW).paginate(page=page,
                                                                                       per_page=per_page)
        results = []
        for item in page_relations.items:
            user=User.query.get(item.target_user_id)

            #相互关注判断
            mutual_follow=False
            for rel in user.followings:
                if rel.target_user_id == user_id:
                    mutual_follow=True
                    break

            results.append( {
                    "id": item.id,
                    "name": user.name,
                    "photo": user.profile_photo,
                    "fans_count": user.fans_count,
                    "mutual_follow": mutual_follow  # 是否为互相关注
                })
        # 3.将对象列表转换为字典，返回相应

        return {
            "total_count": page_relations.total,
            "page": page,
            "per_page": per_page,
            "results": results
        }
```



