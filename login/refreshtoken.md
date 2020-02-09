# 刷新用户token

## 接口分析

**请求方式**：GET /app/v1\_0/authorizations

**请求参数（Headers）**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "token": "xxxx",
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| id | str | 是 | 用户id |
| name | str | 是 | 昵称 |
| photo | str | 是 | 头像 |
| intro | str | 是 | 简介 |
| art\_count | int | 是 | 文章数量 |
| follow\_count | int | 是 | 关注数量 |
| fans\_count | int | 是 | 粉丝数量 |

## 后端实现



