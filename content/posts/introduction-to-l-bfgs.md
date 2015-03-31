+++
author = "Ryan Zhao"
categories = []
date = "2015-03-30T18:57:20+08:00"
description = ""
keywords = []
mathjax = false
tags = []
title = "introduction to l bfgs"

+++

L-BFGS(Limited-Memory BFGS)是BFGS算法在受限内存时的一种近似算法。而BFGS是数学优化中一种无约束最优化算法。本文的目的是介绍L-BFGS算法的具体原理，在此过程中附加上相关背景知识，力求简单易懂。

## 无约束优化

无约束最优化的基本形式是，给定一个多元函数$f(x)$，求出该函数的最小值点$x^*$。形式化地来说，即：

<div>
$$
x^* = \arg\min_x f(x)
$$
</div>

一般称$f(x)$为目标函数，$x^*$为最优解。在本文中，假定目标函数都是凸函数。由凸函数的性质，可知其局部最优解必定为全局最优解，因此保证接下来介绍的算法必定收敛到局部最优解附近。

## 牛顿法

在使用计算机求解一个优化问题时，通常使用迭代方法。即我们的算法不断迭代，产生一个序列$x_1,x_2 /ldots x_k$，若该序列能收敛到$x^*$，则算法是有效的。由于目标函数（假设为凸函数）存在最小值，若上述序列为递减序列，则最后会收敛到$x^*$（不同的算法，收敛速度有差异）。

