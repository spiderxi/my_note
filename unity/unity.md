# 组件

## 1. 基本组件

***Unity组件本质上是什么?***

```
一个组件本质上是一个MonoBehaviour子类, 通过组件的hook方法中定义GameObject的行为
```

***讲一下Transform组件?***

```
Tranform是一个GameObject组件栈最底层的组件, 也是必备的组件, 组件包含三个属性:

>> 相对于父GameObject坐标系中的属性: position

>> 相对于自身坐标系中的属性: rotation & scale
```

***Material组件的作用?***

```
Material组件用于渲染物体表面的呈现效果, 由多个Shader和颜色等参数组成

Shader: GPU渲染管道的中的程序, 用于给几何体顶点和片元渲染
```

## 2. 组件钩子

***讲一下Unity组件的生命周期?***

![1703219493521](image/unity/1703219493521.png)

***Unity组件栈中的组件的Update方法执行顺序是什么?***

```
栈顶先执行, 栈底最后执行
```
