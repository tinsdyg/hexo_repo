---
title: Python Flask入门
tags: study
---
## 1. 新建一个Flask App对象
### 简单Flask代码示例

```python
from flask import Flask, request, session, render_template

app = Flask(__name__)  
app.secret_key = 'blog_session' # 使用session需要指定key

# 统一 404 错误处理
@app.errorhandler(404) 
def err404(e):  
    return render_template('404.html'), 404

if __name__ == '__main__':  
    app.register_blueprint(blog)  # 使用Blueprint进行项目分模块
    print(app.url_map)  # 查看 路由表
    app.run(host='0.0.0.0', port=5000, debug=True)

```

## 2. 项目分模块
### 当项目大到一定规模时,所有代码放在一个文件中便会使代码变得臃肿, 这时就有必要对项目进行分模块的操作了,  Flask 中 Blueprint 可以达到这样的效果

```python
from flask import Blueprint, render_template, session, request, redirect

# 模块的创建和app非常相似, 不过模块没有单独的启动语句
admin = Blueprint('admin', __name__)

@admin.route('/admin/login')  
def admin_login_page():  
    return render_template('admin/login.html')


```

使用blueprint后需要在主app中引入各个blueprint，否则不生效