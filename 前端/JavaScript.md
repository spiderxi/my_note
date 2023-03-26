# JavaScript简介

## JavaScript 的历史

JavaScript最初被称为LiveScript，由Netscape公司在1995年开发。

## JavaScript 与 ECMAScript 的关系

JavaScript是ECMA标准的超集，完整的 JavaScript 是由以下三个部分组成：

* 核心（ECMAScript）：提供语言的语法和基本对象；
* 文档对象模型（DOM）：提供处理网页内容的方法和接口；
* 浏览器对象模型（BOM）：提供与浏览器进行交互的方法和接口。

## Node.js是什么?

Node.js 是一个 JavaScript 运行时（Runtime）

Node.js 运行时主要由 V8 引擎、标准库和本地模块组成

#### 1) V8引擎

V8 引擎是一种JavaScript解释器，它负责解析和执行 JavaScript 代码。

V8引擎借鉴了Java虚拟机和C++编译器的众多技术，它将 JavaScript 代码直接编译成原生机器码，并且使用了缓存机制来提高性能，这使得 JavaScript 的运行速度可以媲美二进制程序。

#### 2) 本地模块

所谓本地模块，就是已经被提前编译好的模块，它们是二进制文件，和可执行文件在内部结构上没有什么区别，只是不能单独运行而已。这些本地模块其实就是动态链接库（在 Windows 下是 .dll 文件）。

#### 3) 标准库

本地模块使用 C/C++ 编写，而 Node.js 面向 JavaScript 开发人员，所以必须要封装本地模块的 C/C++ 接口，提供一套优雅的 JavaScript 接口给开发人员，并且要保持接口在不同平台（操作系统）上的一致性。

## JS-单线程不阻塞机制的原理

* callstack

每次调用一个函数，Javascript解释器会生成一个新的数据结构压入CallStack, 函数执行完成时弹出数据结构

在整个文件开始执行时, 会先压入一个数据结构到CallStack, 相当于main()函数调用

* 单线程和不阻塞的实现(event loop)

JS解释器只有一个CallStack, 所以代码是单线程, setTimeout()异步实现的方式是: 当调用setTimeout()时, 解释器讲该异步任务交给浏览器/Node, 当浏览器/Node完成异步任务时, 将回调任务插入到microQueue/macroQueue,

CallStack中所有函数都执行完成时(CallStack为空), Js会按照优先级从队列中取出一个任务压入到CallStack中去执行

# EMCAScript

## 基础语法

### 代码如何在浏览器中运行

JavaScript程序在浏览器环境来运行时使用 `<script>` 标签。

浏览器在解析 HTML 文档时，将根据文档流从上到下逐行解析和显示。JavaScript 代码也是 HTML 文档的组成部分，因此 JavaScript 脚本的执行顺序也是根据 `<script>` 标签的位置来确定的。

```html
  <head>
    <!-- js代码在<script>标签中 -->
    <script type="text/javascript">
      alert("Hello, world!");
      console.log("Hello, world!");
    </script>
    <!-- js也可以使用<script>导入 -->
      <script type="text/javascript" src="./test.js"></script>
  </head>
```

除了 `<script>`标签, js代码还可以在标签的事件属性中(如onclick)

### JS文件延迟和异步加载

\<script>加载外部脚本时，浏览器会暂停页面渲染，等待脚本下载并执行完成后，再继续渲染。原因是 JavaScript 代码可以修改 DOM，所以必须把控制权让给它，否则会导致复杂的线程竞赛的问题。

将 `<script>`标签放到 `<head>`中的做法，这样的加载方式叫做**同步加载** ，如果加载时间过长（比如下载时间太长），就会造成浏览器页面一片空白。

为了防止这种情况, 一般的处理方式是:

1. **将 `<script>`标签放到 `<body>`底部**
2. defer延迟加载
   defer属性能够将 JavaScript 文件延迟到页面解析完毕后再运行。不适用于 < script>标签包含的 JavaScript 脚本。
3. async异步加载

   在加载 JavaScript 文件时新开一个线程，这样浏览器不会因此暂停，而是继续解析html。

### 类型和变量和运算符

