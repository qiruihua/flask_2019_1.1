# 缓存

## 使用**ab**工具测试接口

**ab**是一个用于测试HTTP服务的工具。他通过命令行交互，可以简单的进行一些压力测试

### 安装**ab**工具

window系统到[https://www.apachelounge.com/download/](https://www.apachelounge.com/download/)去下载

Linux在命令行执行以下指令

```
sudo apt-get install apache2-utils
```

### ab的简单使用

-n ：总共的请求执行数，缺省是1；

-c： 并发数，缺省是1；

-t：测试所进行的总时间，秒为单位，缺省50000s

`ab -n 1000 -c 100 -t 10 http://www.itcast.cn/`

## ![](/assets/ab并发测试.png)

## 定义2个接口

### 接口A查询数据库

```
#查询数据
class UserInfoResource(Resource):

    def get(self):
        user = User.query.filter_by(id=1).first()

        user_data = {
            'id': user.id,
            'mobile': user.mobile,
            'name': user.name,
            'photo': user.profile_photo or '',
            'is_media': user.is_media,
            'intro': user.introduction or '',
            'certi': user.certificate or '',
        }

        return user_data
#路由
user_api.add_resource(views.UserInfoResource,'/user/')
user_api.add_resource(views.CacheUserInfoResource,'/user/cache')
```

### 接口B查询Redis

```
import json
#查询redis缓存
class CacheUserInfoResource(Resource):

    def get(self):
        redis_data=current_app.redis_store.get(1)
        user_data=json.loads(redis_data.decode())
        return user_data
    #用户第一次将数据库中的数据保存到缓存
    def post(self):
        user = User.query.filter_by(id=1).first()

        user_data = {
            'id': user.id,
            'mobile': user.mobile,
            'name': user.name,
            'photo': user.profile_photo or '',
            'is_media': user.is_media,
            'intro': user.introduction or '',
            'certi': user.certificate or '',
        }

        current_app.redis_store.set(1, json.dumps(user_data))

        return {'message':'ok'}

user_api.add_resource(views.CacheUserInfoResource,'/user/cache')
```



