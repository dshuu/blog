---
layout: detail
title: 怎么让Webpack编译更快
tag: webpack 工程化
date: "2020-09-01"
---

### 不打包最快

听起来有点搞事情，事实就是如此

### 还有呢

#### DLL优化
通常我们引用到的一些三方库，是不需要重复打包的，你怎么打包都是那个样子，那我们可以通过DLL处理，这样每次后面打包都不需要再打包。