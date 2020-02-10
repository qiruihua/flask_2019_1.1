# 点赞文章

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
| token | str | 是 | Token |

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
from project.models.user import Relation
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
        user_id=current_app.user_id
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



