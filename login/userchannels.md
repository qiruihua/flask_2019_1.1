## 接口分析

**请求方式**：GET /app/v1\_0/user/channels

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "channels": [
            {
                "id": 0,
                "name": "推荐"
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
| channels | list | 是 | 频道列表 |
| id | dict | 是 | 频道id |
| name | dict | 是 | 频道名字 |

## 后端实现



