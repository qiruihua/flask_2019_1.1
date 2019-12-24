# 项目准备

### 创建项目 {#创建项目}

* 新建项目，信息虚拟环境，创建`manage.py`文件

```
from flask import Flask

app = Flask(__name__)

@app.route('/index')
def index():
    return 'index'

if __name__ == '__main__':
    app.run()
```



