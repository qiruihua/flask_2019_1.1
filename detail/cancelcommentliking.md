# 取消点赞

## 接口分析

**请求方式**：DELETE /app/v1\_0/comment/likings/&lt;target&gt;/

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
class CommentLikingDeleteResource(Resource):
    """
    评论点赞
    """
    method_decorators = [loginrequired]

    def delete(self, target):
        """
        取消对评论点赞
        """
        ret = CommentLiking.query.filter_by(user_id=g.user_id, comment_id=target, is_deleted=False) \
            .update({'is_deleted': True})
        db.session.commit()
        return {'message': 'OK'}, 204
```



