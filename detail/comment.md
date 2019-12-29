# 发布评论

## 接口分析

**请求方式**：POST /app/v1\_0/comments

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |
| target | str | 是 | 文章id |
| content | str | 是 | 评论内容 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "com_id": 108,
        "target": 138154
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| com\_id | str | 是 | 评论id |
| target | str | 是 | 文章id |

## 后端实现

```
from flask_restful import reqparse
from project.utils.user import loginrequired
class CommentsResource(Resource):
    method_decorators = {
        'post':[loginrequired]
    }

    def post(self):
        """
        1.接收数据
        2.验证数据
        3.数据入库
        4.返回相应
        :return:
        """
        user_id=current_app.user_id

        parse=reqparse.RequestParser()
        parse.add_argument('target',required=True)
        parse.add_argument('content',required=True)
        args=parse.parse_args()

        comment=Comment()
        comment.article_id=args.get('target')
        comment.content=args.get('content')
        comment.user_id=user_id
        try:
            db.session.add(comment)
            db.session.commit()
        except Exception:
            return {}

        return {
            "com_id": comment.id,
            "target": comment.article_id
        }
```



