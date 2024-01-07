---
title: 【光流估计3】Horn–Schunck 稠密光流估计算法
author: eren
date: 2023-10-12 23:53:00 +0800
categories: [Blogging]
tags: [robotic_algorithm]
pin: true
math: true
mermaid: true
---

# Horn–Schunck 稠密光流估计算法

> 根据光流估计的三点基本假设，构造优化问题和对应约束

Horn–Schunck光流算法用一种全局方法估计图像的稠密光流场（即对图像中的每个像素计算光流） 算法基于两个假设

-   灰度不变假设 物体上同一个点在图像中的灰度是不变的，即使物体发生了运动。（这个假设在稳定光照的情况可以满足，但是对于存在高光反射的图像是不成立的）
-   光流场平滑假设 （这个是添加的内容） 假设场景中属于同一物体的像素形成光流场向量应当十分平滑，只有在物体边界的地方才会出现光流的突变，但这只占图像的一小部分。总体来看图像的光流场应当是平滑的。 算法构造了一个能量函数，求光流场的问题转化为求能量函数的最小值。

给定图像序列 $I(x, y, t)$ (表达为像素灰度随像素xy空间和时间t的函数) , 求光流场  $\vec{V}(x, y)$ , 等价于求光流的两个分量  $u(x, y)$  和  $v(x, y)$  定义代价函数

$$
E(u, v)=\iint\left[\left(I_{x} u+I_{y} v+I_{t}\right)^{2}+\alpha^{2}\left(\|\nabla u\|^{2}+\|\nabla v\|^{2}\right)\right] d x d y
$$

其中  $I_{x}$, $I_{y}$, $I_{t}$  分别是图像对  $x, y, t$  的导数
 $\left(I_{x} u+I_{y} v+I_{t}\right)^{2}$  是灰度变化因子  $\left(I_{x} u+I_{y} v=-I_{t}\right.$  就是经典的灰度不变假设  ) 
 $\alpha^{2}\left(\|\nabla u\|^{2}+\|\nabla v\|^{2}\right)$  是平滑因子, 可以理解为限制  $u, v$  分量变化的速度

一个合理的光流估计, 应当是使上述两个因子都尽可能小的光流场

其中，$\alpha$ 值越大，光流越平滑。这是一个泛函的极值问题，可以用欧拉-拉格朗日方程求解。对应上式的是双变量双函数一阶导数的欧拉-拉格朗日方程组

$$
\begin{array}{l}
\frac{\partial L}{\partial u}-\frac{\partial}{\partial x} \frac{\partial L}{\partial u_{x}}-\frac{\partial}{\partial y} \frac{\partial L}{\partial u_{y}}=0 \\
\frac{\partial L}{\partial v}-\frac{\partial}{\partial x} \frac{\partial L}{\partial v_{x}}-\frac{\partial}{\partial y} \frac{\partial L}{\partial v_{y}}=0
\end{array}
$$

其中  $L=\left(I_{x} u+I_{y} v+I_{t}\right)^{2}+\alpha^{2}\left(\|\nabla u\|^{2}+\|\nabla v\|^{2}\right)$ , 求导后可得

$$
\begin{aligned}
I_{x}\left(I_{x} u+I_{y} v+I_{t}\right)-\alpha^{2} \Delta u & =0 \\
I_{y}\left(I_{x} u+I_{y} v+I_{t}\right)-\alpha^{2} \Delta v & =0
\end{aligned}
$$

 $\Delta$  是拉普拉斯算子, 定义为  $\Delta=\frac{\partial^{2}}{\partial x^{2}}+\frac{\partial^{2}}{\partial y^{2}}$ , 实际运用中可以用近似算法  $\Delta u(x, y)=\bar{u}(x, y)-u(x, y)$ , 论文 [1]中使用了以下模板来计算  $\Delta u$ 

$$
\frac{1}{12}\left(\begin{array}{ccc}
1 & 2 & 1 \\
2 & -12 & 2 \\
1 & 2 & 1
\end{array}\right)
$$

用  $\Delta u(x, y)=\bar{u}(x, y)-u(x, y)$  替换后, 方程组变化为

$$
\begin{array}{l}
\left(I_{x}^{2}+\alpha^{2}\right) u+I_{x} I_{y} v=\alpha^{2} \bar{u}-I_{x} I_{t} \\
I_{x} I_{y} u+\left(I_{y}^{2}+\alpha^{2}\right) v=\alpha^{2} \bar{v}-I_{y} I_{t}
\end{array}
$$

这是一个线性方程组, 但是由于需要首先计算  $\bar{u}$  和  $\bar{v}$ , 因此可以使用迭代法来求解, 迭代公式为

$$
\begin{aligned}
u^{k+1} & =\bar{u}^{k}-\frac{I_{x}\left(I_{x} \bar{u}^{k}+I_{y} \bar{v}^{k}+I_{t}\right)}{\alpha^{2}+I_{x}^{2}+I_{y}^{2}} \\
v^{k+1} & =\bar{v}^{k}-\frac{I_{y}\left(I_{x} \bar{u}^{k}+I_{y} \bar{v}^{k}+I_{t}\right)}{\alpha^{2}+I_{x}^{2}+I_{y}^{2}}
\end{aligned}
$$
