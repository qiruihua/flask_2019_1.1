# 用户信息

## 接口分析

**请求方式**：GET /app/v1\_0/user

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "id": 1155,
        "name": "18310820688",
        "photo": "xxxxx",
        "intro": "xxx",
        "art_count": 0,
        "follow_count": 0,
        "fans_count": 0
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
|  |  |  |  |

## 后端实现



