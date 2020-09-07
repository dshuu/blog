---
layout: detail
title: JsCore之类型、内存、垃圾回收
tag: 技术
date: "2020-09-01"
---

### Js 数据类型

首先，要明白的是基础类型和对象(引用)类型，这个是前端最基础的东西。类型之间转换是 Js 中很不容易理解的地方之一了。

#### 原始类型

原始类型值是无法修改的，复制的是值，比较是值的比较。原始类型现在一共有 7 种:Number,String,Boolean,Symbol,BigInt,Null,Undefined

```js
let str = 'abc'
str[1] = 'e'
console.log(str) // 还是'abc',str的值是不会修改，str='aec'是重新赋值
let str1 = str
str1 = 'aec'◊
console.log(str1, str) // str1变为'aec',str还是'abc'
str1 = 'abc'
str === str1 //true,值的比较
```

#### 对象类型

对象类型的赋值是引用指针的赋值，属性可以修改，复制的是指针。比较是指针的比较，所以没有两个对象是相等的。

```js
  [1,2,3] === [1,2,3] //false
  {a:1}==={a:1} //false

  let  obj = {a:1}
  let obj1 = obj
  obj1.a = 2
  obj.a //   1 因为同一个指针指向的对象属性改变了
```

对象拷贝分为[浅拷贝](https://github.com/dshuu/hierarchy/blob/master/pages/js/2.%E5%8F%98%E9%87%8F%E3%80%81%E4%BD%9C%E7%94%A8%E5%9F%9F%E4%B8%8E%E5%86%85%E5%AD%98/%E6%B5%85%E6%8B%B7%E8%B4%9D.html),和[深拷贝](https://github.com/dshuu/hierarchy/blob/master/pages/js/2.%E5%8F%98%E9%87%8F%E3%80%81%E4%BD%9C%E7%94%A8%E5%9F%9F%E4%B8%8E%E5%86%85%E5%AD%98/%E6%B7%B1%E6%8B%B7%E8%B4%9D.html)

#### 类型转换

虽然平时我们不会这么写，且现在 TS 越来越普及，大家都用===了，不过还是要理解下，毕竟说不定什么时候就出这个 bug 了呢。

#### 类型判断

### Js 中的内存

### 垃圾回收机制
