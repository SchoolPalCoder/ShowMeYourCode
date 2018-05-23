##vue-i18n
>用于实现多语言功能切换

`main.js`中引入并挂载到Vue根实例中
```
import VueI18n from 'vue-i18n';
//use()方法安装VueI18n插件，并且保证该插件只会安装一次
Vue.use(VueI18n);
//挂载到Vue实例上
const app = new Vue({
    router,
    store,
    i18n,
    render: h => h(App)
  })
  return {
    app,
    router,
    store
  }
```
####locale 全局配置参数 语言标识 
easyMock中通过VueLocalStorage来存取local的值
```
//easyMock源码
const i18n = new VueI18n({
  locale: Vue.ls.get('locale') || 'zh-CN',
  fallbackLocale: 'zh-CN',//缺省语言配置
  //语言包 结合iview的中英文语言包
  messages: {
    'zh-CN': {
      ...zhLocaleIView,
      ...zhLocale
    },
    'en': {
      ...enLocaleIView,
      ...enLocale
    }
  }
})
//通过修改locale的值来切换语言
this.$i18n.locale = this.language; 

```
使用$t()在html模板中使用
```
<li>
    <code class="keyboard-short">s</code>
    {{$t('c.keyboardShort.keyboards[1].list[1]')}}
</li>
```