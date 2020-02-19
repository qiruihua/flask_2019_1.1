# 喜欢文章

## 接口分析

**请求方式**：POST /app/v1\_0/article/dislikes

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token**\(headers\)** | str | 是 | 用户token |
| target**\(body\)** | str | 是 | 不喜欢的文章id |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "target": 1
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| target | str | 是 | 不喜欢的文章id |

## 

## 后端实现

```
from toutiao.utils.decorators import loginrequired
from flask_restful.reqparse import RequestParser
from models.news import Attitude

class DislikeResource(Resource):
    """
    用户不喜欢
    """
    method_decorators = [loginrequired]

    def post(self):
        """
        不喜欢
        """
        json_parser = RequestParser()
        json_parser.add_argument('target', type=int, required=True, location='json')
        args = json_parser.parse_args()
        target = args.target

        atti = Attitude.query.filter_by(user_id=g.user_id, article_id=target).first()
        if atti is None:
            atti = Attitude(user_id=g.user_id, article_id=target, attitude=Attitude.ATTITUDE.DISLIKE)
        else:
            atti.attitude = Attitude.ATTITUDE.DISLIKE

        db.session.add(atti)
        db.session.commit()

        return {'target': target}, 201
```

## ![](/assets/不喜欢文章.png)



