>以下学习笔记来自初学vue小萌新，欢迎指正和补充。

>主要以分析easyMock项目中使用的路由模块内容

## 创建路由实例
```js
//定义路由配置
const routes = {}
//参数为路由配置对象
const router = new VueRouter({
    routes
})
```
## 创建和挂载根实例
`easy-mock\views\entry\main.js`
```js
//new Vue()创建一个vue应用并且将router挂载在vue应用上。
const app = new Vue({
    router,
    store,
    i18n,
    render: h => h(App)
})
```
使用this.$router可以在所有组件中使用router的实例方法。
### router实例`属性 `
+ router.app router 的 Vue 根实例 
+ router.mode 路由的匹配模式 
+ router.currentRoute 当前的路由信息
### router实例`方法 `         
+ router.beforeEach(guard)
+ router.beforeResolve(guard)
+ router.afterEach(hook) 增加全局的导航卫士
+ router.push(location, onComplete?, onAbort?)
+ router.replace(location, onComplete?, onAbort?)
+ router.go(n)
+ router.back()
+ router.forward() 动态的导航到一个新的URL
+ router.getMatchedComponents(location?)
+ router.resolve(location, current?, append?)
+ router.addRoutes(routes)
+ router.onReady(callback, [errorCallback])
+ router.onError(callback)
```js
//TODO: 补充router的实例方法使用用例
this.$router.push('/log-out')
```
## Router 构造配置
```js
mode: 'history',
    routes: [
      { path: '/login', component: login },
      { path: '/log-out', component: logOut },
      
    ]
```
### mode 配置路由模式
路由配置模式主要有三种类型`"hash" | "history" | "abstract"`,easy-mock项目中使用的是history模式，这种模式主要依赖HTML5 History history.pushState API和服务器配置,即需要后端的配置。
### routes:Array`<RouteConfig>` 
routes RouteConfig 类型的泛数组，设置每一个路径对应的视图组件。
```js
 routes: [
      { path: '/login', component: login },
 ]
 ```
children——嵌套路由
```js
{
    path: '/',//设置根路径
    component: layout,
    children: [
        { path: '/', component: project },
        // /workbench 匹配成功则组件project会被渲染在组件layout视图中的<router-view></router-view>
        { path: 'workbench', component: project },
        { path: 'group/:id', component: project },
        { path: 'group', component: group },
        { path: 'docs', component: docs },
        { path: 'changelog', component: docs },
        { path: 'dashboard', component: dashboard },
        { path: 'profile', component: profile },
        { path: 'new', component: createProject },
        { path: 'project/:id', component: detail }
    ]
}
```

`待续`



