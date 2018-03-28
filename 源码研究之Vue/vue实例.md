###创建根Vue实例
```
var vm = new Vue({
    data:data
})
```
###实例的数据属性
data对象在`实例被创建时`，其所有属性的值在发生改变时，视图将会响应更新。`实例创建完`之后添加的属性，不具有响应式。
```
var freeObj ={};
Object.freeze(freeObj);
var freeData = new Vue({
    data:freeObj
})
```
Object.freeze()冻结一个对象，其属性的值不可修改，且不可被删除，不能增加新的属性，data对象的值若是被冻结，该对象的属性也时不具备响应式。
###实例的属性和方法
`$`开头的属性用来表示Vue自带属性和方法，用于和自定义属性区分。
####属性
```
vm.$data //实例的data对象
vm.$el //实例对应的根DOM元素
vm.$parent//父级实例
vm.$root//组件树的根Vue实例
vm.$children // Array|直接子组件
```
```
vm.$props //组件接收的props对象
//easy-mock的EmHeader组件定义的props对象
//对象的每个属性都定义了类型
props: {
    title: String,
    description: String,
    icon: String,
    spots: Number,
    nav: Array,
    value: {}
}
//父级组件调用EmHeader组件
//传入了props对象中定义的参数值
//TODO:了解参数前的冒号含义
<em-header
    icon="ios-book"
    :title="page.title"
    :description="page.description">
</em-header>
```
####方法
```
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
//例子
//this.$nextTick DOM更新之后再执行
watch: {
    title: function () {
        this.routeChanged = false
        this.$nextTick(() => {
        this.routeChanged = true
        })
    }
}
```