假设现在已经有点$x_n$，如何构造$x_{n+1}$，使得$f(x_n) \lt f(x_{n+1}$呢？

牛顿法的基本思想是，在离点$x_n$足够近的距离，$f(x)$可以近似看作一个二次函数。即，在$x_n$附近使用$f(x)$的二次近似来寻找比$x_n$处函数值更小的点。

由[泰勒公式](http://en.wikipedia.org/wiki/Taylor_series)，将$f(x)$在固定点$x_n$处展开，则有：

<div>
$$

\begin{align}
f(x_n + \Delta x)
&\approx f(x_n) + \Delta x^T \nabla f(x_n)  + \frac{1}{2} \Delta x^T \left( \nabla^2 f(x_n) \right)  \Delta x
\end{align}
$$
</div>

其中，$\nabla f(x_n)$和$\nabla^2 f(x_n)$分别为目标函数在点$x_n$处的梯度和[Hessian矩阵](http://en.wikipedia.org/wiki/Hessian_matrix)。当$|| \Delta x || \rightarrow 0$时，上面的近似展开式是成立的。

为了简化符号，我们记

<div>
$$
\begin{align}
h_n(\Delta x) = f(x_n + \Delta x) &= f(x_n) + \Delta x^T \grad_n + \frac{1}{2} \Delta x^T \hessian_n  \Delta x
\end{align}
$$
</div>

其中$\grad_n$和$\hessian_n$分别表示目标函数在点$x_n$处的梯度和[Hessian矩阵](http://en.wikipedia.org/wiki/Hessian_matrix)。假设，我们取$x_{n+1} = x_n + \Delta x$,为了找到比$x_n$处目标函数值更小的点，我们可以取$f(x_n + \Delta x)$的最小值点作为$x_{n+1}$。

因此，当前任务转化为只要找到使$h_n(\Delta x)$最小的点$\Delta x$即可。由于$\hessian_n$为正定矩阵(凸函数任一点的Hessian矩阵为半正定)，$h_n(\Delta x)$也是凸函数，其极小值点即为最小值点。
由下列公式，解出$h_n(\Delta x)$最小值点${\Delta x}^*$

<div>
$$
\begin{align}
\frac{\partial h_n(\Delta x)}{\partial \Delta x} = \grad_n + \hessian_n \Delta x = 0
\end{align}
$$
</div>

<div>
$$
{\Delta x}^* = - \invhessian_n \grad_n
$$
</div>

由此，我们就确定了下一个点$x_{n+1}$的位置。实践上，通常取${\Delta x}^*$作为搜索方向，即取$x_{n+1} = x_n - \alpha (\invhessian_n \grad_n)$，使用一维搜索，找到合适的$\alpha$使得，$f(x_{n+1}$比$f(x)$尽可能小。

下面是算法的伪代码：

<div>
$$
\begin{align}
 & \mathbf{NewtonRaphson}(f,x_0): \\
 & \qquad \mbox{For $n=0,1,\ldots$ (until converged)}: \\
 & \qquad \qquad \mbox{Compute $\grad_n$ and $\invhessian_n$ for $x_n$} \\
 & \qquad \qquad d = \invhessian_n \grad_n \\
 & \qquad \qquad \alpha = \min_{\alpha \geq 0} f(x_{n} - \alpha d) \\
 & \qquad \qquad x_{n+1} \leftarrow x_{n} - \alpha d
\end{align}
$$
</div>

$\alpha$即步长(step-size)的确定可以使用[line search](http://en.wikipedia.org/wiki/Line_search)算法中的任何一种，其中最简单的方法是[backtracking line search](http://en.wikipedia.org/wiki/Backtracking_line_search)。

## 牛顿法的不足

上述的牛顿迭代法最大的问题是需要计算Hessian矩阵的逆。首先，当维度很高时（百万或千万级），此时计算Hessian矩阵的逆几乎是不可能的（存储都很难）。再者，有些函数很难给出Hessian矩阵的解析式。因此，实践上牛顿法很少用在大型的优化问题上。但幸运的是，我们不一定需要一个精确的$\invhessian_n$，一个对其的近似，也可以是我们找到目标函数的递减方向。

## 拟牛顿法

### 基本思想

先看一下拟牛顿法的基本框架

<div>
$$

\begin{align}
& \mathbf{QuasiNewton}(f,x_0, \invhessian_0, \mbox{QuasiUpdate}): \\
& \qquad \mbox{For $n=0,1,\ldots$ (until converged)}: \\
& \qquad \qquad \mbox{// Compute search direction and step-size } \\
& \qquad \qquad d = \invhessian_n \grad_n \\
& \qquad \qquad \alpha \leftarrow \min_{\alpha \geq 0} f(x_{n} - \alpha d) \\
& \qquad \qquad x_{n+1} \leftarrow x_{n} - \alpha d \\
& \qquad \qquad \mbox{// Store the input and gradient deltas } \\
& \qquad \qquad \grad_{n+1} \leftarrow \nabla f(x_{n+1}) \\
& \qquad \qquad s_{n+1} \leftarrow x_{n+1} - x_n \\
& \qquad \qquad y_{n+1} \leftarrow \grad_{n+1} - \grad_n \\
& \qquad \qquad \mbox{// Update inverse hessian } \\
& \qquad \qquad \invhessian_{n+1} \leftarrow \mbox{QuasiUpdate}(\invhessian_{n},s_{n+1}, y_{n+1})
\end{align}

$$
</div>

初始时，给了一个参数$\invhessian_0$，之后每一次迭代通过$\mbox{QuasiUpdate}$方法加上输入变量与梯度的差值($s_n$和$y_n$)作为参数，产生出下一个$\invhessian$的估计。

可以发现，若$\mbox{QuasiUpdate}$每次都返回单位矩阵，则拟牛顿法退化为梯度下降方法(每一次都沿着梯度方向搜索)。

若$\mbox{QuasiUpdate}$能够返回$\nabla^2 f(x_{n+1})$，则拟牛顿法与牛顿法就等价了。

从上述伪代码中可以看出，拟牛顿法仅仅需要函数值和梯度信息，并不需要二阶导信息。


### QuasiUpdate & BFGS

有了上面的算法框架之后，下面的问题就是QuasiUpdate函数如何实现？

由中值定理，我们知道

<div>
$$
(\grad_{n} - \grad_{n-1}) = \hessian_{n}(x_{n} - x_{n-1}) \\
$$
</div>

因此

<div>
$$
\invhessian_{n} \mathbf{y}_{n}   = \mathbf{s}_{n}
$$
</div>

同时，Hessian矩阵是对称矩阵。在这两个条件的基础上，我们希望$\hessian_n$相对于$\hessian_{n-1}$的变化并不大。形式化地讲，即

<div>
$$
\begin{aligned}
\min_{\invhessian} & \hspace{0.5em} \| \invhessian - \invhessian_{n-1} \|^2 \\
\mbox{s.t. } & \hspace{0.5em} \invhessian \mathbf{y}_{n}   = \mathbf{s}_{n} \\
            & \hspace{0.5em} \invhessian \mbox{ is symmetric }
\end{aligned}
$$
</div>

式中的范数$\| \cdot \|$为[Weighted Frobenius Norm](http://mathworld.wolfram.com/FrobeniusNorm.html)。这个式子的解为

<div>
$$
\invhessian_{n+1} = (I - \rho_n y_n s_n^T) \invhessian_n (I - \rho_n s_n y_n^T) + \rho_n s_n s_n^T
$$
</div>

其中$\rho_n = (y_n^T s_n)^{-1}$。

这种更新方式，被称为Broyden–Fletcher–Goldfarb–Shanno (BFGS)更新，由其发明者命名。

值得注意的是，为了计算$\invhessian_{n+1}$，我们只需保存$\invhessian_0$以及{$s_{n-1}$},{$y_{n-1}$}序列即可。下面是计算$\invhessian_n d$的伪代码。

<div>
$$
\begin{align}
& \mathbf{BFGSMultiply}(\invhessian_0, \{s_k\}, \{y_k\}, d): \\
& \qquad r \leftarrow d \\
& \qquad \mbox{// Compute right product} \\
& \qquad \mbox{for $i=n,\ldots,1$}: \\
& \qquad \qquad \alpha_i \leftarrow \rho_{i} s^T_i r \\
& \qquad \qquad r \leftarrow r - \alpha_i y_i \\
& \qquad \mbox{// Compute center} \\
& \qquad r \leftarrow \invhessian_0 r \\
& \qquad \mbox{// Compute left product} \\
& \qquad \mbox{for $i=1,\ldots,n$}: \\
& \qquad \qquad \beta \leftarrow \rho_{i} y^T_i r \\
& \qquad \qquad r \leftarrow r + (\alpha_{n-i+1}-\beta)s_i \\
& \qquad \mbox{return $r$}
\end{align}
$$
</div>

## L-BFGS

BFGS虽然不需要计算Hessian矩阵了，但是保存$s_n$、$y_n$的历史记录仍需要消耗大量的内存。L-BFGS，即限定内存的BFGS算法，其BFGSMultiply仅使用最近的若干次$s_n$、$y_n$记录。