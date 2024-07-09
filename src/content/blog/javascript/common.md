---
title: '常见函数实现'
description: '常见函数实现'
pubDate: 'Mar 03 2024'
updatedDate: 'Jul 09 2024'
heroImage: '/blog-placeholder-3.jpg'
# finished: false
---

## compose

```js
// koa有个特点，调用next参数表示调用下一个函数
function fn1(next) {
  console.log(1, 'before next');
  next();
  console.log(1, 'after next');
}

function fn2(next) {
  console.log(2, 'before next');
  next();
  console.log(2, 'after next');
}

function fn3(next) {
  console.log(3, 'before next');
  next();
  console.log(3, 'after next');
}

const middleware = [fn1, fn2, fn3];

function compose(middleware) {
  function dispatch(index) {
    if (index == middleware.length) return;
    const curr = middleware[index];
    // 这里使用箭头函数，让函数延迟执行
    return curr(() => dispatch(++index));
  }
  dispatch(0);
}

compose(middleware);
```

## promisify

将函数回调形式转为promise形式，现有`loadImage`函数

```js
const imageSrc = 'https://xxx';

function loadImage(src, callback) {
  const image = document.createElement('img');
  image.src = src;
  image.style = 'width: 200px;height: 200px';
  image.onload = () => callback(null, image);
  image.onerror = () => callback(new Error('加载失败'));
  document.body.append(image);
}
```

通用promisify函数

```js
function promisify(original) {
  function fn(...args) {
    return new Promise((resolve, reject) => {
      args.push(function (err, success) {
        if (err) {
          reject(err);
        } else {
          resolve(success);
        }
      });

      // original.apply(this, args);
      Reflect.apply(original, this, args);
    });
  }

  return fn;
}
```

## scheduler
