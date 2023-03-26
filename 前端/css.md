## Basic基本概念

### CSS写的位置

1. 标签的style属性

```html
<p style="background-color: antiquewhite;">Hello, world!</p>
```

2. `<head>`中的`<style>`标签中

```html
    <style>
        p{
            color: aquamarine;
        }
    </style>
```

3. `<link>`

```html
    <!-- 引入外部CSS文件 -->
    <link rel="stylesheet" href="./defaut.css">
```

### 属性的继承性

部分样式属性会继承给后代元素(如color), 而有些属性则不继承给后代.

具体的属性是否继承可以查阅[CSS手册](https://developer.mozilla.org/zh-CN/docs/Web/CSS)

```css
        /* span的后代元素默认color属性为blue(color继承) 
            background-color默认为transparent(background-color不继承)
        */
        span{
            color: blue;
            background-color: blue;
        }
```

### block, inline, inline-block

* block(`<div>`)独占包含块中的一行, width默认100%, height默认auto, 盒子模型
* inline-block(`<span>`)width默认100%, height默认auto, 盒子模型
* inline 不是盒子模型, width和height固定auto, 水平方向会挤开其他元素, 并且没有margin重叠

### 其他必要的知识

* css属性前缀

css属性前缀有：1、“-moz-” 2、“-ms-” 3、“-webkit-” 4、“-o-” 代表不同浏览器渲染内核

**为什么要有私有前缀呢？**

因为制定HTML和CSS标准的组织W3C动作是很慢的。通常，有w3c组织成员提出一个新属性，比如说圆角border-radius，大家都觉得好，但是w3c不会为这个属性制定标准，而是要走很复杂的程序，经过很多审查。而浏览器商不愿意等那么久，他们觉得一个属性已经够成熟了，就会在浏览器中加入支持。

但是避免日后w3c公布标准时有所变更，就会加入一个私有前缀，比如-webkit-border-radius，通过这种方式来提前支持新属性，等到日后w3c公布了标准，border-radius的标准写法确立之后，再让新版的浏览器支持border-radius这种写法。

## CSS选择器

* CSS基本语法

```css
        p{
            color: aquamarine;
            background-color: antiquewhite;
        }
        /*这是注释*/
        /* 语法格式:
        选择器{
            key1: value1;
            key2: value2;
        }
        */
```

* 常用的选择器

```css
        /* 按标签名 */
        div{
            background-color: antiquewhite;
        }
        /* 按id id=home*/
        #home{
            background-color: aqua;
        }
        /* 按class class=nav*/
        .nav{
            background-color: bisque;
        }
        /* 所有标签 */
        *{
            font-size: large;
        }
```

* 复杂的选择器

```css
    	/* 标签名=div 且 class=test */
        div.test{
            background-color: antiquewhite;
        }
        /* 标签名=div 或 标签名=span */
        div, span{
            background-color: aliceblue;
        }
        /* 父子关系  div下的一级span*/
        div > span{
            background-color: aliceblue;
        }
        /* 祖先后代关系 div下的所有span*/
        div span{
            background-color: aliceblue;
        }
        /* 兄弟关系 div的同级span*/
        div + span{
            background-color: aliceblue;
        }

/* 属性选择器, 选择具有id属性的div */
div[id]{
    color: gray;
}

```

* 伪类和伪元素选择器
  ```css
          /* 伪元素 伪类选择器是指特定条件才会有的样式 */
          /* :active 用户按下左键时发生的样式，松开左键即完成 */
          div:active{
              background-color: aliceblue;
          }
          /* 常用的还有: */
          /*  :checked 处于选中状态的单选或复选框 */
          /* :disabled 不可用UI元素 */
          /* :hover 鼠标悬浮到一个元素之上时的样式 */
          /* :focus用于元素成为焦点时的样式 */
          /* ::after 在元素的内容区之后加的一个元素 */
          /* ::before 在元素的内容区之前加的一个元素 */

  /* :root代表文档的根元素 html中一般是<html> */
  :root div{
      color: gray;
  }
  ```
* 选择器优先级

CSS选择器都有权重，权重值越大优先级越高.属性冲突时高优先级有效.

* 属性值style内联的权重值最高，值为1000
* id选择器的权重值为100
* class选择器的权值为10
* 标签名选择器的优先级为1
* 通配符选择器*的优先级为0
* 后定义的要优于先定义的
* 键值对后面用!important修饰的属性 优先级无穷

```css
.myclass {
  background-color: gray;
}
 
p {
  background-color: red !important;
}
/*class=myclass的p标签为红色*/
```

## 单位

* 长度单位

```css
      /* 长度单位 */
      div{
          /* 像素点大小 */
          width: 100px;
          /* 相对于父元素百分比 */
          height: 50%;

          /* 1em= 1个父元素的font-size(字体高度)*/
          font-size: 10px;
          top: 10em;
          /* 1rem= 一个<html>的font-size*/
          bottom: 1rem;

          /*xx-small small medium large xx-large等英文单词*/
          border-width: medium;
          /* in英寸 mm 毫米 cm厘米 */
          margin: 1cm;
      }
```

* 颜色单位

```css
            /* 颜色可以使用颜色对应的英文单词 */
            color: teal;
            /* rgb值 */
            background-color: rgb(255, 255, 255);
            /* rgb简写(16进制)*/
            border-color: #112345;
            /* rgba, alpha不透明度值范围: 0完全透明->1完全不透明 */
            stop-color: rgba(2, 5, 6, 0.5);
            /* HSL HSLA H:色相(0-360) S:饱和度(0-100%) L:亮度(0-100%) A:透明度*/
            color: hsla(105, 50%, 50%, 0.5);
```

## 盒子模型(box-model)

所有元素的形状为矩形, 就像一个box

从内到外区域以此为: content, padding(内边距), border, margin(外边距)

* **content area**

`weight`和 `height`是content区的宽高,

子元素在content区中

* border有关属性

```css
        div{
            border: 1px blue dotted;
            /* border-color: aliceblue; */
            border-width: medium;
            /* style默认值为none不显示 */
            border-style: solid;
            /* border-xxx-yyy可以设置具体边的属性 */
            border-top-color: aqua;
        }
```

* padding

```css
        padding: 10px;
        /* padding-xxx指定具体方向 */
        padding-left: 5px;
        /* background-color属性会作用于content 和 padding区域 */
```

* margin

content padding border 属于box模型的可见区域, ***margin主要用于控制box的位置***

```css
        /* margin用于控制元素在父元素content区的位置 */
        /* left, top 会移动自己 */
          margin-left: 10px;
          margin-top: 10px;
          /* right bottom会挤掉其他元素 */
          margin-right: 1px;
          margin-bottom: 1px;
```

### 块元素的布局

1. 块元素水平布局

子元素水平方向的属性默认满足:

> margin-left + border-left + width+ border-right + margin-right = 父元素的width

2. 块元素垂直布局

* 父元素height=auto时子元素会撑开父元素
* 若父元素height固定, 子元素可能溢出

溢出时使用 `overflow`属性设置

```css
        /* 溢出内容可见 */
        overflow: visible;
        /* 溢出内容隐藏 */
        overflow: hidden;
        /* 溢出时添加两个滚动条 */
        overflow: scroll;
        /*  自动添加滚动条*/
        overflow: auto;
```

### 块元素的垂直margin的重叠

* 当垂直相邻的同级两个元素上面的`margin-bottom=100px ` 下面的`margin-top=120px`

实际margin区域重叠, 所以margin的宽度实际为120px

* 垂直相邻的父子两个元素(父元素与它的第一个子元素)margin-top区域也会重叠

### display visibility可见性

```css

          /* display可以指定元素的展示方式 */
          display: block;
          display: inline;
          /* 元素退出文件流, 不会站着原来的位置 */
          display: none;
          /* 默认值, 可见 */
          visibility: visible;
          /* 会占着原来的位置, 但不可见 */
          visibility: hidden;
```

### 其他属性

```css
        /* box-sizing用于指定height和weight指定的范围(默认为content-box) */
          box-sizing: content-box;
          box-sizing: border-box;
          /* outline 用法同border, 与border不同的是不会影响页面布局*/
          outline: 1px red solid;
          /* box-shadow 指定阴影, 阴影不影响布局, x偏移 y偏移 模糊宽度 颜色*/
          box-shadow: 1px 1px 10px red;
          /* 圆角 */
          border-radius: 5px;
          border-radius: 50%;
            /* 最大/小高度 最大/小宽度 */
            max-width: 100px;
            min-height: 200px;

        /* z-index, 层级, 越大越不会被覆盖 */
        z-index: 100;
```

## 特定结构的CSS样式

### ol/ul/table

```css
ol, ul{
    /* list item的marker的样式 位置 marker如果用图片代替时图片的url*/
    list-style: circle inside url("www.baidu.com/test.img");
}
table{
    /* 单元格边框是否合并 */
    border-collapse: collapse;
    /* 单元格边框不合并时的边框间距(四个外边距) */
    border-spacing: 0;
}
```

## 浮动float

### Basic

```css
        /* float默认值不浮动 */
        float: none;
        /*可选值 left right  */
        float: right;
        /* float的元素会脱离文档流*/
        /* 浮动元素会在其父元素的conotent里浮动且浮动元素之间会阻挡漂浮 */
```

脱离文档流特点:

* 宽度默认被内容撑开(同高度相同)

> 浮动的元素不会遮挡文字, 文字会环绕!!!

### 高度坍塌

> 高度坍塌: 由于子元素浮动脱离文档流, 父元素的heigth = 0(被非浮动子元素撑开)

* clear属性

```css
        /* clear属性可以根据浮动元素自动设置margin使得不被浮动元素覆盖*/
        /* left时 左浮动的元素不会覆盖自己 */
        clear: left;
        clear: right;
```

> clear属性可以阻止高度坍塌
>
> ```css
>       #div::after {
>         /* 使用clear阻止高度坍塌 (div是高度为auto的父元素)*/
>         display: block;
>         clear: both;
>       }
> ```

## 弹性盒

用来代替float来实现布局, 子元素的大小会随着父元素大小变化而变化

```css
	.inner1{
        background-color: red;
        width: 100px;
        height: 100px;
        /* 设置主轴方向弹性基础值(默认为弹性元素的高度或宽度) */
        flex-basis: 100px;
        /* 当父弹性容器由空余空间时, 弹性元素所分的空余空间的比例 */
        flex-grow: 1;
        /* 当父弹性容器空间不足时, 弹性元素缩小的比例 */
        flex-shrink: 1;
        /* 覆盖弹性容器的属性align-items */
        align-self: flex-start;
        /* 指定弹性元素在弹性方向的排列顺序 */
        order: 1;
      }
      .outter{
        /* 设置为弹性容器*/
        display: flex;
        /* 设置弹性方向 默认row */
        flex-direction: column;
        /* 从右往左 */
        flex-direction: row-reverse;
        /* 当弹性容器一行满了, 是否会自动换行 */
        flex-wrap: nowrap;
        /* 设置弹性元素在主轴上如何对齐 */
        justify-content: space-around;
        /* 弹性元素在辅轴上如何对齐 */
        align-items: center;
      }
```

## 定位

```css
        /* position默认值为static, 不开启定位 */
        position: static;
        /* 相对定位会提高元素层级 */
        /* 相对于自己之前的位置定位 并设置偏移量*/
        position: relative;
        top: 10px;
        bottom: 10px;
        left: 28px;

        /* 绝对定位 */
        position: absolute;
        top: 50px;
        /* 绝对定位会使得元素层级提高一级 */
        /* 元素会变成行内元素 */
        /* 绝对定位的偏移是相对于最近的开启position的包含块 */

        /* 固定定位偏移量是相对于浏览器的视窗 *
        position: fixed;

        /* 粘滞定位 */
        position: sticky;
        top: 7px;
	/*粘滞定位元素高度必须小于父元素高度(这样才容得下), 且他的包含元素overflow不能为hidden*/
```

## 字体和文本

```css
    /* 
    font有四条线
    从上到下四条线分别是顶线、中线、基线、底线，很像学英语字母时的四线三格
    */
    /* 字体特殊样式 */
    font-style: italic;
    /* 字体变种, 常用于大写化字母 */
    font-variant: small-caps;
    /* 字体粗细 */
    font-weight: lighter;
    /* 字体大小 */
    font-size: large;
    /* 相邻baseline之间的垂直距离 */
    line-height: 10;
    /* 字体样式 */
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    /* 属性简写 */
    font: italic small-caps;


    /* 前景色 */
    color: red;


    /* 文本垂直对齐方式 */
    vertical-align: baseline;
    /* 文本水平对齐方式 */
    text-align: center;
    /* 单词过长时的换行方式 */
    overflow-wrap: break-word;
    /* 文本缩进大小 */
    text-indent: 2em;
    /* 换行方式: nowrap:忽略元素中的换行符, 合并多个连续空白 pre:不合并空白 pre-wrap:不合并空白, 不忽略换行符 */
    white-space: nowrap/pre/pre-wrap;
    /* 文本溢出时的处理方式 */
    text-overflow: ellipsis;
    /* 下滑线, 删除线等 */
    text-decoration: none;
    /* 阴影:  h-shadow  v-shadow  blur(模糊距离)  color; */
    text-shadow: 1px 1px 2px pink;

    text-transform: capitalize;

  /* 改变文本渲染策略, 提高文本可读性(清晰度) */
  text-rendering: optimizelegibility;

```

## 背景

```css
        /* 设置背景图片 */
        background-image: url('./defaut.png');
        /*图片较大时显示不全, 较大时会重复直到铺满元素*/
        /* 设置不重复 */
        background-repeat: no-repeat;
        /* 设置背景图片位置 */
        background-position: center center;
        /* 设置背景的范围 */
        /* 设置背景包括border */
        background-clip: border-box;
        /* 设置背景图片的大小 */
        background-size: 100px 100px;
        background-size: contain;
        background-size: cover;
        /* 设置线性渐变色 */
        background-image: linear-gradient(to right, red, yellow);
        /* 设置径向渐变 */
        background-image: radial-gradient(red, yellow);
```

## 过渡动效transition

通过transition属性, 可以指定属性改变时的过渡动画.

```css
      .box2{
        width: 100px;
        height: 100px;
        background-color: aquamarine;
        /* 下面四个属性综合简写 */
        transition: all 0.5s;
        /* 指定过渡的属性 */
        transition-property: height, width;
        /* 指定过渡的速度曲线函数 */
        transition-timing-function: ease;
        /* 指定函数 */
        transition-timing-function: cubic-bezier();
        /* 指定过渡动画的延迟 */
        transition-delay: 1s;
        /* 指定过渡时长 */
        transition-duration: 0.3s;
      }
```

## 动画animation

动画与过渡的区别是动画不用属性变化时就可以触发.

```css
      /* 设置动画需要先声明关键帧keyframe */
      @keyframes identifier {
        /* from to设置关键帧的起始状态 */
        from{
          margin-left: 0;
        }
        50%{
          margin-left: 100px;
        }
        to{
          margin-left: 500px;
        }
      }
      .box2{
        width: 100px;
        height: 100px;
        background-color: aquamarine;
        /* 用关键帧设置动画 */
        animation-name: identifier;
        animation-duration: 1s;
        animation-delay: 1s;
        animation-timing-function: ease;
        /* 动画执行次数 */
        animation-iteration-count: 2;
        /* 动画的方向 */
        animation-direction: alternate;
        /* 动画执行的状态 */
        animation-play-state: running;
      }
```

## 变形transform

变形与margin不同的是变形不会影响布局

```css
      .box1{
        margin: 50px auto auto auto;
        width: 500px;
        height: 500px;
        background-color: rgb(78, 177, 111);
        transition: all 0.2s;
      }
      .box1:hover{
        /* 向上移动5px */
        transform: translateY(-5px);
        /* 添加阴影实现浮动的卡片效果 */
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
      }
```

* Z轴变形

```css
      .box1{
        margin: 200px auto;
        width: 200px;
        height: 200px;
        background-color: rgb(78, 177, 111);
        transition: 0.3s;
      }
      body:hover .box1{
            /* perspective(200px)设置基础视距 */
            transform: perspective(200px) translateZ(100px);
            transition: transform 0.3s;
      }
```

* 旋转

```css
      body:hover .box1{
        /* 旋转 */
        transform: rotateZ(360deg);
      }
```

* 缩放

  ```css
          /* 放大到1.2倍 */
          transform: scale(1.2)
  ```

## 移动端适配

### Viewport

* **什么是Viewport?**

手机浏览器是把页面放在一个虚拟的“窗口”（viewport）中，通常这个虚拟的“窗口”（viewport）比屏幕宽，这样就不用把每个网页挤到很小的窗口中（这样会破坏没有针对手机浏览器优化的网页的布局），用户可以通过平移和缩放来看网页的不同部分。移动版的 Safari 浏览器最新引进了 viewport 这个 meta tag，让网页开发者来控制 viewport 的大小和缩放，其他手机浏览器也基本支持。

* **viewport的设置**

width：控制 viewport 的大小，可以指定的一个值，如果 600，或者特殊的值，如 device-width 为设备的宽度（单位为缩放为 100% 时的 CSS 的像素）
height：和 width 相对应，指定高度
initial-scale：初始缩放比例，也即是当页面第一次 load 的时候缩放比例
maximum-scale：允许用户缩放到的最大比例
minimum-scale：允许用户缩放到的最小比例
user-scalable：用户是否可以手动缩放

minimal-ui: 最小化浏览器地址栏和菜单

* 示例

```
  <meta name="viewport" content="width=device-width, minimal-ui">
```

### 文本溢出算法

一般的画面，我们将DIV的宽度设置为百分比后，其文字内容会自动填充满该宽度，超过的部分自动换行（纯英文不换行，需要用word-break属性设置强制换行）。

随着宽度的减小，文字也自动换行，并且整体高度逐渐增加。然而，在模拟手机端画面显示时，相同的内容，其缩放至一定比例时，文字会突然变大。

这种现象只在手机端浏览器上出现，查找资料发现，是手机端浏览器为了让小屏幕的设备能够看清一些网页上的文字，自动改变了文字的大小，即开头提到的文本溢出算法。而停用这个浏览器行为的方式，就是在CSS中加入text-size-adjust属性，设置完这个属性之后，在移动端浏览器中就不会自动将文字放大了。由于该属性还不是一个标准，所以在书写时最好添加前缀

```css
  -webkit-text-size-adjust: 100%;
  -ms-text-size-adjust: 100%;
```

### 媒体查询样式

```css
@media screen and (max-width: 992px){
    /* 当为屏幕且显示宽度<=992px时启用的css格式 */
    div{
        color: green;
    }
}
```

### 移动端适配的相关属性

```css
  /*控制滚动是否有回弹效果(松开手之后会惯性继续滑动)*/
-webkit-overflow-scrolling: touch;
```

## 不同浏览器的适配

### Firefox按钮边框

```css
/* 去除点击某些元素后边框 */
button:focus { 
    outline: none；
}
/* Firfox使用 */
input::-moz-focus-inner {
    padding: 0;
    border: 0;
}


```

## 一些不会分类的属性

### filter___滤镜效果

CSS属性 filter 将模糊或颜色偏移等图形效果应用于元素。滤镜通常用于调整图像、背景和边框的渲染

```css
.filter{
    /* 高斯模糊 */
    filter: blur(5px);
    /* 灰度滤镜 */
    filter: grayscale(100%);
}
```
