# 发布评论

## 接口分析

**请求方式**：POST /app/v1\_0/comments

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |
| target | str | 是 | 文章id |
| content | str | 是 | 评论内容 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "com_id": 108,
        "target": 138154
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| com\_id | str | 是 | 评论id |
| target | str | 是 | 文章id |

## 后端实现



