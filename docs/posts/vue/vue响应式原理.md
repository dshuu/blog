---
layout: detail
title: Vue原理之数据响应式
tag: 优化
date: "2020-09-01"
---

### 什么是响应式

在三大框架(Angular,React,Vue)之前,Jquery 或者原生 Js 变成的时候，我们通过获取一个 input 值是这么做的

```js
  const form = { title:''}
  document.querySelector('input[name='title']').oninput = (e)=>{
    form.title = e.target.value
  }
```

这样倒是也能够在 input 变化时，让 form 对象的数据变化，也可以做到数据改变时候，改变 dom。但是当业务逻辑越来越复杂，面对的很大的困扰就是拼接字符串的繁琐、DOM 频繁变化的低效率，代码量太大。

不如让我们只需要关注数据的变化，让页面根据我们的数据自己变化，岂不美哉。

MVVM 让我们重点关注在 Model，View 会通过 ViewModel 的数据绑定更新,我们可以自己写一个简单的 Mvvm,最小化的表达这个过程。完整历程可以参考[这里](https://github.com/dshuu/hierarchy/tree/master/pages/vue/%E5%88%86%E5%B8%83%E5%8F%8C%E5%90%91%E7%BB%91%E5%AE%9A)

### Vue2 的响应式

```js
以一个很简单典型的场景来看
const app = new Mvvm(
  el:'#app',
  data(){
    return{
      title:'',
    }
  }
)
//那我们要定义一下Mvvm这个类
class Mvvm{
  constructor(options){
    this.$data = options.data()//为了保持组件间数据的独立
    this.$data = data;
    observe(data);
  }
}
//这里observe是因为对于Object对象，要深度响应，所以递归操作
function observe(data) {
  return new Observe(data);
}
class Observe {
  constructor(data) {
    if (!data || typeof data !== "object") {
      return;
    }
    //在每个key上做劫持
    Object.keys(data).forEach((key) => {
      let val = data[key];
      Object.defineProperty(data, key, {
        get() {
          console.log("val==>", val);
          //在完整流程的vue中，这里应该收集依赖
          return val;
        },
        set(newVal) {
          console.log("newVal==>", newVal);
          if (newVal === val) {
            return;
          }
          val = newVal;
          //在完整流程的vue中，这里应该notify触发compiler过程
          observe(newVal);
        },
      });
    });
  }
}
```

### Vue3 响应式

Vue2 中是通过 DefineProperty 来对每个属性劫持，这样有几个问题：

1.对于数组，不能通过下标来触发变化(push 等方法是重写触发的)

2.性能相比 proxy 不好，只能对对象每个属性劫持，而 proxy 可以通过监听整个对象的变化

这里需要对 ES6 的[Proxy](https://es6.ruanyifeng.com/#docs/proxy)和[Reflect](https://es6.ruanyifeng.com/#docs/reflect)有一个基础的了解

```js
//简略版，源码我还没看QAQ
class Mvvm {
  constructor(options = {}) {
    let data = options.data();
    this.$data = observe(data); //返回的是proxy类型
  }
}
function observe(data) {
  return new Proxy(data, {
    get(target, key) {
      console.log(target, key);
      return Reflect.get(target, key);
    },
    set(target, key, value, receiver) {
      Reflect.set(target, key, value, receiver);
      console.log("render");
    },
  });
}
```
