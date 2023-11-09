
# 1. 微积分

## 1. 导数

***导数&偏导&方向导数&梯度分别的含义是什么?***

```
>> 导数是指y随x的变化率

>> 偏导是z在x切面或y切面上的导数

>> 方向导数是任意一个垂直切面上z的导数

>> 梯度是与最大方向导数的方向同方向的一个向量, 单位向量与梯度的内积 === 方向导数
```

$$
\begin{align*}
方向导数的计算公式:\\
f^{'}_{\vec{e}}(x, y) 
&= \lim_{\sqrt{\Delta x^2 + \Delta y^2}\to 0} \frac{f(x+\Delta x, y+\Delta y) - f(x, y)}{\sqrt{\Delta x^2 + \Delta y^2}}\\
&= ......\\
&= \frac{\delta z}{\delta x} \cos{\theta} + \frac{\delta z}{\delta y} \sin{\theta} \qquad (\theta为\vec{e}与x轴的夹角)\\
&= (\frac{\delta z}{\delta x}, \frac{\delta z}{\delta y})^T \cdot(\cos{\theta}, \sin{\theta})\\
&= <梯度\nabla z(x, y). 方向单位向量\vec{e}>
\end{align*}
$$

$$
这里的\nabla是一个记号(又称算子), 用于方便地表示(\frac{\partial}{\partial x},\frac{\partial}{\partial y})
$$

## 2. 求导公式

***常见一元函数的求导公式?***

***多元函数(矩阵表示)的方向导数求导公式?***

## 3. 向量场

***什么是向量场?***

$$
向量场\vec{X}是值域为向量的函数\\
向量场可以看作标量函数组成的向量, 例如:(f_1, f_2)
$$

***什么是向量场的散度(divergence)和旋度?***

散度

$$
向量场\vec{X}的散度
div(\vec{X}) 
\\ = \nabla . \vec{X} 
\\= (\frac{\partial{}}{\partial{x}}, \frac{\partial}{\partial{y}}) . (f_1, f_2) 
\\ = \frac{\partial{f_1}}{\partial{x}} + \frac{\partial{f_2}}{\partial{y}}
$$

> 散度衡量了向量场某一个点周围的向量向外扩散的程度/向内收缩的程度

旋度

$$
向量场\vec{X}的旋度
curl(\vec{X}) 
\\= \nabla \times \vec{X}
\\= (\frac{\partial{}}{\partial{x}}, \frac{\partial}{\partial{y}}) \times (f_1, f_2)
$$

> 旋度衡量了向量场中一个点周围的向量的旋转程度和旋转轴的方向

***什么是拉普拉斯算子?***

$$
拉普拉斯算子\Delta f \\=多元函数f在某一点的领域的函数平均值和这一点的函数值的差
\\= \nabla^2 f = (\frac{\partial}{\partial x^2} + \frac{\partial}{\partial y^2}).f
$$


# 2. 级数

***傅里叶变换的思想?***

***泰勒展开式的核心思想?***

```
泰勒展开式目的是为了使用一个多项式函数去尽可能地逼近任意一个无限可导的函数

核心思想: 如果两个函数在一个点的函数值/任意阶导数相同, 那么这两个函数在这个点的领域直观上就是相同的
```

***单元函数的泰勒展开式和多元函数的区别是什么?***

```
>> 自变量x => 向量X

>> f的导数 => f的梯度 (2阶梯度对应2阶Hessian矩阵)

>> 乘积 => 内积
```

![1699522222419](image/calculus/1699522222419.png)
