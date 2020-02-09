# 搭建前端环境 {#搭建前端环境}

## 学习目标 {#学习目标}

* 了解前端项目的基本配置修改和项目运行的方法

## 1 准备 {#1-准备}

将资料中**heimatoutiaomobile**拷贝到前端项目目录。

将以下配置文件中的**target**修改为后端服务的**IP**和端口\(**vue.config.js**\)。

```
  devServer: {
    port: '8081',
    open: true,
    proxy: {
      '/api/app': {
        target: 'http://127.0.0.1:5000',
        changeOrigin: true,
        pathRewrite: {
          '^/api/app': ''
        }
      }
    }
  }

```

这个配置主要是解决跨域问题的。

## 2 运行 {#2-运行}

执行以下命令启动前端项目：

```
npm run dev
```



