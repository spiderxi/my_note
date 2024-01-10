# 基本概念

## 1. 组件

**_Unity 组件本质上是什么?_**

```
一个组件本质上是一个MonoBehaviour子类, 通过组件的hook方法中定义GameObject的行为
```

**_讲一下 Transform 组件?_**

```
Tranform是一个GameObject组件栈最底层的组件, 也是必备的组件, 组件包含三个属性:

>> 相对于父GameObject坐标系中的属性: position

>> 相对于自身坐标系中的属性: rotation & scale
```

**_Material 组件的作用?_**

```
Material组件用于渲染物体表面的呈现效果, 由多个Shader和颜色等参数组成

Shader: GPU渲染管道的中的程序, 用于给几何体顶点和片元渲染
```

**_讲一下 Unity 组件的生命周期?_**

![1703219493521](image/unity/1703219493521.png)

**_Unity 中不同组件间组件方法执行的顺序?_**

```
>> 相同优先级: 栈顶先执行, 栈底最后执行

>> 不同优先级: 优先级高的先执行
```

## 2. 效率工具

**_讲一下 Unity 中的图层?_**

```
>> 一个组件属于一个图层, 最多可以有32个图层

>> 组件默认属于Default Layer

>> 摄像头可以筛选可见的图层
```

**_什么是 prefab?_**

```
一个prefab相当于一个类, 修改类后可以应用到所有实体
```

**_UnityEngine 中提供的常用类有哪些?_**

```
>> Debug: Unity控制台输出和辅助线绘制

>> Vector2/3/4: 2/3/4维向量

>> MonoBehaviour

>> Quaternion: 旋转四元组
```
