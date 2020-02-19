# 回复评论

## 接口分析

**请求方式**：POST /app/v1\_0/comments

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |
| target | str | 是 | 评论文章为文章id，对评论进行回复则是评论id |
| content | str | 是 | 评论内容 |
| art\_id | int | 否 | 对文章评论不需要传递此参数。对评论内容进行回复时，必须传递此参数 |

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
from toutiao.utils.decorators import loginrequired
from models.news import Comment

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
        user_id=g.user_id

        parse=reqparse.RequestParser()
        parse.add_argument('target',required=True)
        parse.add_argument('content',required=True)
        parse.add_argument('art_id', type=int, required=False, location='json')
        args=parse.parse_args()

        comment=Comment()
        comment.article_id=args.get('target')
        comment.content=args.get('content')
        comment.user_id=user_id
        #如果有回复id,则表示回复评论
        art_id=args.art_id
        if art_id:
            comment.article_id=art_id
            comment.parent_id=args.target

        try:
            db.session.add(comment)
            db.session.commit()
        except Exception:
            return {}

        return {
            "com_id": comment.id,
            "target": comment.article_id,
            "art_id": art_id
        }
```

![](/assets/回复评论.png)

