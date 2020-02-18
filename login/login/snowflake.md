# 生成用户id

## **雪花算法-Snowflake**

Snowflake是Twitter提出来的一个算法，其目的是生成一个64bit的整数:

![](/assets/snowflake原理.png)

* 1bit:一般是符号位，不做处理
* 41bit:用来记录时间戳，这里可以记录69年，如果设置好起始时间比如今年是2018年，那么可以用到2089年，到时候怎么办？要是这个系统能用69年，我相信这个系统早都重构了好多次了。
* 10bit:10bit用来记录机器ID，总共可以记录1024台机器，一般用前5位代表数据中心，后面5位是某个数据中心的机器ID
* 12bit:循环位，用来对同一个毫秒之内产生不同的ID，12位可以最多记录4095个，也就是在同一个机器同一毫秒最多记录4095个，多余的需要进行等待下毫秒。

## 添加**id\_worker**文件

将课件中的id\_worker.py文件拷贝到common目录下新建的snowflake中

![](/assets/snowflake.png)

> 测试运行id\_worker文件

## 配置

然后在toutiao目录的`__init__`.py中创建id生成器

```
from snowflake.id_worker import IdWorker
# 创建Snowflake ID worker
app.id_worker = IdWorker(Config.DATACENTER_ID,
                         Config.WORKER_ID,
                        Config.SEQUENCE)
```

然后在settings.py配置文件中增加相关配置：

```
    # Snowflake ID Worker 参数
    DATACENTER_ID = 0
    WORKER_ID = 0
    SEQUENCE = 0
```

## 使用

指定用户id，不让系统自己生成

```
if user is None:
    user = User()
    user_id = current_app.id_worker.get_id()
    user.id=user_id
    user.mobile = args.get('mobile')
    user.name = args.get('mobile')
    db.session.add(user)
    db.session.commit()
```



