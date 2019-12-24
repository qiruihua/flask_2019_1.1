## 接口分析

**请求方式**：GET /app/v1\_0/articles/&lt;article\_id&gt;/

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| article\_id | str | 是 | 文章id |

**返回数据**： JSON

```
{
    "message": "ok",
    "data": {
        "art_id": 138154,
        "title": "PMBOK(第六版) PMP笔记——《九》第九章（项目资源管理）",
        "pubdate": "2018-11-29T17:16:16",
        "aut_id": 2,
        "aut_name": "18310820688",
        "aut_photo": "",
        "content": "xxxxxx"
        "is_followed": false,
        "attitude": -1,  # 不喜欢0 喜欢1 无态度-1  
        "is_collected": false
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| art\_id | str | 是 | 文章id |
| title | str | 是 | 文章标题 |
| pubdate | str | 是 | 发布日期 |
| aut\_id | str | 是 | 作者id |
| aut\_name | str | 是 | 作者name |
| content | str | 是 | 内容 |
| is\_followed | bool | 是 | 是否关注 |
| attitude | int | 是 | 0不喜欢 1喜欢 -1无态度 |
| is\_collected | bool | 是 | 是否喜欢 |

## 后端实现



