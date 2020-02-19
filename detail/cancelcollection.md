# 取消收藏

## 接口分析

**请求方式**：DELETE /app/v1\_0/article/collections/&lt;target&gt;

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token\(**headers**\) | str | 是 | token |
| target\(**url**\) | str | 是 | 取消收藏的文章id |

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
class CollectionDeleteResource(Resource):

    method_decorators = [loginrequired]

    def delete(self, target):
        """
        用户取消收藏
        """
        Collection.query.filter_by(user_id=g.user_id, article_id=target, is_deleted=False) \
            .update({'is_deleted': True})
        db.session.commit()

        return {'message': 'OK'}, 204
```



