# 1. Node开发环境

## 安装创建项目

1. 安装 `node.js`
2. `npm install -g vue-cli`安装vue-cli
3. 到项目文件夹下使用 `vue create 项目名`创建项目
4. 在项目文件夹下使用 `npm run serve`运行
5. `npm run build`打包项目

vue-cli: https://cli.vuejs.org/zh/guide/

## 项目目录结构

* node-modules 依赖库
* public网页资源
* assets图片等资源
* components --- vue组件
* package-lock.json控制依赖版本
* vue.config.js脚手架配置

```js
const { defineConfig } = require('@vue/cli-service')
module.exports = defineConfig({
  transpileDependencies: true,
  lintOnSave:false //关闭vue cli 的 eslint检查
})
```

# 2. Basic

## data(){}函数

```js
export default {
  name: "HelloWorld",
  data() {
    return {
      name: 'Jean',
      age: 18,
      sex: 'female'
    }
  },
};
```

## 插值

```html
<template>
  <div>
    Hello, I am {{name}}, {{age}} years old.
  </div>
</template>
```

## : 单向绑定

```html
<template>
  <div >
    <a :href="url">点击</a>
  </div>
</template>
<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      url: 'http://www.baidu.com'
    }
  },
};
</script>
```

## v-model 双向绑定

```html
<template>
  <div >
    <input v-model="text">
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      text: '',
    }
  },
};
</script>
```

## Vue底层原理: Object.defineProperty

```js
    let p = {name: 'sunny'};
    let age = 16;
    Object.defineProperty(p, 'pAge', {//给对象定义属性
      get(){
        return age;
      },
      set(newV){
        age = newV;
      }
    });
    //这样p的属性pAge被外部变量age代理了
```

# 3. 事件绑定@

## methods和func($event, ...params)用法

```html
<template>
  <div 
  @click="handleClick"
  style="width: 100px; height: 100px; backgroundcolor: green">
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  methods: {
    handleClick(){
      console.log('click');
    }
  },
};
</script>
```

```html
<template>
  <div 
  @click="handleClick($event, 'param1', 'param2')" class="box">
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  methods: {
    handleClick(event, a, b){
      console.log(event);//原生event对象
      console.log(a, b);
  }
  }
}
</script>
```

## 事件修饰符

```html
<template>
<!-- .stop阻止事件冒泡 -->
<!-- .prevent阻止标签默认事件 -->
<!-- .once事件只触发一次 -->
  <div 
  @click.once.stop="handleClick($event, 'param1', 'param2')" class="box">
  </div>
</template>
```

## 键盘事件@keyup @keydown

> 按下enter触发事件

```html
<template>
  <div>
    <input @keydown.enter="handleKeyDown">
  </div>
</template>
```

> 松开esc触发事件

```html
<template>
  <div>
    <input @keyup.esc="handleKey">
  </div>
</template>
```

> ctrl+y触发事件

```html
<template>
  <div>
    <input @keydown.ctrl.y="handleKey">
  </div>
</template>
```

> 配置修饰符和keyCode的映射关系

```js
import Vue from 'vue'
import App from './App.vue'

Vue.config.productionTip = false;
// main.js中配置按键修饰符
Vue.config.keyCodes.e = 69;//keyCode=69的按键对应修饰符e
```

# computed-计算属性

```html
<template>
  <div>
    <!-- [H]Albert(smart) -->
    {{fullName}}
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      name: 'Albert',
      prefix: '[H]',
      suffix: '(smart)'
    }
  },
  computed:{
    fullName(){
      return this.prefix+this.name+this.suffix;//三者任一变化计算属性fullName会重新缓存
    }
  }
}
</script>
```

# watch-监视属性和深度监视

> 简单写法

```简单使用
    watch: {
//name值变化时调用
//name需要再data()中先声明
        name(newVal,oldVal){
            console.log('name的值发生了变化')
            console.log(newVal,oldVal)
        }
    }
```

> 完整写法

```js
        name:{
//深层监听
            deep:true,
//回调
            handler:function(newVal,oldVal){
                console.log('value is change')
            },
//初始化时是否调用回调
            immediate:true
        }
```

# :class和:style-实现动态样式

> :class绑定动态类描述对象

