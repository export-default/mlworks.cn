+++
author = "Ryan Zhao"
categories = []
date = "2015-03-19T15:17:17+08:00"
description = ""
keywords = []
mathjax = false
tags = []
title = "introduction to roc curve"
draft = true
+++

ROC（receiver operating characteristic）曲线和AUC（Area Under the Curve）常用作分类器的性能指标。本文详细介绍了ROC和AUC的概念。

## Confusion Matrix

二类分类器预测的类标与真实类标之间有$2^2=4$种组合，画成表格即：


