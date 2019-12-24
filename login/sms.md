# 获取短信验证码

## 接口分析

**请求方式**：GET`/meiduo_admin/users/?keyword=<搜索内容>&page=<页码>&pagesize=<页容量>`

**请求参数**： 通过请求头传递jwt token数据。

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| keyword | str | 否 | 搜索用户名 |
| page | int | 否 | 页码 |
| pagesize | int | 否 | 页容量 |

**返回数据**： JSON

```
 {
        "count": "用户总量",
        "lists": [
            {
                "id": "用户id",
                "username": "用户名",
                "mobile": "手机号",
                "email": "邮箱"
            },
            ...
        ],
        "page": "页码",
        "pages": "总页数",
        "pagesize": "页容量"
      }

```

| 返回值 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| count | int | 是 | 用户总量 |
| Lists | 数组 | 是 | 用户信息 |
| page | int | 是 | 页码 |
| pages | int | 是 | 总页数 |
| pagesize | int | 是 | 页容量 |

## 后端实现



