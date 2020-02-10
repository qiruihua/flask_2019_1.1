# 评论列表

## 接口分析

**请求方式**：GET /app/v1\_0/comments?source=x&offset=xxx&limit=xxx

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| type | str | 是 | 评论类型，a表示文章评论 c表示回复评论 |
| source | int | 是 | 文章id或者评论id |
| offset | int | 否 | 获取评论数据的偏移量 |
| limit | int | 否 | 评论条数 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "total_count": 1,
        "end_id": 108,  # 最后一条评论的id, 前端用于判断是否剩余评论, 无值返回None
        "last_id": 108,  # 本次请求最后一条评论的id, 作为下次请求的offset, 无值返回None
        "results": [
            {
                "com_id": 108,
                "aut_id": 1155989075455377414,
                "aut_name": "18310820688",
                "aut_photo": "",
                "pubdate": "2019-08-07T08:53:01",
                "content": "你写的真好",
                          "reply_count": 0,
                          "like_count": 0
            }
        ]
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| token | str | 是 | Token |

## 后端实现



