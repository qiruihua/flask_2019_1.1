# Flask-Script 扩展 {#flaskscript-扩展}

[文档](https://flask-script.readthedocs.io/en/latest/)

通过使用Flask-Script扩展，我们可以在Flask服务器启动的时候，通过命令行的方式传入参数。而不仅仅通过app.run\(\)方法中传参，比如我们可以通过：

```
python manage.py runserver -h ip地址 -p 端口号
```

## 代码实现 {#代码实现}

```
from flask import Flask
from flask_script import Manager
...
app = Flask(__name__)
# 把 Manager 类和应用程序实例进行关联
manager = Manager(app)
...

if __name__ == "__main__":
    manager.run()
```

## 集成数据模型管理

```
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand
...
manager = Manager(app)
Migrate(app, db)
manager.add_command('db', MigrateCommand)
...

if __name__ == '__main__':
    manager.run()
```



