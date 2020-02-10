# 用户收藏列表

## 接口分析

**请求方式**：GET /app/v1\_0/article/collections

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |
| page | int | 否 | 页数，默认是1 |
| per\_page | int | 否 | 每页数量 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "page": 1,
        "per_page": 10, 
        "total_count": 100,  
        "results": [
            {
                "art_id": 108,
                "title":"Python如何学好"
                "aut_id": 1155989075455377414,
                "aut_name": "18310820688",
                "aut_photo": "",
                "pubdate": "2019-08-07T08:53:01",
                "cover": "封面",
                  "type": 0,
                  "images":[]
                  "is_liking": 0
            }
        ]
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| total\_count | int | 是 | 评论总数或评论回复总数 |
| results | list | 是 | 数据列表 |
| com\_id | int | 是 | 评论或回复id |
| aut\_id | int | 是 | 评论或回复用户id |
| aut\_name | str | 是 | 用户昵称 |
| aut\_photo | str | 是 | 用户头像 |
| pubdate | str | 是 | 创建时间 |
| content | str | 是 | 评论或回复内容 |
| is\_top | int | 是 | 是否置顶，0表示不置顶 1表示置顶 |
| is\_liking | bool | 是 | 当前用户是否点赞 |

## 后端实现

```
from project.apps.home import constants
from flask_restful.inputs import positive,int_range
```



