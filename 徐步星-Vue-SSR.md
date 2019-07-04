## 什么是服务器端渲染 (SSR)？

将Vue程序渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将这些静态标记"激活"为客户端上完全可交互的应用程序。
## 原理
![avatar](https://user-gold-cdn.xitu.io/2018/5/24/16390509c83aef03?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
#### 两个entry
* Client entry
Client entry的功能很简单，就是挂载我们的Vue实例到指定的dom元素上
* Server entry
是一个使用export导出的函数。主要负责调用组件内定义的获取数据的方法，获取到SSR渲染所需数据，并存储到上下文环境中。

# 优势

* 更利于SEO
除了 Google 和 Bing 比较完美地实现了对于 SPA（Single-Page Application）的爬虫渲染及内容抓取，大多数搜索引擎包括百度都没有支持
* 更利于首屏加载
特别是对于缓慢的网络情况或运行缓慢的设备。无需等待所有的 JavaScript 都完成下载并执行，才显示服务器渲染的标记，所以你的用户将会更快速地看到完整渲染的页面

# 局限性
* 服务端压力较大 
本来是通过客户端完成渲染，现在统一到服务端node服务去做。尤其是高并发访问的情况，会大量占用服务端CPU资源；
* 开发条件受限
在服务端渲染中，created和beforeCreate之外的生命周期钩子不可用，因此项目引用的第三方的库也不可用其它生命周期钩子，这对引用库的选择产生了很大的限制；
* 学习成本相对较高
除了对webpack、Vue要熟悉，还需要掌握node、Express相关技术。相对于客户端渲染，项目构建、部署过程更加复杂。


# SSR方案
* 基于官方Vue SSR指南文档的官方方案
* vue.js通用应用框架--NUXT


## 更多框架

https://github.com/ream/ream
与Next 不同的是， 你在设计程序上有更多的自由

https://github.com/wallstreetcn/vue-ssr-boilerplate
特点：不依赖vuex
## Nuxt.js
## 特点
默认服务器端渲染；

为加速页面加载，自动进行代码分割；

简化客户端路由（基于页面）；

基于 Webpack 的开发环境，支持热模块更换（HMR）；

可以通过 Express 或任何其他 Node.js HTTP 服务器实现；

可以通过你自己的 Babel 和 Webpack 配置进行定制；


 
## 目录结构

![](https://cdn-images-1.medium.com/max/1600/0*OsCwSkgtlfadtFmU.jpg)

![](https://cdn-images-1.medium.com/max/1600/0*OoR8J9bswCpE6WWU.jpg)
## 路由配置

```
pages/
--| _slug/
-----| comments.vue
-----| index.vue
--| users/
-----| _id.vue
--| index.vue
```
Nuxt.js 生成对应的路由配置表为：

```
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'users-id',
      path: '/users/:id?',
      component: 'pages/users/_id.vue'
    },
    {
      name: 'slug',
      path: '/:slug',
      component: 'pages/_slug/index.vue'
    },
    {
      name: 'slug-comments',
      path: '/:slug/comments',
      component: 'pages/_slug/comments.vue'
    }
  ]
}
```
> 路由自动注入插件 https://github.com/Qymh/vue-router-invoke-webpack-plugin
> 文章地址 https://juejin.im/post/5cb4ad82e51d456e7e297bbf

# 静态构建
Nuxt 最大的创新 就是 ```nuxt generate``` 命令，这条命令可以构建完整的静态页面
举个例子
```
-| pages/
----| about.vue
----| index.vue
```
```
-| dist/
----| about/
------| index.html
----| index.html
```
动态域名
```
generate: {
    routes: [
      '/users/1',
      '/users/2',
      '/users/3'
    ]
  }
```

### 链接

[SSR指南](https://ssr.vuejs.org/zh/)

[Nuxt.js文档](https://zh.nuxtjs.org/guide)