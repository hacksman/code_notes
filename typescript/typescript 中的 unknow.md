`unknow` 和 `any` 类型的变量都可以被赋给任何值，但是 `unknow` 类型做了一个更加安全的类型校验机制，就是当你定义了一个对象为 `unknow` 类型后，你使用这个 `unknow` 对象之前，你必须使用类型断言或者 `typeof` 判断，得到这个类型后才能够使用这个 `unknow` 对象，以保证类型安全


## 用例
```
function invokeAnything(callback: unknown) {
  // 可以将任何东西赋给 `unknown` 类型，
  // 但在进行类型检查或类型断言之前，不能对 `unknown` 进行操作
  if (typeof callback === 'function') {
    callback();
  }
}

invokeAnything(1); // You can assign anything to `unknown` type
```


## 参考资料
[聊聊TypeScript中unknown和any的差异](https://m.php.cn/article/483451.html)