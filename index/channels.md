## 接口分析

**请求方式**：GET /app/v1\_0/channels

**请求参数**：无

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "channels": [
            {
                "id": 11,
                "name": "html"
            },
            {
                "id": 7,
                "name": "数据库"
            }
        ]
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
|  |  |  |  |

## 后端实现