```js
// 标识符可以使用_$和abc123
// js默认使用unicode编码, 可以使用汉字做标识符
// String类型字面值可以使用''
// unicode转义
console.log('/u2620');
// Number类型特殊值为NaN
// typeof 返回字符串类型的值, 表示变量类型(基本类型或者"object", 不能体现继承)

//运算符新增===比较类型的同时比较值
//===  比较类型和值是否都相等
//!!!判断是否为NaN时不要使用===
let a = NaN;
console.log(a === NaN);;//false
console.log(Number.isNaN(a));//true

//==不比较类型, 会做类型转换

// var和let的[最大]区别
// 1. let声明的全局变量不是window的属性
// 2. let变量为block scope, var变量为function scope
// 3. var可以重复声明变量, let不可以

//!!!var变量可以在声明前使用(undefined), 使用let或者严格模式阻止这种行为
"use strict";
```

* 其他类型强制转为boolean类型

```js
//0 NaN null undefined ""为false, 其余都为true
console.log(!0 && !null && !NaN && !undefined && !"");//true
```

* primitive type VS reference type

```js
//primitive type 有:
//Boolean Number String Undefined null Symbol
//primitive 用=赋值传递值 , 而Object是传递引用
```

### 函数

```js
//1. 函数声明
var func1 = function () {
    console.log("func1");
};
//!!!函数体内可以访问外部变量
let c = 12;
function func2(a, b) {
    console.log(a + b + c);
}
//2.  传入参数不足时为undefined, 无返回值时返回undefined
//this, 返回方法调用者 全局域里调用者是windows
function test() {
    console.log(this);
}
//3. IIFE防止命名空间污染(ES6之前只有函数作用域)
(function(){
    console.log(1);
})();
//4. 箭头函数
let pf = (a) => a+1;
//!!! 箭头函数体内没有this变量, 函数体内出现this变量会顺着声明的作用域向上查找this
```

* 函数的方法

```js
//1. bind()
//用一个对象替换掉函数体中的this, 得到一个新的函数
let func = function(param){
    console.log(param + this.val);
}
let obj = {val: 11}
let copiedFunc = func.bind(obj);
copiedFunc(1);//return 12

//2. call(), apply()
//函数调用
func.call(obj, 11);//12, 参数使用不定参数的方式传递
func.apply(obj, [11]);//参数使用数组传递
```

* 闭包

```js
// 闭包在代码层的体现上是，函数作用域内部的函数 + 函数作用域内部的函数引用过的数据
// 其核心的作用是用来隔离数据，防止数据被随意篡改
function getUser(NAME){
    let name = NAME;//返回的对象会保留它的语义环境
    return {
        getName(){
            return name;
        },
        setName(n){
            name = n;
        }
    }
}
let tom = getUser("tom");
let jerry = getUser("jerry");
console.log(tom.name);//undefined
```

* 函数的组合(不知道有什么用, 炫技???)

```js
let pipe = (...functions) => 
args => functions.reduce((tempArgs, f) => f(tempArgs), args);

//pipe函数将输入的函数依次连接成管道, 返回最终的函数
let add1 = n => n+1;
let mul2 = n => 2*n;
pipe(add1, mul2)(2);//(2+1)*2 = 6
```

## 部分常用的内置类

### 数组

```js
//1. 数组基本用法
arr = [2, 1];
arr.length;
arr[0] = 1;
arr[100];//undefined

// 2. 方法
arr.push(1);//末尾添加
arr.pop();//末尾弹出
arr.unshift(1);//头部添加
arr.shift();//头部弹出
arr.sort((a, b) => a > b);
arr.forEach(element => {
  console.log(element);
});
arr.slice(start, end);//[start, end)副本
let joined = arr.join(' ');//元素拼接

//3. 函数式编程
arr.map(x => x+1);
arr.filter(x => x > 1);
arr.reduce((sum, ele) => sum+ele, 1);
```

### Date

GMT是前世界标准时，UTC是现世界标准时。

UTC 比 GMT更精准，以原子时计时，适应现代社会的精确计时。但在不需要精确到秒的情况下，二者可以视为等同。每年格林尼治天文台会发调时信息，基于UTC。

时区是基于GMT来偏移的，往东为正，往西为负, 相邻时区的时间相差一个小时。

```js
let now1 = new Date();//返回本地时间戳(电脑上的时间, 可以随意调的)
let now2 = Date.now();//同上
//一系列get, set方法
now1.getTime();//1970-now1的毫秒数
now1.getMonth();
now1.setHours();
//......
//可以有多种构造方式
let d = new Date("December 31, 1975, 23:15:30 GMT+8");
//......
```

### Math String

```js
   
    // Math提供的数学函数
      Math.ceil(1.5);
      Math.sin(3.14);
      Math.pow(3, 2);
      Math.max(1, 2);
      Math.random();//[0, 1)之间的随机数
      // String类的方法
      var s = 'hello, world!';
      s.charAt(1);//'e'
      s = s.concat('haha');//hello, world!haha
      s.indexOf('h');//0
      s.substring(0, 2);//he
      s.split(',');//hello   world!haha
```

