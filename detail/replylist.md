## 接口分析

**请求方式**：GET /app/v1\_0/comments

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| mobile | str | 是 | 手机号 |
| code | str | 是 | 验证码 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "total_count": 1,
        "end_id": 108,  # 最后一条评论的id, offset < end_id时才会发请求获取更多评论, 无值返回None
        "last_id": 108,  # 本次请求最后一条评论的id, 作为下次请求的offset, 无值返回None
        "results": [
            {
                "com_id": 108,
                "aut_id": 1155989075455377414,
                "aut_name": "18310820688",
                "aut_photo": "",
                "pubdate": "2019-08-07T08:53:01",
                "content": "你写的真好"
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



