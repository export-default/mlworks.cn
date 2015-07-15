+++
author = "Ryan Zhao"
categories = ["数值计算"]
date = "2015-07-15T09:58:56+08:00"
description = "关于BLAS、ATLAS与LAPACK的简单介绍。"
keywords = ["BLAS","ATlAS","LAPACK","数值计算","科学计算"]
mathjax = false
tags = ["数值计算"]
title = "BLAS、ATLAS、LAPACK"

+++

很久没有发文了，最近一段时间在UVA刷了一些题，也回顾了一些之前在Java Web上的一些工作。之后的一段时间里，我得主要工作可能是机器学习算法的实现与库的设计上，今天先简单说一下数值计算方面的基础软件包。

<!--more-->

## BLAS (Basic Linear Algebra Subprograms)

从名字可以看出来，[BLAS](http://www.netlib.org/blas/)提供了一套基础的线性代数运算例程。其内部，将这些函数分为三个层级。

Level 1 例程执行标量与向量、向量与向量之间的运算，诸如向量的标量乘法、点乘、范数等。

Level 2 例程执行向量与矩阵之间的运算。

Level 3 例程执行矩阵与矩阵之间的运算。

具体的例程，可以参考[此手册](http://www.netlib.org/blas/blasqr.pdf)。

上述手册是针对Fortran的，C的API于此大同小异。

BLAS虽然是一套可移植且高效的线代库，但是并没有针对具体的及其架构进行优化。当然，机器生产商可能会提供优化后的BLAS供我们使用。此外，还可以使用ATLAS来代替BLAS。

## ATLAS （Automatically Tuned Linear Algebra Software）

同样，[ATLAS](http://math-atlas.sourceforge.net/)的名字表明其是一款可自动调整优化的线性代数库。ATLAS在提供BLAS的Fortran 77 与 C接口的同时，其性能可以与针对机器优化过的BLAS例程相媲美。

因此，可以说ATLAS是BLAS的一种替代方案，提供更加好的例程性能。

## LAPACK (Linear Algebra PACKage)

BLAS只提供基础的线性代数例程，[LAPACK](http://www.netlib.org/lapack/)则提供更高等级的线性代数函数，诸如矩阵分解（LU、QR、SVD等）、线性方程组求解、最小二乘等。LAPACK内部使用BLAS执行基础运算。

## 总结

数值计算是机器学习算法的基础，诸如scikit-learn、spark ml-lib等机器学习库均直接或间接的使用BLAS、ATLAS、LAPACK作为底层的数值计算库。

上述软件包，均有在不同语言下的绑定。此外，关于更具体的使用说明，请期待后续文章。