```html
<template>
  <div class="basicStyleClass" 
        :class="dynamicClasses"
        @click="dynamicClasses.greenClass = true;">
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      dynamicClasses:{
        redClass: true,
        greenClass: false
      }
    }
  }
}
</script>

<style scoped>
/* 基础样式 */
.basicStyleClass{
  width: 100px;
  height: 100px;
  border: 1px solid black;
}
/* 动态变化的样式 */
.redClass{
  background-color: red;
}
.greenClass{
  background-color: green;
}
</style>
```

> :style绑定动态样式对象

```html
<template>
  <div :style="dynamicStyleObj"
       @click="dynamicStyleObj.backgroundColor = 'green'">
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      dynamicStyleObj:{
        width: '100px',
        height: '100px',
        backgroundColor: 'red'
      }
    }
  }
}
</script>
```

# 4. 其他指令

## v-show, v-if---条件渲染

```html
<template>
  <div>
    <span v-show="isRight">正确</span>
    <!-- 下面这个标签的样式为display: none -->
    <span v-show="!isRight">错误</span>

    <span v-if="isRight">正确</span>
    <!-- 下面这个标签不会渲染到dom中 -->
    <span v-if="!isRight">错误</span>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      isRight: true,
    };
  },
};
</script>
```

## v-for 和 :key---循环渲染

```html
<template>
  <div>
    <!-- in后面可以是数组, 也可以是一个数字标表示循环次数 -->
    <span v-for="item in list" :key="item.id">我的序号是:{{item.id}}<br></span>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      list: [
        {id: 177},
        {id: 178},
        {id: 179},
        {id: 180},
        {id: 181},
        {id: 182},
      ]
    };
  },
};
</script>
```

> :key用于指定唯一标识, 强烈建议使用, 否则当list改变时diff算法可能由于无法区分差异, 导致没有更新DOM.

## v-html添加内部Html

```js
<template>
  <div v-html="innerHtml">
  
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return{
      innerHtml: '<h3>Hello</h3>'
    }
  },
};
</script>
```

> 使用时小心XSS攻击

## 自定义指令

```js
// main.js中添加自定义全局指令
Vue.directive('myDirective', {
  // 绑定指令时调用
  bind(element, Info, Vnode){
    //element: 绑定到的dom中的对象
    //Info: 使用指令时的详细信息
    //Vnode: 管理绑定的元素对应的虚拟DOM节点
    console.log('指令已经绑定到元素上了');
    console.log(element, Info, Vnode);
  },

  inserted(element, Info, Vnode){
    console.log('指令所绑定的元素已经插入到Dom中了');
    console.log(element, Info, Vnode);
  },
  update(element, Info, Vnode, oldVnode){
    console.log('指令所绑定的元素对应的Vnode更新前');
    console.log(element, Info, Vnode, oldVnode);
  },
  componentUpdated(element, Info, Vnode, oldVnode){
    console.log('指令所绑定的元素对应的Vnode更新后');
    console.log(element, Info, Vnode, oldVnode);
  }
});
```

> 使用指令

```html
<template>
  <div 
    @click="name='Klee'"
    v-myDirective:Name.modifier1.modifier2='name'>
    {{name}}
  </div>
</template>
```

# `$`set()---添加非一级响应式属性

```html
<template>
<div>
  <!-- 使用这种方法vue 实例的persion.name会改变但vue不会检测其变化然后更新到dom中 -->
  <div style="background: green; width: 100px; height: 100px; margin-bottom: 100px" 
      @click="this.persion.name = 'klee'">
    {{persion.name}}
  </div>
  <!-- $set方法可以添加非一级响应式属性 -->
  <div style="background: green; width: 100px; height: 100px" 
      @click="func">
    {{persion.name}}
  </div>
  <br>
</div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return{
      persion:{
      }
    }
  },
  methods: {
    func(){
      this.$set(this.persion, 'name', 'klee');
      // 相当于中途给this.persion添加了响应式属性name
      // $set方法不能给vue实例添加属性, 只能添加vue实例的非一级属性.
    }
  },
};
</script>
```

# Vue对数组的变更方法的包装

```html
<script>
export default {
  name: "HelloWorld",
  data() {
    return{
      names:['Klee', 'Jean']
    }
  },
  methods: {
    func(){
      // 直接索引的方式, Vue无法检测到数组改变, 无法实现reactive 
      this.names[0] = 'Albert';
      // Vue底层重写了数组的变更方法来实现数组的检测
      this.name.push("Doo");
    }
  },
};
</script>
```

# 5. 生命周期

## 组件的生命周期

