# Promise-limit(限制并发数量promise)

```js
function promiseLimit(arr, maxCount) {
  let current = 0;
  let callback = [];
  for (let i = 0; i < arr.length; i++) {
    doSend(arr[i]);
  }

  function doSend(item) {
    if (current < maxCount) {
      current++;
      loadImg(item).then(() => {
        current--;
        if (callback.length > 0) {
          doSend(callback.shift());
        }
      });
    } else {
      callback.push(item);
    }
  }
}
const urls = [
  { info: 'link1', time: 3000 },
  { info: 'link2', time: 2000 },
  { info: 'link3', time: 5000 },
  { info: 'link4', time: 1000 },
  { info: 'link5', time: 1200 },
  { info: 'link6', time: 2000 },
  { info: 'link7', time: 800 }
];

// 设置我们要执行的任务
function loadImg(url) {
  return new Promise((resolve, reject) => {
    console.log('----' + url.info + ' start!');
    setTimeout(() => {
      console.log(url.info + ' OK!!!');
      resolve();
    }, url.time);
  });
}

promiseLimit(urls, 3);
```

