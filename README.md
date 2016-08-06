# 两步实现Vue + Express全栈自动刷新
## 吐槽
vue和express单独拿出来每个都很容易实现自动刷新
 vue可以通过vue-cli生成的webpack模板实现自动刷新
 express我一般就直接通过node-dev启动 简单粗暴
但前后端联调的时候就比较不方便了，尤其是跨域的话
网上也有一些全栈刷新的实现，看着都太复杂了。。说实话，看不懂！

## 实现思路
下面说下我的实现思路：
vue-cli生成的webpack模板，通过npm run dev可以启动一个调试服务，访问这个调试地址即可
也就是说，仅仅从这个层面来看，vue的自动刷新其实一个网页就搞定了，而网页凭什么能实现自动刷新？那肯定就是通过js了，右键查看页面源码:
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>vue-express</title>
  </head>
  <body>
    <app></app>
    <!-- built files will be auto injected -->
  <script type="text/javascript" src="/app.js"></script></body>
</html>
```
其实就是一个模板html引用了一个app.js嘛
那如果我新建一个同样的html文件给弄到项目的存放网页的目录中去，岂不美哉？
再然后分析一下自动刷新页面需要请求的其它资源，通过express的重定向给打到vue的调试服务路径去，反正只要页面的请求路径适合后端接口同源就好了

## 实现步骤(2步！)
### 一，启动vue调试->新建网页文件
    就是上面思路里说的，在vue项目中执行npm run dev之后，访问调试地址，直接邮件源码复制到后端映射静态资源的目录中去

### 二，express重定向->node-dev
    毕竟目前只有一个网页放在后端嘛，还有一些其它网页资源和自动刷新模块需要请求，那就直接给重定向到npm run的服务中去好了。
    这一步我是图省事直接在app.js中全局拦截，代码如下：
```
//  开发环境
if (app.get('env') === 'development') {
   // 拦截所有请求
   app.use(function (req, res, next) {
       // 过滤自动刷新需要用的资源
       if (req.url === '/__webpack_hmr'
           || req.url.indexOf('.hot-update') !== -1
           || req.url.substr(0, 8) === '/static/'
           || req.url === '/app.js')
           // 3001端口:vue调试服务的调试端口 可以自己去 public/config/index.js中修改
           return res.redirect('http://localhost:3001' + req.url)
       next()
   })
}
```
最后通过node-dev启动express吧
ok！大功告成！

## 关于vue-express
这个就是全栈自动刷新的示例，其实直接就是express-generate生成的express模板，和vue-cli生成的webpack模板(在express目录的public文件夹中)稍作改动
### 使用方法:
1. 安装
```
    git clone https://github.com/aweiu/vue-express.git
    cd vue-express
    npm install
    cd public
    npm install
```
2. 开始
```
  cd vue-express
  node-dev bin/www
  cd public
  npm run dev
```
访问:http://localhost:3000/static/index_dev.html

随便改点后端或前端文件看看吧~