* new Vue()
* beforeCreate
* **实现Vue实例的数据监测**(getter, setter方法初始化, 重写数组变更方法, 将data中的属性添加到Vue实例的属性中, methods, computed属性同理. watch/event的函数初始化)
* **created**
* 初始化el(挂载点变量), 模板编译为虚拟dom
* beforeMount
* 将编译好的html替换Dom中的el
* **mounted**
* beforeUpdate
* 监测到数据变化, 根据新数据粉刷和修补好新的虚拟Dom, 并插入的真实Dom中
* updated
* **beforeDestory**
* Vue实例摧毁
* destoryed

## 父子组件的生命周期顺序

* 创建和挂载

> 父beforeCreate -> 父created -> 父beforeMount -> 子beforeCreate -> 子created -> 子beforeMount -> 子mounted -> 父mounted

* 更新

> 父beforeUpdate -> 子beforeUpdate -> 子updated -> 父updated

* 摧毁

> 父beforeDestroy -> 子beforeDestroy -> 子destroyed -> 父destroyed

## 常见钩子及其用法

```js
export default {
  name: "HelloWorld",
  data() {
    return {
      persion: null
    }
  },
  created() {
    //使用ajax初始化data中的内容, 避免在解析模板时由于data中的默认值为null报错
    this.persion ={};
    this.persion.name = 'Jean';
  },
  mounted() {
    //ajax请求
    //设定定时器
    //设置监听器
    //初始化全局事件总线的监听

  },
  beforeDestroy() {
    //摧毁定时器
    //摧毁监听器
    //解除全局事件总线的监听
  },
};
```

# `$`refs----获取子组件实例

```js
//template中:
<video ref="videoPlayer"></video>
<School ref="vc"></School>
//在父组件中可以访问元素
    this.$refs.videoPlayer;//普通html element
    this.$refs.vc;//vue实例
```

# props---给子组件传递响应式参数

```js
  //  完整写法
  props: {//对接收到的参数进行限制
    direction: {
      type: String,
      default: 'column',
      required: false
    }
  },
  //简单写法
  props: ['name', 'age']//参数为字符串类型, 无默认值, 非必须
  //子组件中可以直接使用变量名(相当于data中声明的一样)
```

* 使用组件时可以传入参数

```html
    <course-card v-for="obj in 10" :key="obj" direction="row">
    </course-card>
```

> props参数不可写

```js
export default {
  name: "HelloWorld",
  props: ['name'],//从父组件通过props传递过来的参数是只读的, 父组件中修改参数时会响应式更新到子组件(:key必须为唯一id)
  data() {
    return {
      Name: this.name//通过data(){}可以复制一份可写的参数
    }
  }
};
```

# Vue.use()---插件的使用和定义

```js
//定义一个插件, 一般是单独写一个js文件再export
const myPlugin = {
  //options为Vue.use使用插件时传入的参数
  install(Vue, ...options){
    //可以对Vue进行功能扩展
    Vue.prototype.$hello = function(){console.log('Hello');};
    console.log('插件传入的参数是', options);//['Klee', 'Jean']
  }
}

//在main.js中使用Vue.use(插件对象)来使用插件
Vue.use(myPlugin, 'Klee', 'Jean');
```

* 在vue实例中的使用插件的功能

```js
<script>
export default {
  name: "HelloWorld",
  methods: {
    sayHello(){
      this.$hello();
    }
  }
};
</script>
```

# localStorage---浏览器的本地存储

```js
        //查询一个cookie
        window.localStorage.getItem('user')
        //添加一个cookie
        window.localStorage.setItem('key', 'value');
        //清除一个cookie
        window.localStorage.removeItem('key');
        //清除所有cookie
        window.localStorage.clear();
//sessionStorage API相同, 但cookie在浏览器关闭时释放
```

# mixin

> 全局混入

```js
//main.js中混入全局复用性高的方法或钩子或属性
Vue.mixin({
    data() {
      return {
        VALUE: 18//每个vue实例都可以访问, 如果key冲突以实例中的为准
      }
    },
    methods:{
      sayHello(){//方法和data同理
        console.log('hello');
      }
    },
    created(){
      console.log('mixin created');//钩子函数会在vue实例的相同钩子调用前调用
    }
});
```

> 实际使用时mixin的对象单独写一个js后暴露出来

> 单个vue实例混入多个对象

