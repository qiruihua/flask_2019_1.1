# 取消点赞

## 接口分析

**请求方式**：DELETE /app/v1\_0/user/likings/&lt;target&gt;/

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | token |
| target | str | 是 | 取消喜欢的文章id |

**返回数据**： JSON

```
{
    "message": "OK"
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |

## 后端实现

```
class ArticleLikeDeleteResource(Resource):
    """
    文章点赞取消
    """
    method_decorators = [loginrequired]

    def delete(self, target):
        """
        取消文章点赞
        """
        # 修改数据
        ret = Attitude.query.filter_by(user_id=g.user_id, article_id=target, attitude=Attitude.ATTITUDE.LIKING) \
            .update({'attitude': 0})
        db.session.commit()

        return {'message': 'OK'}, 204
```

## 更新缓存数据

```

```

## 完善详情页面是否关注的功能

```

```



