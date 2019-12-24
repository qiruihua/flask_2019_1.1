# Git版本控制

## 本地提交 {#本地提交}

* 初始化 git

```
$ cd [项目目录]
$ git init
```

* 配置当前项目git提交信息\(可省略此步，如不配置则使用全局配置\)

```
git config user.name XXX
git config user.email XXX@xxx.com
```

* 添加忽略文件

```
$ touch .gitignore
```

* 设置忽略文件内容\(后续根据需要再添加\)

```
.idea
*.py[cod]
```

* 添加所有文件到暂存区

```
$ git add .
```

* 提交到本地仓库并填写注释

```
$ git commit -m '第一次提交'
```

## 远程提交 {#远程提交}

使用码云[https://gitee.com/](https://gitee.com/)作为在线 git 源代码仓库，免费注册账号

* 在码云上创建项目：

![](/assets/码云创建项目.png)

* 创建完成之后，将已有项目上传到码云：

![](/assets/将已有项目上传到码云.png)

### 上传项目到码云 {#上传项目到码云}

#### 使用命令行的形式将项目上传到码云 {#使用命令行的形式将项目上传到码云}

```
$ cd 项目根目录
$ git remote add origin 仓库地址
$ git push -u origin master
```

## 回滚代码 {#回滚代码}

* 回滚到上一版本

```
$ git reset --hard HEAD~1
```

* 查看所有提交版本记录

```
$ git reflog
```

* 回到指定版本

```
$ git reset --hard 提交id
```



