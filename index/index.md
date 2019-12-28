# 首页文章

## 接口分析

**请求方式**：GET /app/v1_0/articles?channel\_id=xxx&page=xxx&per\_page_

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 否 | 用户token |
| channel\_id | str | 是 | 频道id |
| page | int | 否 | 页码 |
| per\_page | int | 否 | 每页条数 |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "pre_page": 2,
        "results": [
            {
                "art_id": 140901,
                "title": "机器学习技术前沿与未来展望",
                "aut_id": 1,
                "pubdate": "2018-11-29T17:18:33",
                "aut_name": "黑马头条号",
                "comm_count": 0
            },
            {
                "art_id": 139940,
                "title": "Enscape 2.4新功能预览",
                "aut_id": 1,
                "pubdate": "2018-11-29T17:17:46",
                "aut_name": "黑马头条号",
                "comm_count": 0
            }
        ]
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| per\_page | int | 否 |  |
| results | dict |  |  |
| art\_id |  |  |  |
| title |  |  |  |
| aut\_id |  |  |  |
| pubdate |  |  |  |
| aut\_name |  |  |  |
| comm\_count |  |  |  |

## 后端实现



