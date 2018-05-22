####Vuex 状态管理模式，用于集中式管理所有组件的状态
安装vuex之后，调用Vue.use(vuex)
获取store对象
```js
const store = new Vuex.Store({
  
})
```
在Vue根实例中注册store选项，将store的实例注入所有的子组件中

```js
new Vue({
  el: '#app',
  router,
  store,
  components: { App },
  template: '<App/>'
})
```
store.commit用来触发状态变更，store.state.count获取对应状态
```js
store.commit('increment')

console.log(store.state.count) // -> 1
```
####State
>单一状态树，用一个对象包含全部应用层级的状态，且在整个应用中是唯一的。
```js
const store = new Vuex.Store({
  //state对象
  state:{
    count :1
  }
});
```
```js
//在组件中获取state的属性值
computed:{
    count:function(){
      return this.$store.state.count
    }
  },
```
####Getter
>store中的计算属性，getter的返回值会根据所依赖的值，进行计算返并且缓存结果。直到依赖的值再次发生改变，返回值才会更新。
```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  //state是gettters的第一个参数
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```
在组件中访问getters对象
```js
this.$store.getters.doneTodos
```
在getters方法中返回一个带参数的方法
```js
getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
}
//调用该方法
this.$store.state.getTodoById(1)
```
####Mutation
>用于修改vuex的store中的状态的唯一方法是提交Mutation。
每一个Mutation都有一个 `事件类型`和`回调方法`，state作为回调函数的第一个参数。
```js
const store = new Vuex.Store({
  state:{
    count:1,
  },
  mutations:{
    //increment 就是事件类型
    increment:(state){
      state.count++;
    }
  }
})
//触发类型为increment的mutation
//调用响应type对应的方法
store.commit("increment")
```
*提交载荷（Payload）*
>store.commit除事件类型的额外传参，大多为一个对象
```js
store.commit("increment",{
  number:10,
})

mutations:{
    //payload第二个参数，额外传参
    increment:(state,payload){
      state.count += payload.number;
    }
  }
```
*对象风格的提交*
```js
store.commit({
  type:"increment",
  number:10,
})
```
`mutation只能处理同步事务`