+++
author = "Ryan Zhao"
categories = ['Web','CSS']
date = "2015-08-19T13:53:44+08:00"
description = "关于CSS实现等高列的介绍"
keywords = ['等高列','CSS','Flex Display']
mathjax = false
tags = ['Web','CSS']
title = "使用CSS实现等高列的几种策略"
alias = "css-equal-height-columns"

+++


在开发Web应用时，很常见的一种需求是，希望在同一行的几列元素等高。本文记录了三种实现策略：

 1. 使用伪类 + 绝对定位
 2. 使用table display
 3. 使用flex display
 
<!--more-->

### 使用伪类 + 绝对定位
主要思想是，使用before伪类，创建一个背景图层（使用z-index控制，其背景与列背景相同），并使用绝对定位让其占据父元素的全部高度。以此来实现等高的效果，实际上，元素并不是等高的。
关于这种方法的例子和详细解释，请参考 http://webdesign.tutsplus.com/tutorials/quick-tip-solving-the-equal-height-column-conundrum--cms-20403。

### 使用 table display

将父元素设为table display，子元素设为 table-cell display。此法利用了table cell之间的等高特性。

### 使用 flex display

关于flex的介绍，可以参考 https://css-tricks.com/snippets/css/a-guide-to-flexbox/。flex display的父元素， 可以控制其子元素的扩展与伸缩，因此只需要设置父元素为flex，并将flex-grow设为1。子元素则自动与父元素等高，因此，列之间也变为等高了。

关于使用table display 和 flex display的例子，可以参考 http://stackoverflow.com/questions/19695784/how-can-i-make-bootstrap-columns-all-the-same-height。

> Written with [StackEdit](https://stackedit.io/).

