# 生成用户id

## 添加**id\_worker**文件

## 配置

然后在中创建**id生成器**\(**app/**`__init__`**.py**\)：

```
# 创建Snowflake ID worker
from utils.id_worker import IdWorker
app.id_worker = IdWorker(app.config['DATACENTER_ID'],
                         app.config['WORKER_ID'],
                         app.config['SEQUENCE'])
```

然后在配置文件中增加相关配置：

```
    # Snowflake ID Worker 参数
    DATACENTER_ID = 0
    WORKER_ID = 0
    SEQUENCE = 0
```

## 使用



