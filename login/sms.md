# 获取短信验证码

## 接口分析

**请求方式**：GET /app/v1\_0/sms/codes/&lt;mobile&gt;/

**请求参数**： 

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| keyword | str | 否 | 搜索用户名 |
| page | int | 否 | 页码 |
| pagesize | int | 否 | 页容量 |

**返回数据**： JSON

```

```

| 返回值 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| count | int | 是 | 用户总量 |
| Lists | 数组 | 是 | 用户信息 |
| page | int | 是 | 页码 |
| pages | int | 是 | 总页数 |
| pagesize | int | 是 | 页容量 |

## 后端实现



