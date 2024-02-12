# 1. 内置组件

## 1. 组件基础

_讲一下 Unity 组件的生命周期?_

![1703219493521](image/unity/1703219493521.png)

_Update 和 FixedUpdate 的区别?_

```
1. Update每一帧执行一次

2. FixedUpdate每隔固定时间Time.fixedDeltaTime执行一次
```

_GameObject 中不同组件执行的顺序?_

```
>> 相同优先级: 栈顶先执行, 栈底最后执行(transform组件最后执行)

>> 不同优先级: 优先级高的先执行
```

## 2. 图像与音频

_讲一下 Unity 中的图层?_

```
>> 一个组件属于一个图层, 最多可以有32个图层

>> 组件默认属于Default Layer

>> 摄像头可以筛选可见的图层
```

_实时灯光和烘焙灯光的区别?_

_摄像机的深度表示什么?_

_将一下 AudioListener 和 AudioSource?_

_渲染器纹理是什么?_

_如何将用户鼠标的点击映射到刚体上?_

## 3. 地形

## 4. 角色和刚体

_控制角色移动有哪些方式?_

```
1. Transform.Translate方法
>> 该方法不考虑碰撞和重力

2. CharacterController组件
>> 考虑碰撞|重力|上坡

3. Rigidbody组件
>> 考虑碰撞|重力|中心|摩擦等所有刚体因素
```

_计算移动距离时为什么采用 deltaTime 乘以速度?_

```
避免不同帧率下移动速度不一致
```

_离散碰撞检测和连续碰撞检测的区别?_

_碰撞和触发的区别?_

_什么是铰链?_

## 5. 动画和特效

## 6. 导航

## 6. Unity Editor

_什么是 Gizmos?_

_如何快速复制一个物体并同时修改所有复制体?_

```
使用prefab, 一个prefab相当于一个类, 修改类后可以应用到所有实体
```

_如何让一个物体吸附到另一个物体上?_

```
选中物体后按住V键, 移动物体直到顶点吸附后松开V键
```

如何快速跳转到物体所在位置?

```
双击物体对应的文件或点击场景中的物体再按F键
```

_如何打开控制台?_

```
Ctrl + Shift + C
```

# 2. UnityEngine API

## 1. GameObject 和 Component

_如何获取同场景中的一个 GameObject?_

```
1. 通过路径|名称获取
>> GameObject.Find()

2. 通过Tag获取
>> GameObject.FindWithTag()
```

_如何获取一个 GameObject 关联的组件?_

```
使用Component#GetComponent()

tips: MonoBehaviour继承自Component
```

_如何创建和销毁一个 GameObject?_

```
Object.Instantiate() 和 Object.Destroy()
```

## 2. Application

_Time.deltaTime 和 Time.fixedDeltaTime 的区别?_

```
1. Time.deltaTime是两帧之间间隔的时间

2. Time.fixedDeltaTime是两次FixedUpdate调用的间隔时间
```

_如何实现慢动作?_

```
修改Time.timeScale
```

## 3. SceneManager & AsyncOperation

_为什么要异步加载场景?_

## 4. Vector & Quaternion & Transform

## 5. Input

_Input 类的作用?_

_什么是虚拟按键和虚拟轴?_

_触摸的阶段 TouchPhase 有哪些?_
