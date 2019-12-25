## 接口分析

**请求方式**：GET /app/v1_0/user/followings?page=xxx&per\_page=xxx_

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户Token |
| code | str | 是 | 验证码 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "total_count": 1,
        "page": 1,
        "per_page": 20,
        "results": [
            {
                "id": 1,
                "name": "黑马头条号",
                "photo": "Fph9hHDwU9UzMTUAqDwd6bFGaWbg",
                "fans_count": 1,
                "mutual_follow": false  # 是否为互相关注
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



