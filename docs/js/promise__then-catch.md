# Promise原理分析一
Promise对象用于异步计算。一个Promise对象代表着一个还未完成，但预期将来会完成的操作。

Promise对象有以下几种状态:

- **pending**: 初始状态, 既不是 fulfilled 也不是 rejected.
- **fulfilled**: 成功的操作.
- **rejected**: 失败的操作.

pending状态的Promise对象既可转换为带着一个成功值的fulfilled状态，也可变为带着一个失败信息的 rejected状态。当状态发生转换时，Promise.then绑定的方法就会被调用。(当绑定方法时，如果 Promise对象已经处于fulfilled或rejected状态，那么相应的方法将会被立刻调用，所以在异步操作的完成情况和它的绑定方法之间不存在竞争条件。)

因为Promise.prototype.then和Promise.prototype.catch方法返回Promises对象, 所以它们可以被链式调用。

![状态转换图](../../public/img/js/promise/promises__state-transition.png)


## constructor
### 说明
语法

```js
new Promise(executor);
new Promise(function(resolve, reject) { ... });
```

参数

| name | desc |
|------|------|
| executor | 带有resolve、reject两个参数的函数对象。第一个参数用在处理执行成功的场景，第二个参数则用在处理执行失败的场景。一旦我们的操作完成即可调用这些函数。 |

### 实现
构造函数主要完成状态的初始化，并提供`resolve`和`reject`两个方法给创建者以转变状态。

```js
const utils = require('./utils');
const isFunction = utils.isFunction;

const STATUS = {
  PENDING: 'pending',
  FULFILLED: 'fulfilled',
  REJECTED: 'rejected'
};

const SYMBOL_STATUS = Symbol('status');
const SYMBOL_DATA = Symbol('data');
const SYMBOL_FULFILLED_CALLBACK = Symbol('fulfilledCallback');
const SYMBOL_REJECTED_CALLBACK = Symbol('rejectedCallback');

class Promise {
  constructor(executor) {
    if (!isFunction(executor)) {
      throw Error(`Promise executor ${executor} is not a function`);
    }

    const self = this;
    // 初始状态为pending
    const status = self[SYMBOL_STATUS] = STATUS.PENDING;

    // 使用symbol实现私有化
    self[SYMBOL_FULFILLED_CALLBACK] = undefined;
    self[SYMBOL_REJECTED_CALLBACK] = undefined;
    self[SYMBOL_DATA] = undefined;

    function resovle(value) {
      // todo
    }

    function reject(reason) {
      // todo
    }

    // 执行executor函数，若出现异常则调用reject方法，将状态变为rejected，同时调用回调函数
    try {
      executor(resovle, reject);
    } catch (err) {
      reject(err);
    }
  }
}
```

上面代码中完成了构造函数的雏形，而`resolve`和`reject`两个函数的职责为状态转变和回调函数的调用：

**resolve**，接受一个成功值，传递给绑定的fulfilled回调函数中。主要工作是将当前状态变为`fulfilled`状态，同时调用绑定的fulfilled回调函数：

```js
function resovle(value) {
  const fulfilledCallback = self[SYMBOL_FULFILLED_CALLBACK];

  if (status === STATUS.PENDING) {
    self[SYMBOL_STATUS] = STATUS.FULFILLED;  // 状态转换
    self[SYMBOL_DATA] = value;  // 保存成功值

    if (isFunction(fulfilledCallback)) {
      setTimeout(() => {  // 不阻塞主流程，在下一个事件轮询中再调用fulfilled回调函数
        fulfilledCallback(value);
      });
    }
  }
}
```

**reject**，接受一个失败信息，传递给绑定的rejected回调函数中。主要工作是将当前状态变为`rejected `状态，同时调用绑定的rejected回调函数：

```js
function reject(reason) {
  const rejectedCallback = self[SYMBOL_REJECTED_CALLBACK];

  if (status === STATUS.PENDING) {
    self[SYMBOL_STATUS] = STATUS.REJECTED;  // 状态转换
    self[SYMBOL_DATA] = reason;  // 保存成功值

    if (isFunction(rejectedCallback)) {
      setTimeout(() => {  // 不阻塞主流程，在下一个事件轮询中再调用rejected回调函数
        rejectedCallback(reason);
      });
    }
  }
}
```

fulfilled`和rejected回调函数是通过Promise.prototype.then和Promise.prototype.catch注册的。


## then
### 说明
then方法返回一个Promise。它有两个参数，分别为Promise在成功和失败情况下的回调函数。

语法

```js
p.then(onFulfilled, onRejected);

