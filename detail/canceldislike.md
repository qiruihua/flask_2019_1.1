# 取消点赞

## 接口分析

**请求方式**：DELETE /app/v1\_0/article/dislikes/&lt;target&gt;

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token\(**headers**\) | str | 是 | token |
| target\(**url**\) | str | 是 | 取消不喜欢的文章id |

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
class DisLikeDeleteResource(Resource):

    method_decorators = [loginrequired]

    def delete(self, target):
        """
        取消不喜欢
        """
        # 更新数据
        ret = Attitude.query.filter_by(user_id=g.user_id, article_id=target, attitude=Attitude.ATTITUDE.DISLIKE) \
            .update({'attitude': None})
        db.session.commit()
        return {'message': 'OK'}, 204
```



