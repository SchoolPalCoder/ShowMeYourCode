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

`router.push(location, onComplete?, onAbort?)`（编程式）
```js
//在history栈中增加一条url记录，可通过游览器的返回操作回到上一个url
//用法一：字符串路径
this.$router.push('/docs')
//用法二：路径对象 path + query
this.$router.push({path:'/docs'})
// /docs?type="preview"
this.$router.push({path:'/docs',query:{type:'preview'}})
// /docs/123  
this.$router.push({path:`/docs/${userId}`})
//用法三：路径对象 name + params 
//name是在配置路由时添加的命名路由，params只和name配合使用才有用
this.$router.push({name:"docs",params:{userId:123}})
//router-link的to功能同上 用来定义导航链接  （声明式）
<router-link to="/docs" class="link">Document</router-link>
```

`router.replace(location, onComplete?, onAbort?)`（编程式）
```js
//与.push()方法不同的是会替换掉当前的url，不会在history栈中增加记录
this.$router.relace('/docs')
//等同于 （声明式）
<router-link to="/docs"replace>Document</router-link>
```

`router.go(n)`（编程式）
n为正数前进n条history栈记录，为负数则为后退。
超出栈的范围则会失败。


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
        //动态匹配路由  id = 123 路径为/group/123
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



