---
layout: detail
title: web 页面加载性能
tag: 优化
date: "2020-09-01"
---

### 为什么要在乎性能

首先要清楚性能的重要性，快一点慢一点影响大吗？想象一下，一个用户打开竞品 1 秒不到，打开你司网站等了 3 秒，用户选谁？归根结底，技术是要为了业务服务的，也就是性能最终会与利益相关。用户体验不好，可能就直接放弃使用了，且长期内不会再尝试。
下面我们按照用户使用体验，列出四个指标:白屏时间、首屏时间、可操作性时间、总下载时间

### 白屏时间

白屏时间一般是指用户从输入 Url 或者点击跳转后，到首次看到内容(无论是啥)的时间。

> chrome 可以通过特有的 firstPaintTime 来获取首次渲染的时间戳(秒)

```js
const firstPaintTime =
  window.chrome.loadTimes().firstPaintTime * 1000 -
  window.performance.timing.navigationStart;
```

对于非 chrome 浏览器或者低版本浏览器，首次渲染时间，可以通过头部资源加载的时间来统计。根据浏览器渲染原理，那可以通过在头部末尾加上一个时间戳来判断。

```js
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <script>
  const startTime = +new Date
  </script>
</head>
<body>
  <script>
  const endTime = +new Date
  const loadTime = endTime-startTime//最终结果比chrome下少几十ms
  </script>
</body>
</html>
```

### 首屏时间

首屏时间是用户感知中，近似加载完成的时间，包括 iframe，图片等，以最后一个资源完成的时间为准。我们可以以[DOMContentLoaded](https://developer.mozilla.org/zh-CN/docs/Web/Events/DOMContentLoaded)或最后一张图片加载完成的时间来作为指标

```js
function getEleOffsetTop(ele) {
  let offsetTop = ele.offsetTop;
  if (ele.offsetParent !== null) {
    offsetTop += getEleOffsetTop(ele.offsetParent);
  }
  return offsetTop;
}
let domContentLoadedTime,
  firstScreenTime,
  firstScreenImgs = [],
  allImgLoaded = false,
  fistScreenHeight = window.screen.height;
document.addEventListener("DOMContentLoaded", (e) => {
  domContentLoadedTime = +new Date();
  // 找到所有的图片
  let allImgs = document.querySelector("img");
  if (!allImgs.length) {
    //没有图片，首屏时间就是DOM加载解析完成时间，不考虑iframe等情况
    return (firstScreenTime = domContentLoadedTime);
  }
  // 判断图片是否在首屏，这里写的很简单，效率较低
  for (let i = 0; i < allImgs.length; i++) {
    let img = allImgs[i];
    if (!img.src) {
      continue;
    }
    let imgOffsetTop = getEleOffsetTop(img);
    if (imgOffsetTop < fistScreenHeight) {
      firstScreenImgs.push({
        img,
        loaded: false,
      });
    }
  }
  //判断是否都加载完毕
  const timer = setInterval(() => {
    if (firstScreenImgs.length) {
      for (let i = 0; i < firstScreenImgs.length; i++) {
        if (!firstScreenImgs[i].complete) {
          allImgLoaded = false;
          break;
        } else {
          allImgLoaded = true;
        }
      }
    } else {
      allImgLoaded = true;
    }
    if (allImgLoaded) {
      firstScreenTime = +new Date();
      clearInterval(timer);
    }
  }, 0);
});
```

### 可操作和总下载时间

可操作性时间以 domready 为准,基本可以认为是 DOMContentLoaded 的时间。

总下载时间可以统计 onload 的时间,即所有同步资源都加载完成，如果存在很多异步渲染，则需要将所有异步加载完成的时间作为总下载时间。