```js
  mixins: [//数组, 可以混入多个对象
    {
      methods: {
        sayGood() {
          console.log("good");
        },
      }
    },
  ],
  methods: {
    clicked() {
      this.sayHello(); //全局混入的方法
      this.sayGood(); //单个vue实例混入的方法
    },
  },
```

# 6. 全局事件总线

## Vue.prototype.$bus

```js
//main.js
new Vue({
  render: h => h(App),
  router: router,
  beforeCreate() {
    Vue.prototype.$bus = this;
  }
}).$mount('#app')
```

## this.$bus发送和接收消息

```js
//发送消息
methods:{
        send(){
            this.$bus.$emit('hello',this.name);
        }
    }
```

```js
//监听发送消息事件 
mounted(){
        this.$bus.$on('hello',(data)=>{
            console.log(data);
        })
    }
```

```js
//取消事件监听
beforeDestroy() {
       this.$bus.$off('hello');
    }
```

# $nextTick()---下一次解析模板后回调

```js
  data() {
    return {
      myName: 'Jean'
    }
  },
  methods: {
    changeName(){
      for (let index = 0; index < 1000; index++) {
        this.myName = 'Klee';//vue不会粉刷修改1000遍模板, 而是通过异步缓存队列只刷新一次模板
      }
      //在下一次修改模板完成时指定回调
      this.$nextTick(() => {
        console.log('我只会打印一遍');
      });
    }
  }
```

# slot---传递组件标签内部html

> 具名插槽

* 父组件中

```html
<template>
<div>
  <child>
    <!-- 使用template声明插槽  #+插槽名字-->
    <template #red>
      <!-- 插槽中的内容 -->
      <div style="color: red">red</div>
    </template>
    <template #blue>
      <div style="color: blue"> blue</div>
    </template>
  </child>
</div>
</template>
```

> #red是v-slot=''red''的简写

* 子组件中

```html
<template>
  <div class="box">
      <!-- 子组件内部使用slot使用插槽 -->
      <!-- 下面这一行相当于 <div style="color: red">red</div> -->
      <slot name="red"></slot>
      <slot name="blue"></slot>
  </div>
</template>
```

> 插槽参数

* 子组件给父组件中的插槽传参

```html
<slot name="red" message='子组件传递出的参数'></slot>
```

* 父组件插槽接收参数

```html
  <child>
    <template #red='props'>
      {{props.message}}
      <!-- 所有参数会被封装成一个对象 -->
    </template>
  </child>
```

# 7. 前端路由

## 1.  单页面应用原理

***使用vue默认路由时如何在前端控制路由页面?***

```
使用v-if v-show控制html元素
```

***vue-router是如何实现前端路由控制的?***

```
使用History API修改浏览器显示的地址, 同时监听并拦截浏览器前进/后退/跳转/点击等事件, 改为渲染对应的组件到页面上并修改页签icon+标题
```

***如果直接在浏览器新空白页签地址栏上输入路由后的地址会怎样?***

```
服务器接收到请求后, 服务器判断url是否匹配" /static/* ":

>> 如果匹配, 返回服务器/static/*里的静态文件

>> 否则, 返回单页面应用的index.html, 然后由前端的router渲染该前端路由对应的页面
```

***单页面应用和多页面应用的区别?***

```
>> 单页面应用过渡动画更流畅, 搭配XHR可以局部更新

>> 多页面应用只能共享同域名下的cookie和localStorage
```

## 2. Vue-Router API

## 3. 一次路由的生命周期

## 路由基本使用

```js
export default {
  name: 'App',
  mounted() {
    // push replace等可以
    this.$router.push({
      name: 'child'//要路由到的组件的name
      // 路由到子组件会先路由到父组件
    });
  },
}
```

* 在main.js中修改原来的push来防止重复路由报错

```js
import VueRouter from 'vue-router';
const originalPush = VueRouter.prototype.push;
VueRouter.prototype.push = function push(location) {
  return originalPush.call(this, location).catch((err) => {
    return err;
  });
}
Vue.use(VueRouter);
```

## params传参

* 这种方式参数会以less风格传递

```js
//index.js中标识路径
{
                    //搜索结果页面
                    name: 'search',
                    path: '/home/search/:select?/:searchKey?',
                    component: CourserSearchLayout,
                    meta: {title: '搜索课程', changeTitle: true}
}
```

```js
      this.$router.push({
        name: 'search',
        params:{
//参数为空串时不会传递
          select: this.select || undefined,
          searchKey: this.searchKey || undefined
        }
      })
```

