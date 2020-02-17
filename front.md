# 运行前端vue {#搭建前端环境}

## 准备 {#1-准备}

将中**toutiaomobile**解压到桌面。

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

## 运行 {#2-运行}

执行以下命令启动前端项目：

```
npm run serve
```

> 如果启动报错在终端中执行 npm install 之后再执行启动命令



