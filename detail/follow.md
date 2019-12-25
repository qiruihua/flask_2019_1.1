## 接口分析

**请求方式**：POST /app/v1\_0/user/followings

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |
| targe | str | 是 | 关注用户id |

**返回数据**： JSON

```
{
  "message": "ok",
  "data": {
    "token": "xxxxxxxx"
  }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| token | str | 是 | Token |

## 后端实现



