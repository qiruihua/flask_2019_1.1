# 用户粉丝列表

## 接口分析

**请求方式**：GET /app/v1\_0/followers?_page=xxx&per\_page=xxx_

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户Token |
| page | str | 否 | 页码 |
| per\_page | str | 否 | 每页多少条数据 |

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
| total\_count | int | 是 | 总记录数 |
| page | int | 是 | 当前页码 |
| per\_page | int | 是 | 每页多少条记录 |
| results | list | 是 | 记录数 |
| id | str | 否 | id |
| name | str | 否 | 名字 |
| photo | str | 否 | 头像路由 |
| fans\_count | int | 否 | 粉丝数量 |
| mutual\_follow | bool | 否 | 是否互相关注 |

## 后端实现

```
from project.apps.home import constants
from flask_restful.inputs import positive,int_range
```