### 正则表达式和JSON

```js
var reg = /.+/i; //ignore case
reg.test('ttt');//true
let obj = JSON.parse('{name: "tom"}');
JSON.stringify(obj);
```

### 集合

```js
//1. Set
let set = new Set([1, 2, 2, 3]);
set.add(2);
set.delete(1);
set.size;
set.has(1);
set.clear();
set.forEach(e=>e++);
set.values();

//2. Map
let map = new Map([["key1", 1], ["key2", 2]]);
map.set("key3", 3)
map.get("key3")
map.delete("key3")
map.entries();

//如果使用shallow查重, 使用Weak+XXX
```

### Promise

```js
let p = new Promise((resolve, reject)=>{
    //...异步操作...
    let result = true;
    if(result) resolve();
    else reject();
})
//!!!一创建Promise就会立刻执行上面的代码, 不管是否通过then/catch指定回调
p.then(()=>{
    console.log("success");
}).catch(()=>{
    console.log("failed");
})

```

* async和await语法糖(这个我目前也没见过, 没用过, 不太清楚, 以后修订)

```js
async function request(){
    let resolveReturn = await request1();//等待异步方法1返回的promise resolve
    let resolveRet2 = await request2(resolveReturn);//将异步请求1的结果作为异步请求2的参数
    return new Promise((resovle)=>{resovle()});//异步方法返回值必须是一个Promise
}
//async方法会被封装成一个Promise, 执行时不会阻塞
```

## 面向对象

### 类

实际上，类是“特殊的[函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions)”

```js
//1. 类的函数式声明

//构造函数
function Person(name) {
    this.name = name;
    let msg;//构造函数里可以有local variable
}
//通过prototype设置类方法
Person.prototype.say = function(msg){
    console.log(this.name + " say: " + msg);
}
let p = new Person("tom");
p.say("hello");

//2.类的class声明
class Student extends Person{
    constructor(id){
        super("student_name");
        this.id = id;
    }
    //getter setter
    get sid(){
        return this.id + 1;
    }
    set sid(newId){
        this.id = newId - 1;
        return;
    }
    //method
    study(){
        console.log("study...");
    }
    static run(){
        console.log('run');
    }
}
```

### prototype 和 \_\_proto\_\_

prototype属性是只有函数(Function类)才特有的属性，当你创建一个函数时，js会自动为这个函数加上prototype属性，值是一个空对象。

所有对象(Object.prototype除外)具有属性__proto__  ，一个对象的__proto__是该对象的构造函数.prototype

一个类的构造函数的prototype.__proto\_\_是父类的构造函数的prototype

> 对象里找不到的属性和方法会顺着__proto__向上找(原型链)

* 判断一个对象是否属于某种类型使用instanceof

```js
function Animal(){

}
let a = new Animal();
console.log(a instanceof Object);//true
console.log(a instanceof Animal);//true
```

### Object

```js
//Object里的常用方法

//Object.assign相当于mixin, 常用于实现类似于继承的机制
let newobj = Object.assign({}, {name: "tom", age:18});

//Object.defineProperty添加属性
Object.defineProperty(newobj, id, {
    value: 18,
    writable: false,
    enumerable: false
})

//Object.create()
let Animal = {
    name: 'tom',
    run(){
        //....
    }
}
//实现类似于new的效果
let a = Object.create(Animal);
console.log(a.__proto__);//Animal对象

//...[待补充]
```

# DOM

## document

```js
//1. 获取Dom树中的Element
//getElementBy...
document.getElementById("btn123");
//getElementsBy...
let resList = document.getElementsByTagName("div");
resList[0];
//css选择器
document.querySelector("#id");
document.querySelectorAll(".test");
//常用节点
document.body
document.head

//2. 创建节点
let node = document.createElement("div")
//node只是一个Object, 还没有插入到Dom树中
```

## element

