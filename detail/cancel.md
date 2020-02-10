# 取消关注用户

## 接口分析

**请求方式**：DELETE /app/v1\_0/user/followings/&lt;target&gt;/

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | token |
| target | str | 是 | 取消关注用户id |

**返回数据**： JSON

```
{
    "message": "OK"
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 否 | 数据 |

## 后端实现

```
class FollowDeleteResource(Resource):

    method_decorators = [loginrequired]

    def delete(self,target):

        Relation.query.filter(Relation.user_id == g.user_id,
                                    Relation.target_user_id == target,
                                    Relation.relation == Relation.RELATION.FOLLOW)\
            .update({'relation': Relation.RELATION.DELETE})
        db.session.commit()


        return {"message": "OK"}
```

## 更新缓存数据

```

```

## 完善详情页面是否关注的功能

```

```