```js
    console.log(this.$route.params.select);
    console.log(this.$route.params.searchKey);
```

## query配置项传递参数

* 这种方式参数以url的参数格式传递

```js
      this.$router.push({
        name: 'search',
        query:{
          searchSelect: this.select,
          searchKey: this.searchKey
        }
      }
```

```js
console.log('路由参数', this.$route.query.searchKey);
```

## activated和deactivated---路由钩子

* keep-alive缓存路由组件

```html
    <!-- 被keepalive包裹的路由视图会缓存(跳转离开视图后不会立刻摧毁组件) -->
    <!-- include是组件名字字符串组成的数组 -->
    <keep-alive :include="['HelloWorld', 'Child']">
    <router-view></router-view>
    </keep-alive>
```

* 路由组件钩子

```js
export default {
  name: "HelloWorld",
  data() {
    return {
    }
  },
  // 由于组件缓存, created, mounted, beforeDestory等钩子不会调用
  // 使用activated deactivated代替这些钩子的功能
  activated() {
    console.log('HelloWorld组件视图又回来了');
  },
  deactivated() {
    console.log('HelloWorld组件视图又走了');
  },
};
```

## router.beforeEach---全局路由守卫

> index.js中

```js
// 全局守卫
router.beforeEach((to, from, next)=>{
    console.log('我在每个路由跳转前调用');
    next();
});

router.afterEach((to, from)=>{
    console.log('我在每个路由跳转成功后调用');
});
```

## beforeEnter--单个路由守卫

```js
const router = new VueRouter({
    routes:[
        {
            name: 'helloworld',
            component: HelloWorld,
            path: '/helloworld',
            children:[
                {
                    name: 'child',
                    component: Child,
                    path: '/helloworld/child',
                    beforeEnter: (to, from, next)=>{
                        console.log('我只在将进入child时调用');
                        next();
                    }
                }
            ]
        }
    ]
});
```

## 组件路由守卫

```js
export default {
  name: "HelloWorld",
  beforeRouteEnter (to, from, next) {
    // ...
    console.log('beforeRouteEnter');
    next();
  },
  beforeRouteLeave (to, from, next) {
    // ...
    console.log('beforeRouteLeave');
    next();
  },
  methods: {
    clicked(){
      this.$router.push({
        name:'child'
      });
    }
  },
};
```

# axios

## 基本使用

* `npm intall axios --save`安装
* main.js中挂载

```js
//axios导入
import axios from "axios";
Vue.prototype.$axios = axios;
```

* 基本使用

```js
          // axios通用请求方法, 向服务器发送data
          axios({
            url: "http://127.0.0.1:8080",
            method: "post",
            // 请求体
            data: {
              name: 'tom',
              age: 16,
              sex: 'male'
            }
          })//返回一个 Promise对象
            .then((response) => {
              console.log(response);
            })
            .catch((err) => {
              console.log(err);
            });
        };
        // !axios()方法相当于直接调用axios.request()
```

* 为方便起见，为所有支持的请求方法提供了别名

```js
axios.request()
axios.get()
axios.delete()
axios.head()
axios.options()
axios.post()
axios.put()
axios.patch()
```

## response的结构

```js
  // `data` : 响应体的JSON对象
  data: {},

  // `status` 响应行的 HTTP 状态码
  status: 200,

  // `statusText` 响应的 HTTP 状态信息
  statusText: 'OK',

  // `headers` 响应头
  headers: {},

   // `config` 请求的配置信息
  config: {},
 // 'request' 请求的原生 XMLHttpRequest
  request: {}
```

## 请求配置

