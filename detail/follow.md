# 关注用户

## 接口分析

**请求方式**：POST /app/v1\_0/user/followings

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| token | str | 是 | 用户token |
| targe | str | 是 | 关注用户id |

**返回数据**： JSON

```
{
    "message": "OK",
    "data": {
        "target": 1
    }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| target | str | 是 | 关注id |

## 模型类

```
class Relation(db.Model):
    """
    用户关系表
    """
    __tablename__ = 'user_relation'

    class RELATION:
        DELETE = 0
        FOLLOW = 1
        BLACKLIST = 2

    id = db.Column('relation_id', db.Integer, primary_key=True, doc='主键ID')
    user_id = db.Column(db.Integer, db.ForeignKey('user_basic.user_id'), db.ForeignKey('user_profile.user_id'), doc='用户ID')
    target_user_id = db.Column(db.Integer, db.ForeignKey('user_basic.user_id'), doc='目标用户ID')
    relation = db.Column(db.Integer, doc='关系')
    ctime = db.Column('create_time', db.DateTime, default=datetime.now, doc='创建时间')
    utime = db.Column('update_time', db.DateTime, default=datetime.now, onupdate=datetime.now, doc='更新时间')
```

## 后端实现

```
from models.user import Relation
class FollowResource(Resource):

    method_decorators = [loginrequired]

    def post(self):
        """
        1.接收参数
        2.验证参数
        3.数据入库
        4.返回相应
        :return:
        """
        user_id=g.user_id
        # 1.接收参数
        # 2.验证参数
        parse=reqparse.RequestParser()
        parse.add_argument('target',location='json',required=True)
        args=parse.parse_args()
        user=None
        try:
            user=User.query.get(args.get('target'))
        except Exception as e:
            current_app.logger.error(e)

        if user is None:
            abort(404)
        # 3.数据入库
        relation=Relation()
        relation.user_id=user_id
        relation.target_user_id=args.get('target')
        relation.relation=Relation.RELATION.FOLLOW
        try:
            db.session.add(relation)
            db.session.commit()
        except Exception as e:
            current_app.logger.error(e)
            return {'message':'error','data':{}}
        # 4.返回相应
        return {'target':args.get('target')}
```

## 缓存关注信息

| key | 类型 | 说明 | 举例 |
| :--- | :--- | :--- | :--- |
| user:{user\_id}:following | zset | user\_id的关注用户 | - |

我们需要更新用户关注的id

我们可以在common的cache包中，在user.py文件中定义关注缓存类

```
class UserFollowingCache(object):
    """
    用户关注缓存数据
    """
    def __init__(self, user_id):
        self.key = 'user:{}:following'.format(user_id)
        self.user_id = user_id

    def update(self, target_user_id, timestamp, increment=1):
        """
        更新用户的关注缓存数据
        :param target_user_id: 被关注的目标用户
        :param timestamp: 关注时间戳
        :param increment: 增量 1表示增加 0表示删除
        :return:
        """

        try:
            if increment > 0:
                current_app.redis_store.zadd(self.key, {timestamp:target_user_id})
            else:
                current_app.redis_store.zrem(self.key, target_user_id)
        except RedisError as e:
            current_app.logger.error(e)
```

修改视图调用缓存类

```
 #添加缓存
 from cache.user import UserFollowingCache
 import time
 UserFollowingCache(user_id).update(args.get('target'),time.time())
```