```js
//1. 设置节点的css样式
element.style.width = "18px";
//style属性的命名方法与css属性名可能有区别
element.style.paddingLeft = "20px";

//2. 节点属性信息
element.className;
element.id;
element.attributes;
element.getAttribute("style");
element.removeAttribute("atr");
element.setAttribute("haha", "123");
element.innerHTML;
element.innerText;
element.nodeName;

//3. 节点包括在Dom树中的关系
element.childNodes;
element.parentNode;
element.appendChild(element);
element.removeChild(element);
element.insertBefore(newNode);//插入一个子节点(默认在末尾插入)

//4.操作节点(包括event listener等)
element.addEventListener("click", ()=>console.log("点击"))
element.removeEventListener("click", ()=>{});

//5. 尺寸信息
//滚动信息
element.scrollHeight//包含不可见部分, 不包含border
element.scrollWidth;
element.scrollLeft
element.scrollTop
element.scrollBy(1, 1);

//元素尺寸
element.clientHeight;//不含border和位于content area+padding里的滚动条
element.clientWidth;
element.offsetHeight;//包含border(css属性值, !!!如果元素缩放了不要使用)
element.offsetWidth;
element.offsetTop;//相较父元素的左上角偏移量
element.offsetLeft;
//获取实际可见矩形的尺寸和位置信息(不被缩放影响的offsetHeight), 
element.getBoundingClientRect();
```

## event

```js
// 事件的lifecycle: 捕获阶段 -> 目标阶段 -> 冒泡阶段
// 捕获阶段从父元素到子元素捕获事件, 此时默认不会触发事件的回调函数
// 冒泡阶段从子元素到父元素, 冒泡式调用事件回调函数
// addEventListener 第三个参数设置是否在捕获阶段就调用回调函数而不是在冒泡阶段


//Event通用的属性和方法
e.target//触发事件的element
e.preventDefault();//阻止事件的默认回调(如点击<a>会跳转)
e.stopPropagation();//阻止事件冒泡
e.stopImmediatePropagation();//阻止事件冒泡的同时阻止其他相同事件监听器的执行

//不同的event还有不同的成员属性和方法, 如鼠标点击事件会有鼠标的坐标信息
```

## setTimeout和setInterval

## 和requestAnimationFrame

```js
//1. setInterval--循环执行
let num = 0;
var intervalId = setInterval(function (msg) {
  num++;
  //关闭定时器
  if (num > 5) {
    console.log(msg);
    clearInterval(intervalId);
  }
}, 1000, "循环执行结束"); //定时1000ms 调用一次函数
// 当定时器被关闭时, msg会被作为参数传递给定时调用的函数

// 2.setTimeout
// 用法和定时器相同, 但只会执行一次
var tid = setTimeout(function(){
  console.log(1);
}, 1000);
clearTimeout(tid);

//3.requestAnimationFrame
//设置下一次渲染网页时的回调
let id = requestAnimationFrame(callback);
cancelAnimationFrame(id);
//使用原因: 
//1.如果网页处于后台, 使用setInterval会占用过多的资源
//2.setInterval回调的间隔不固定, 可能会出现动画不流程或不必要的刷新
```

# BOM

所有浏览器都支持 window 对象。它表示浏览器窗口(不是网页窗口)。

所有 JavaScript 全局对象、函数以及变量均自动成为 window 对象的成员。

全局函数是 window 对象的方法。

全局变量是 window 对象的属性。

甚至 HTML DOM 的 document 也是 window 对象的属性之一

## location

```js
//当前地址栏映射的对象
window.location;
//1. 属性
location.href == location.protocol + 
"//"+location.hostname+ ":"  + location.port
+ location.pathname + "?" + location.search;
//特别地
location.host == location.hostname+":"+location.port

//2.方法
location.assign("www.baidu.com");//跳到指定网址
location.href = "www.baidu.com";//效果同上
location.replace("www.baidu.com");//效果同上, 但是没有历史记录
location.reload();//刷新
```

## history

```js
//浏览器的历史访问的URL信息对象
//history对象无法获得历史记录隐私
window.history;
window.history.forward();
window.history.back();
window.history.go(-2);//相当于连续两次back
```

## navigator

```js
//浏览器和操作系统信息映射的对象
window.navigator
//1.属性
let uad = navigator.userAgentData;//useragent映射的对象
navigator.languages//系统使用的语言
let {platform} = uad;//获取useragent里的platform信息

//deprecated的属性
navigator.userAgent//ueragent字符串
navigator.platform//操作系统名称
navigator.appName//浏览器名称
//deprecated的还有其他关于浏览器和操作系统的属性, 略过
```

## screen

```js
//显示屏幕区域信息映射的对象
window.screen;
//属性
screen.width;
screen.height;
screen.availWidth;//除开windows任务栏的区域宽度
screen.availHeight;
//其他属性, 包括调色板比特深度等等
```

## 窗口尺寸相关属性

```js
//窗口的文档显示区的高度, 宽度
window.innerHeight;
window.innerWidth;
//浏览器界面的宽高
window.outerHeight
window.outerWidth
//浏览器界面左上角的屏幕坐标
window.screenX;
window.screenY;
```