```js
 // `url` 是用于请求的服务器 URL
  url: '/user',

  // `method` 是创建请求时使用的方法
  method: 'get', // default

  // `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
  // 它可以通过设置一个 `baseURL` 便于为 axios 实例的方法传递相对 URL
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest` 允许在向服务器发送前，修改请求数据
  // 只能用在 'PUT', 'POST' 和 'PATCH' 这几个请求方法
  // 后面数组中的函数必须返回一个字符串，或 ArrayBuffer，或 Stream
  transformRequest: [function (data, headers) {
    // 对 data 进行任意转换处理
    return data;
  }],

  // `transformResponse` 在传递给 then/catch 前，允许修改响应数据
  transformResponse: [function (data) {
    // 对 data 进行任意转换处理
    return data;
  }],

  // `headers` 是即将被发送的自定义请求头
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` 是即将与请求一起发送的 URL 参数
  // 必须是一个无格式对象(plain object)或 URLSearchParams 对象
  params: {
    ID: 12345
  },

   // `paramsSerializer` 是一个负责 `params` 序列化的函数
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function(params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` 是作为请求主体被发送的数据
  // 只适用于这些请求方法 'PUT', 'POST', 和 'PATCH'
  // 在没有设置 `transformRequest` 时，必须是以下类型之一：
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属：FormData, File, Blob
  // - Node 专属： Stream
  data: {
    firstName: 'Fred'
  },

  // `timeout` 指定请求超时的毫秒数(0 表示无超时时间)
  // 如果请求话费了超过 `timeout` 的时间，请求将被中断
  timeout: 1000,

   // `withCredentials` 表示跨域请求时是否需要使用凭证
  withCredentials: false, // default

  // `adapter` 允许自定义处理请求，以使测试更轻松
  // 返回一个 promise 并应用一个有效的响应 (查阅 [response docs](#response-api)).
  adapter: function (config) {
    /* ... */
  },

 // `auth` 表示应该使用 HTTP 基础验证，并提供凭据
  // 这将设置一个 `Authorization` 头，覆写掉现有的任意使用 `headers` 设置的自定义 `Authorization`头
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

   // `responseType` 表示服务器响应的数据类型，可以是 'arraybuffer', 'blob', 'document', 'json', 'text', 'stream'
  responseType: 'json', // default

  // `responseEncoding` indicates encoding to use for decoding responses
  // Note: Ignored for `responseType` of 'stream' or client-side requests
  responseEncoding: 'utf8', // default

   // `xsrfCookieName` 是用作 xsrf token 的值的cookie的名称
  xsrfCookieName: 'XSRF-TOKEN', // default

  // `xsrfHeaderName` is the name of the http header that carries the xsrf token value
  xsrfHeaderName: 'X-XSRF-TOKEN', // default

   // `onUploadProgress` 允许为上传处理进度事件
  onUploadProgress: function (progressEvent) {
    // Do whatever you want with the native progress event
  },

  // `onDownloadProgress` 允许为下载处理进度事件
  onDownloadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },

   // `maxContentLength` 定义允许的响应内容的最大尺寸
  maxContentLength: 2000,

  // `validateStatus` 定义对于给定的HTTP 响应状态码是 resolve 或 reject  promise 。如果 `validateStatus` 返回 `true` (或者设置为 `null` 或 `undefined`)，promise 将被 resolve; 否则，promise 将被 rejecte
  validateStatus: function (status) {
    return status >= 200 && status < 300; // default
  },

  // `maxRedirects` 定义在 node.js 中 follow 的最大重定向数目
  // 如果设置为0，将不会 follow 任何重定向
  maxRedirects: 5, // default

  // `socketPath` defines a UNIX Socket to be used in node.js.
  // e.g. '/var/run/docker.sock' to send requests to the docker daemon.
  // Only either `socketPath` or `proxy` can be specified.
  // If both are specified, `socketPath` is used.
  socketPath: null, // default

  // `httpAgent` 和 `httpsAgent` 分别在 node.js 中用于定义在执行 http 和 https 时使用的自定义代理。允许像这样配置选项：
  // `keepAlive` 默认没有启用
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  // 'proxy' 定义代理服务器的主机名称和端口
  // `auth` 表示 HTTP 基础验证应当用于连接代理，并提供凭据
  // 这将会设置一个 `Proxy-Authorization` 头，覆写掉已有的通过使用 `header` 设置的自定义 `Proxy-Authorization` 头。
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  // `cancelToken` 指定用于取消请求的 cancel token
  // （查看后面的 Cancellation 这节了解更多）
  cancelToken: new CancelToken(function (cancel) {
  })
```

## 全局默认配置

```js
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';

```

## 添加拦截器

```js
// 添加请求拦截器
axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
  }, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  });

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 对响应数据做点什么
    return response;
  }, function (error) {
    // 对响应错误做点什么
    return Promise.reject(error);
  });
//移除拦截器
const myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

## 并发请求

```js
function getUserAccount() {
  return axios.get('/user/12345');
}

function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}

axios.all([getUserAccount(), getUserPermissions()])
  .then(axios.spread(function (acct, perms) {
    // 两个请求现在都执行完成
  }));
```
