# vite--构建工具

* `npm init vite-app (项目名)`
* `npm install`
* `npm run dev`
* `npm run build`

> 如果需要配置vite, 在项目根目录添加vite.config.js导出配置JSON

* `npm install @vitejs/plugin-vue`安装vite关于vue的插件来打包vue文件
* vite.config.js中导入插件

```js
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
    plugins: [vue()]
})
```

# Vue3中的路由


* `index.js`中使用组合式api配置路由

```js
import {createRouter, createWebHistory} from 'vue-router';

//动态加载的方式
const HelloWorld = ()=> import('../components/HelloWorld.vue');

const router = createRouter({
    history: createWebHistory(),//history路由方式---弃用mode配置项
    routes:[
        {
            path: '/',
            redirect: '/home'
        },
        {
            name: 'home',
            path: '/home',
            component: HelloWorld,
            meta: {title: '黄金矿工(vue3+vite)'}
        }
    ]
});
//路由守卫
router.beforeEach((to, from, next) => {
    to.meta.title && (document.title = to.meta.title);
    next();
});

export  default router;
```



* `main.js`中使用router

```js
let app = createApp(App);

//vue-router
import Router from './router/index';
app.use(Router);
```


* useRouter()和useRoute()

```js
  setup(){
    //在setup中获取route和router
    let router = useRouter();
    let route = useRoute();//route是一个reactive对象
    onMounted(()=>{
      console.log(route);
      console.log(router);
    });
    //获取路由query参数
    let id = route.query.id;
    //动态路由
    // router.push({
    //   name: 'home',
    //   query:{
    //     id: 666
    //   }
    // });
    return {
    };
  }
```

> 其他组合式api以此类推


# setup(props, context)--代替date,methods

## setup()在beforeCreate()前执行

## props--父组件传参

## context.slots--插槽

## context.emit(eventStr, params)--发射自定义事件

## return {}--返回变量和方法

# 组合式api--需要时导入(选项式api底层)

# reactive(targetObj)--包装对象为响应式对象

# ref(value)--包装基本类型为响应式

# Proxy--vue3底层响应式原理

## new Proxy(target, {})--构造代理对象

## get(target, propName)--获取属性

## set(target, propName, newV)--新增或修改属性

## deleteProperty(target, propName)--删除属性

## window.Reflect.get(target, propName)---利用反射操作target对象防止抛错

# computed(()=>{})--计算属性

# watch(expOrFn, (newV)=>{}, {})--监视器

# watchEffect(()=>{})--更注重过程的computed

# 生命周期

## beforeUnmount()和unmounted()--声明周期钩子易名

## onXXX(()=>{})--组合式api形式的生命周期钩子

# 自定义hook--用函数实现复用代码

# toRef(obj, prop)/toRefs(obj)--获取对象的属性的响应式引用

# 使用频率小的组合式api

## shallowReactive()和shallowRef()--浅层封装响应式

## readonly(exp)--包装成只读

## toRaw(exp)--将reactive()包装的对象解封

## markRaw(prop)--添加非响应式属性

## provide(nameStr, value)和inject(nameStr)--爷孙组件通信

# isRef, isReactive, isProxy, isReadonly--判断是否为响应式

# `<teleport to='selector'>`传送成为其他标签的子组件

# `<Suspence>`和default, fallback插槽实现异步组件替代展示

# app.xxx代替vue2全局api: Vue.xxx