## 其他属性

```js
window.length//窗口包含的iframe, frame的个数
window.name//主要用于为超链接和表单设置目标（targets）, 相当于给窗口一个别名
window.closed;//窗口是否关闭
window.self;//相当于window
window.document;//dom
window.parent;
//如果当前窗口为frame, parent就为包含frame的窗口
//如果没有parent, window.parent为自身引用
window.top;
//window.parent一直到最顶级的祖先就是window.tops
window.status//窗口底部状态栏文本[deprecated]
window.opener
//如果当前窗口是其他窗口打开的, 返回打开者的引用, 否则为null
```

## window常用方法

```js
//交互
window.close();
window.moveTo(0, 0);//移动窗口左上角
window.moveBy(1, 1);
window.resizeBy(1, 1);//调整窗口宽高
window.resizeTo(100, 100);
window.scrollTo(500, 0);//滑动窗口
window.scrollBy(1, 1);

//弹窗
window.alert("msg");
window.confirm("msg");
window.open("www.baidu.com");//重新打开一个窗口

//控制
window.blur();
window.focus();
window.print();//打印机打印网页
//还有其他重要方法(如定时器方法), 单独讲
```

# ES6

## let和const

```js
      // 1. let 声明时变量名不可以重复
      //2. let 声明变量之前不能使用, 会报错
      // 3. var是函数作用域或全局作用域，let是块作用域。
      //4. const变量 相当于不能修改的let变量
```

## 解构

```js
      // 1. 数组
      let arr = [1, 2];
      let [num1] = arr;
      console.log(num1);//1

      //2. 对象
      let p = {name: 'tom', hello: function(){}};
      let {hello} = p;
      hello();

      // 3.函数中使用解构
      function hello({name, sex='male'}){
        console.log(`name=${name}, sex=${sex}`);
      }
      let p = {name: 'tom'};
      hello();
```

## 模式字符串

```js
  let name = "tom";
  console.log(`i am ${name}`);
```

## Object字面值简化

```js
      let name = 'tom';
      let hello = function(){console.log('hello')};
      let person = {
        name,//之间写变量名作为属性名, 变量值作为属性值
        hello,
        say(){//方法可以省略: function
          console.log('say..');
        }
      };
```

## 函数参数默认值和不定参数

```js
      // 1. 参数可以有默认值
      function add(a, b ,c=10){
        return a+b+c;
      }  

      //2. ...参数
      function test(...args){
        console.log(args);//args数组
      }
```

## 扩展运算符

```js
      //1. 扩展运算符可以将数组转化为序列
      let arr1 = ['tom', 'jerry'];
      let arr2 = ['john', 'alice'];
      let arr3 = [...arr1, ...arr2];//合并数组
      let nums = [1, 2, 3];
      sum(...nums);//拆开数组用来传参
```

## Promise

```js
      //1. 使用Promise完成异步编程
      const p = new Promise(function(resolve, reject){
        setTimeout(function(){
          reject('失败原因');//resolve调用时p的状态变为成功
        }, 1000);
      });
      p.then(function(value){
        console.log(value);//成功后返回的值
      }, function(reason){
        console.log(reason);//失败原因
      });
```

## async & await

async

* async作为一个关键字放在函数的前面，表示该函数是一个异步函数，意味着该函数的执行不会阻塞后面代码的执行 异步函数的调用跟普通函数一样.
* 并且async函数默认返回一个Promise对象

await

* await即等待，用于等待一个Promise对象。它只能在异步函数 async function中使用，否则会报错
* await表达式会暂停当前 async function的执行，等待Promise 处理完成。若 Promise 正常处理(fulfilled)，其回调的resolve函数参数作为 await 表达式的值，继续执行 async function，若 Promise 处理异常(rejected)，await 表达式会把 Promise 的异常原因抛出。如果 await 操作符后的表达式的值不是一个 Promise，那么该值将被转换为一个已正常处理的 Promise。


## ES模块化与CommonJS模块化

```js
export default {'age': 18};//默认暴露
//解构暴露
export let name = 'tom';
export function test(){

};
export * from "d3-array";//将子包导出

import {name} from './test';//适合解构暴露
import person from './test';//仅适用于默认暴露
import * as test from './test';//全部导入, 使用namespace
```

CommonJs是一种js语言规范,由于EMCA早期没有模块化, js使用的是CommonJs的模块化规范

```js
let engine = require("./engine");//使用require导入
//require除了导入模块, 还可以导入img, json等文件

//2. 使用module.export导出模块
module.exports = {
    name: 18
}
```
