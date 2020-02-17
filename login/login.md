# 登录注册

## 接口分析

**请求方式**：POST /app/v1\_0/authorizations

**请求参数**：

| 参数 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| mobile | str | 是 | 手机号 |
| code | str | 是 | 验证码 |

**返回数据**： JSON

```
{
  "message": "ok",
  "data": {
    "token": "xxxxxxxx"
  }
}
```

| 返回数据 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| message | str | 是 | 消息内容 |
| data | dict | 是 | 数据 |
| token | str | 是 | 用户token |

## 模型类

```
from datetime import datetime
from toutiao import db


class User(db.Model):
    """
    用户基本信息
    """
    __tablename__ = 'user_basic'

    class STATUS:
        ENABLE = 1
        DISABLE = 0

    id = db.Column('user_id', db.Integer, primary_key=True, doc='用户ID')
    mobile = db.Column(db.String, doc='手机号')
    password = db.Column(db.String, doc='密码')
    name = db.Column('user_name', db.String, doc='昵称')
    profile_photo = db.Column(db.String, doc='头像')
    last_login = db.Column(db.DateTime, doc='最后登录时间')
    is_media = db.Column(db.Boolean, default=False, doc='是否是自媒体')
    is_verified = db.Column(db.Boolean, default=False, doc='是否实名认证')
    introduction = db.Column(db.String, doc='简介')
    certificate = db.Column(db.String, doc='认证')
    article_count = db.Column(db.Integer, default=0, doc='发帖数')
    following_count = db.Column(db.Integer, default=0, doc='关注的人数')
    fans_count = db.Column(db.Integer, default=0, doc='被关注的人数（粉丝数）')
    like_count = db.Column(db.Integer, default=0, doc='累计点赞人数')
    read_count = db.Column(db.Integer, default=0, doc='累计阅读人数')

    account = db.Column(db.String, doc='账号')
    email = db.Column(db.String, doc='邮箱')
    status = db.Column(db.Integer, default=1, doc='状态，是否可用')

    # 两种方法都可以
    # followings = db.relationship('Relation', primaryjoin='User.id==Relation.user_id')
    followings = db.relationship('Relation', foreign_keys='Relation.user_id')


class UserProfile(db.Model):
    """
    用户资料表
    """
    __tablename__ = 'user_profile'

    class GENDER:
        MALE = 0
        FEMALE = 1

    id = db.Column('user_id', db.Integer, primary_key=True, doc='用户ID')
    gender = db.Column(db.Integer, default=0, doc='性别')
    birthday = db.Column(db.Date, doc='生日')
    real_name = db.Column(db.String, doc='真实姓名')
    id_number = db.Column(db.String, doc='身份证号')
    id_card_front = db.Column(db.String, doc='身份证正面')
    id_card_back = db.Column(db.String, doc='身份证背面')
    id_card_handheld = db.Column(db.String, doc='手持身份证')
    ctime = db.Column('create_time', db.DateTime, default=datetime.now, doc='创建时间')
    utime = db.Column('update_time', db.DateTime, default=datetime.now, onupdate=datetime.now, doc='更新时间')
    register_media_time = db.Column(db.DateTime, doc='注册自媒体时间')

    area = db.Column(db.String, doc='地区')
    company = db.Column(db.String, doc='公司')
    career = db.Column(db.String, doc='职业')

    followings = db.relationship('Relation', foreign_keys='Relation.user_id')
```

## 后端实现

### 登录视图逻辑实现

```
from flask import request
from flask_restful import reqparse
from flask_restful.inputs import regex
from models.user import User
from toutiao import db
from toutiao.utils.parsers import check_mobile
from toutiao.utils.token import get_user_token

class LoginResource(Resource):

    def post(self):

        """
        1.接收数据
        2.验证数据
        3.根据手机号进行数据库查询，如果不存在则保存数据
        4.生成token
        5.返回相应
        """
        # 1.接收数据
        data = request.json
        # 2.验证数据
        parser = reqparse.RequestParser()
        parser.add_argument('mobile', type=check_mobile, location='json', required=True, )
        parser.add_argument('code', type=regex(r'\d{6}'), location='json', required=True)
        args = parser.parse_args()
        mobile=args.get('mobile')
        sms_code=args.get('code')

        # 验证短信码省略
        # 从redis中获取验证码
        key = 'app:code:{}'.format(mobile)
        redis_code = current_app.redis_store.get(key)

        if not redis_code or redis_code.decode() != sms_code:
            return {'message': 'Invalid code.'}, 400

        # 3.根据手机号进行数据库查询，如果不存在则保存数据
        user = None
        try:
            user = User.query.filter_by(mobile=args.get('mobile')).first()
        except Exception as e:
            current_app.logger.error(e)

        if user is None:
            user = User()
            user.mobile = args.get('mobile')
            user.name = args.get('mobile')
            db.session.add(user)
            db.session.commit()
        else:
            from datetime import datetime
            user.last_login = datetime.now()
            db.session.commit()
        # 4.生成token
        token = get_user_token(user.id)
        # 5.返回相应
        return {'token': token}
```

### 验证手机号格式

在toutiao目录的utlis下创建parsers

![](/assets/toutiao_utils_parsers.png)

定义验证手机号方法

```
import re

#验证手机号规格
def check_mobile(mobile_str):
    """
    检验手机号格式
    :param mobile_str: str 被检验字符串
    :return: mobile_str
    """
    if re.match(r'^1[3-9]\d{9}$', mobile_str):
        return mobile_str
    else:
        raise ValueError('{} is not a valid mobile'.format(mobile_str))
```

### 生成token

在toutiao目录的utlis下创建token

![](/assets/toutiao_utils_token.png)

定义生成token的方法

```
from itsdangerous import TimedJSONWebSignatureSerializer
from settings import Config
#生成token
def get_user_token(user_id):

    serializer=TimedJSONWebSignatureSerializer(secret_key=Config.SECRET_KEY,expires_in=3600)

    data = {
        'user_id':user_id
    }

    return serializer.dumps(data).decode()
```

在settings.py文件中添加SECRET\_KEY

```
SECRET_KEY = 'AHBvaWtqaGdmZHh6cXdlZHJmZ2hqbSwuLz0tMDk4NzY1NDMyMXEALg'
```



