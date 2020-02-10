# 首页文章列表

## 接口分析

**请求方式**：GET /app/v1_0/articles?channel\_id=xxx&page=xxx&per\_page=xxx_

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
| per\_page | int | 否 | 文章数 |
| results | dict | 否 | 文章列表 |
| art\_id | str | 否 | 文章id |
| title | str | 否 | 文章标题 |
| aut\_id | str | 否 | 作者id |
| pubdate | str | 否 | 发布日期 |
| aut\_name | str | 否 | 作者名字 |
| comm\_count | int | 否 | 评论数量 |

## 后端实现

```
class IndexResource(Resource):

    def get(self):
        """
        1.获取参数
        2.根据参数查询数据
        3.分页处理
        4.将对象列表数据转换为字典，返回相应
        :return:
        """
        # 1.获取参数
        channel_id=request.args.get('channel_id',0)
        page=request.args.get('page',1)
        per_page=request.args.get('per_page',10)

        try:
            page=int(page)
            per_page=int(per_page)
        except Exception:
            page=1
            per_page=10

        if int(channel_id) == 0:
            channel_id=1
        # 2.根据参数查询数据
        # 3.分页处理
        page_articles=Article.query.filter_by(channel_id=channel_id,
                                   status=Article.STATUS.APPROVED).paginate(page=page,
                                                                            per_page=per_page)

        # 4.将对象列表数据转换为字典，返回相应
        results = []
        for item in page_articles.items:
            results.append({
                "art_id": item.id,
                "title": item.title,
                "aut_id": item.user.id,
                "pubdate": item.ctime.strftime('%Y-%m-%d %H:%M:%S'),
                "aut_name": item.user.name,
                "comm_count": item.comment_count
            })

        return {'per_page': per_page, 'results': results}
```



