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
        """
        1.接收参数
        2.验证参数
        3.删除数据
        4.返回相应
        :return:
        """

        user_id=current_app.user_id
        try:
            relation=Relation.query.filter_by(user_id=user_id,target_user_id=target).first()
        except Exception as e:
            current_app.logger.error(e)
            abort(404)
        try:
            db.session.delete(relation)
            db.session.commit()
        except Exception as e:
            current_app.logger.error(e)


        return {"message": "OK"}
```