p.then(function(value) {
  // todo
}, function(reason) {
  // todo
});
```

参数

| name | desc |
|------|------|
| onFulfilled | 一个函数，当Promise的状态为fulfilled时调用。该函数有一个参数，为成功的返回值。 |
| onRejected | 一个函数，当Promise的状态为rejected时调用。该函数有一个参数，为失败的原因。 |

### 实现
根据当前状态对回调函数进行处理，同时返回一个新的Promise对象，以便链式调用。

```js
then(onFulfilled, onRejected) {
  const self = this;
  const status = self[SYMBOL_STATUS];
  const data = self[SYMBOL_DATA];
  const resolveValue = self[SYMBOL_RESOLVE_VALUE];
  // 如果onFulfilled不是函数，回调函数仅返回成功值
  const fulfilledCallback = isFunction(onFulfilled)
      ? onFulfilled
      : function returnFunc(value) { return value; };
  // 如果onRejected不是函数，回调函数仅返回失败信息
  const rejectedCallback = isFunction(onRejected)
      ? onRejected
      : function throwFunc(reason) { throw reason; };
      
  // 当前状态为pending
  if (status === STATUS.PENDING) {
    // 返回一个新的Promise对象，可以被链式调用
    return new Promise((resolve, reject) => {
      // 将fulfilled回调函数注册到当前Promise对象中（非新Promise对象）
      self[SYMBOL_FULFILLED_CALLBACK] = value => {
        // 根据回调函数的执行情况，通过传递的新Promise对象的resolve和reject方法对其状态进行转变
        try {
          const newValue = fulfilledCallback(value);
          // 解析成功值
          resolveValue(newValue, resolve, reject);
        } catch (err) {
          reject(err);
        }
      };

      // 同上
      self[SYMBOL_REJECTED_CALLBACK] = reason => {
        try {
          const newReason = rejectedCallback(reason);
          resolveValue(newReason, resolve, reject);
        } catch (err) {
          reject(err);
        }
      };
    });
  }

  // 当前状态为fulfilled
  if (status === STATUS.FULFILLED) {
    // 返回一个新的Promise对象，可以被链式调用
    return new Promise((resolve, reject) => {
      // 在下一个事件轮询中立即调用fulfilled回调函数，根据执行情况决定新Promise对象的状态转变
      setTimeout(() => {
        try {
          const newValue = fulfilledCallback(data);
          resolveValue(newValue, resolve, reject);
        } catch (err) {
          reject(err);
        }
      });
    });
  }

  // 当前状态为rejected
  if (status === STATUS.REJECTED) {
    // 返回一个新的Promise对象，可以被链式调用
    return new Promise((resolve, reject) => {
      // 在下一个事件轮询中立即调用rejected回调函数，根据执行情况决定新Promise对象的状态转变
      setTimeout(() => {
        try {
          const newReason = rejectedCallback(data);
          resolveValue(newReason, resolve, reject);
        } catch (err) {
          reject(err);
        }
      });
    });
  }
}

// 解析传递值函数
[SYMBOL_RESOLVE_VALUE](value, resolve, reject) {
  // 若传递值为Promise对象，将新Promise对象的resolve和reject传递给Promise传递值以触发状态的转换
  if (value instanceof Promise) {
    value.then(resolve, reject);
    return;
  }

  // 若传递值不是Promise对象，传递给resolve方法
  resolve(value);
}
```

根据当前Promise对象的状态，对回调函数进行注册或立即执行，同时返回一个新的Promise对象。

- **pending**，注册回调函数到当前的Promise对象中
- **fulfilled**或**rejected**，立即执行回调函数


## catch
### 说明
catch方法只处理Promise被拒绝的情况，并返回一个Promise。该方法的行为和调用Promise.prototype.then(undefined, onRejected)相同。

语法

```js
p.catch(onRejected);

p.catch(function(reason) {
   // todo
});
```

参数

| name | desc |
|------|------|
| onRejected | 当Promise被拒绝时调用的函数。该函数调用时会传入一个参数：失败原因。 |

### 实现
catch方法是对then方法的包装，仅传递onRejected失败回调函数。

```js
catch(onRejected) {
  return this.then(null, onRejected);
}
```


## 总结
现在回顾下Promise的实现过程，其主要使用了设计模式中的观察者模式：

- 通过Promise.prototype.then和Promise.prototype.catch方法将观察者方法注册到被观察者Promise对象中，同时返回一个新的Promise对象，以便可以链式调用。
- 被观察者管理内部pending、fulfilled和rejected的状态转变，同时通过构造函数中传递的resolve和reject方法以主动触发状态转变和通知观察者。

<br>
**Note:**

[Promise原理分析二](http://ipluser.github.io/speechless/#docs/js/promise__static-methods.md)


## 关键知识点
> [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
>>MDN

> [symbol](http://es6.ruanyifeng.com/#docs/symbol)
>>ruanyifeng

> [观察者模式](http://baike.baidu.com/link?url=LK58N44Qo3m3uWHkRHhsDklbayVDMG0ozqqKYR_CzlSSxZjYwMl9qpxjPbXTspC5QqXELWKuHKAdHu8Ptjm7L_)
>>百度百科

## 资源
### [完整代码](https://github.com/ipluser/anole/blob/master/src/promise/index.js)
