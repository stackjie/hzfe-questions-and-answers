# Vue 双向绑定底层实现原理与源码解析

## 神奇的defineProperty

`defineProperty`是ES5新增的API也是 Vue 实现双向绑定的一个核心，`defineProperty`可以给对象的属性加上`setter`和`getter`访问器，这样就能感知属性的变化。
```js
var obj = {};
Object.defineProperty(obj, 'a',  {
   configurable: true,
   enumerable: true,
   get: function () {
     console.log('a 被get啦!');
   },
   set: function () {
      console.log('a 被set啦!');
   }
});

obj.a;
// 输出 a 被get啦!

obj.a = 'abc';
// 输出 a 被set啦!
```
## 源码分析
