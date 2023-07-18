---
title: 基于菜单的动态路由
subtitle: 理解基于菜单的动态路由
link: dynamicRouter
catalog: true
date: 2023-02-22 14:28:54
tags:
  - 项目
  - 动态路由
categories:
  - [笔记, 项目]
---

# 基本原理

1. 根据用户的 id 发送请求获取用户能够看到的菜单
2. 将需要动态注册的路由对象放到一个数组里面
   - 路由对象在一个个单独的文件里面
   - 需要从文件中读取路由对象，然后放到数组里面（这一步我们将进行自动化操作）
3. 根据菜单去匹配对应的路由对象，并将其注册
   - `Router.addRoute('main', route)`

# 详细步骤

1. 根据用户的 id 发送请求获取用户能够看到的菜单
   - 这里我们看一下服务器返回给我们的信息
     ![返回的菜单信息](https://i.328888.xyz/2023/02/22/xySLq.png)
   - 对返回的信息进行一波浅浅的分析 🤔
     是一个数组，数组里面存储的是所有一级菜单的信息，`name`,`url`，然后我们发现还有一个`children`熟悉，也是一个数组, 里面存储的是二级菜单的信息
   - 好家伙，你现在是不是有想法了 🤔
     我们直接拿着这个服务器返回给我们的数据，然后去生成菜单不就可以了吗，这样用户看见的就是他能看见的菜单了，他看不到的菜单我们也不会渲染，万事大吉 😁。
   - 这样操作带来的问题
     如果用户也是一个前端呢，他直接在 url 那里输入他没有权限访问的路由，不就可以进入他本不能访问的界面了吗？哦豁，完蛋 🥲。所以我们需要动态注册路由啦，连路由都没有，我看他怎么访问，嗯哼。
2. 将需要动态注册的路由对象放到一个数组里面

   - 首先明白一点，我们的路由对象是放在一个个文件里面的，这样结构更清晰
     举个列子，在`router/main/analysis/dashboard/dashboard.ts`里面才有我们导出的路由对象，如下图
     ![路由对象列子](https://i.328888.xyz/2023/02/22/x4g2p.png)
   - 像这样的文件及其里面的路由对象还有很多，接下来我们要做的就是将其放进一个数组里面，这里我们会进行自动化操作
     - 利用`import.meta.glob(path, options)`这个方法来自动获取`path`路径里面的所有文件，那么我们先用`files = import.meta.glob('../router/main/**/*.ts')`来获取一下我们在此路径里面的 ts 文件导出的路由对象吧，接下来给你们打印一下`files`看看。
       ![获取的files](https://i.328888.xyz/2023/02/22/x4G48.png)
       我们发现是一个对象，key 是文件的`path`路径，value 却是一个箭头函数，这是因为他默认是懒加载获取的文件的，可是这样问题就来了，这样我们要怎么取出文件里面导出的路由对象呢 🤔？
     - 通过`options`选项来获取路由对象
       别急别急，`import.meta.glob(path, options)`方法不是还有第二个参数`options`吗，它是一个配置对象，我们可以设置这个配置对象的`eager`属性为`true`，好的，那我们接下来`files = import.meta.glob('../router/main/**/*.ts', {eager: true})`这样试一下，给你们看看这样做后打印的`files`
       ![配置eager后的files](https://i.328888.xyz/2023/02/22/x4P4J.png)
       怎么样，是不是 value 由箭头函数变成了模块 Moudule，这个模块其实就是一个对象，然后对象里面有个`default`属性，`default`属性的 value 可不就是我们导出的路由对象吗，欧克，大功告成，至此我们已经能够获取我们在文件里面导出的路由对象了。
     - 将路由对象注册到数组里面
       ```typescript
       for (const key in files) {
         const module = files[key];
         localRoutes.push(module.default);
       }
       ```
       欧克，这样我们就成功的将我们在文件里面导出的路由对象放到`localRoutes`这个对象里面了，大功告成，顺便提醒一下这里用`for in`是因为数组也是一个对象嘛，相信你肯定是知道的。
   - **最后读取文件里面导出的路由对象并放到一个数组里面的操作，我们最好给封装到一个函数里面，接下来是封装的函数的代码**

     ```typescript
     function loadLocalRoutes() {
       // 1.动态获取所有的路由对象, 放到数组中
       // * 路由对象都在独立的文件中
       // * 从文件中将所有路由对象先读取数组中
       const localRoutes: RouteRecordRaw[] = [];

       // 1.1.读取 router/main 所有的 ts 文件
       const files: Record<string, any> = import.meta.glob('../router/main/*_/_.ts', {
         eager: true,
       });
       // 1.2.将加载的对象放到 localRoutes
       for (const key in files) {
         const module = files[key];
         localRoutes.push(module.default);
       }

       return localRoutes;
     }
     ```

3. 根据菜单去匹配对应的路由对象，并将其注册

   - 根据菜单匹配需要注册的路由对象，并将其放进一个数组
     我们可以发现我们请求菜单返回的数据里面每一个一级菜单 or 二级菜单都是有一个 url 的，并且这个 url 和我们的路由对象的 path 是对应的，所以我们就可以基于这个来匹配。_但是有一个注意点，一级菜单是没有对应的界面，所以注册一级菜单对应的路由的时候，我们应该将其重定向到子菜单的第一个选项的路由_ 。**同样，这些操作，我们也可以将其封装到一个函数里面，下面是函数的实现**

     ```typescript
      export function mapMenusToRoutes(userMenus: any[]) {
       // 1.加载本地路由
       const localRoutes = loadLocalRoutes()

       // 2.根据菜单去匹配正确的路由
       const routes: RouteRecordRaw[] = []
       for (const menu of userMenus) {
         for (const submenu of menu.children) {
           const route = localRoutes.find((item) => item.path === submenu.url)
           if (route) {
             // 1.给route的顶层菜单增加重定向功能(但是只需要添加一次即可)
             if (!routes.find((item) => item.path === menu.url)) {
               routes.push({ path: menu.url, redirect: route.path })
             }

             // 2.将二级菜单对应的路径
             routes.push(route)
           }
           // 记录第一个被匹配到的菜单
           if (!firstMenu && route) firstMenu = submenu
         }
       }
       return routes
     ```

   - 最后遍历`routes`将里面存储的需要注册的路由对象，动态注册就可以啦

     ```typescript
     const routes = mapMenusToRoutes(userMenus);
     routes.forEach((route) => router.addRoute('main', route));
     ```

4. 注意事项
   这个动态注册路由的操作，应该是在进行登录操作，并且在登录成功跳转到 main 界面之前进行的。
5. 请思考，在登录操作里面跳转到主页面里面对路由做一个动态的注册真的就万事大吉了吗 🤔。

   - 问题：
     想象这样的一个场景，用户在登录之后进入了主界面，点击了它能看到的菜单，并进入了对应的界面，这时候用户点击了刷新会发生什么呢？答案很简单，我们在登录操作里面注册的动态路由没有了，因为用户刷新之后是不会再执行登陆操作的，也就是说刚才注册的路由不会再注册一遍，所以我们需要解决这个问题。
   - 解决方案：

     - 粗糙的解决方案:
       在`main.ts`里面再执行获取需要注册的路由对象，并遍历将其注册的操作，因为刷新的时候`main.ts`会执行，但是我们是一般不希望`main.ts`里面有过多的业务代码的。
     - 优雅的解决方案：
       将上面的操作封装成一个函数`loadLocalCacheAction`放进 Pinia 的 loginStore 里的`action`, 然后在`Store/index.ts`里面导出一个`registerStore`函数，函数里面进行`app.use(Pinia)`使用 pinia 插件操作和`loadLocalCacheAction`操作，详细代码如下

       ```typescript
       import { createPinia } from 'pinia';
       import type { App } from 'vue';
       import useLoginStore from './login/login';

       const pinia = createPinia();

       function registerStore(app: App<Element>) {
         // 1.use的pinia
         app.use(pinia);

         // 2.加载本地的数据
         const loginStore = useLoginStore();
         loginStore.loadLocalCacheAction();
       }

       export default registerStore;
       ```

       最后在`main.js`里面`app.use(registerStore)`调用 registerStore 函数，完成对 pinia 插件的使用，并执行`loadLocalCacheAction`函数，完成对动态路由的注册
