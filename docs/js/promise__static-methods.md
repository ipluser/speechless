# Promise原理分析二
前面我们分析了Promise的`then`和`catch`方法，接下来我们一起来看看`reject`、`resolve`、`race`和`all`方法的实现。

<br/>
**Note:**

[Promise原理分析一](http://ipluser.github.io/speechless/#docs/js/promise__then-catch.md)


## reject
### 说明
`Promise.reject(reason)`方法返回一个被拒绝的Promise对象。

语法

```js
Promise.reject(new Error('something wrong')).then(null, err => {
  // todo
});

Promise.reject(new Error('something wrong')).catch(err => {
  // todo
});
```

参数

| name | desc |
|------|------|
| reason | 被拒绝的原因。 |

### 实现
创建一个新的Promise对象，通过其构造函数的参数`reject`函数对象将状态变为`rejected`。

```js
static reject(reason) {
  return new Promise((resovle, reject) => {
    reject(reason);
  });
}
```


## resolve
### 说明
`Promise.resolve(value)`方法返回一个以给定值解析后的Promise对象。但如果这个值是个Promise对象，返回的Promise会采用它的最终状态；否则以该值为成功状态返回promise对象。

语法

```js
Promise.resolve(1000).then(value => {
  // todo
});
```

参数

| name | desc |
|------|------|
| value | 用来解析待返回Promise对象的参数。（可以是一个Promise对象） |

### 实现
如果是一个Promise对象，直接返回该值；否则创建一个新的Promise对象，通过其构造函数的参数`resolve`函数对象将状态变为`fulfilled`。

```js
static resolve(value) {
  // 如果为Promise对象，直接返回当前值
  if (value instanceof Promise) {
    return value;
  }

  return new Promise(resovle => {
    resovle(value);
  });
}
```


## race
### 说明
`Promise.race(values)`返回一个Promise对象，这个Promise在`values`中的任意一个Promise被解决或拒绝后，立刻以相同的解决值被解决或以相同的拒绝原因被拒绝。

语法

```js
Promise.race([p1, p2]).then(value => {
  // todo
}, reason => {
  // todo
});
```

参数

| name | desc |
|------|------|
| values | 一个Array对象。 |

### 实现
使用`Promise.resolve`对迭代对象值进行解析，且将新Promise的参数`resolve`和`reject`函数对象传递给`then`方法，以触发新Promise对象的状态转换。

```js
static race(values) {
  // 校验values参数是否为数组
  if (!isArray(values)) {
    return Promise.reject(new Error('Promise.race must be provided an Array'));
  }

  return new Promise((resovle, reject) => {
    values.forEach(value => {  // 遍历迭代对象
      // 使用Promise.resolve解析value值（可能为Promise对象或其他值）
      // 将新Promise对象的resolve, reject传递给解析后的Promise.prototype.then
      Promise.resolve(value).then(resovle, reject);
    });
  });
}
```


## all
### 说明
`Promise.all(values)`方法返回一个Promise对象，该Promise会等`values`参数内的所有值都被resolve后才被resolve，或以`values`参数内的第一个被reject的原因而被reject。

语法

```js
Promise.all([p1, p2]).then(values => {
  // todo
});
```

参数

| name | desc |
|------|------|
| values | 一个Array对象。|

### 实现
通过`Promise.resolve`对迭代对象值进行解析，使用数组记录`values`参数的所有值被解析后的结果，当所有值解析后resolve，并传递其所有解析结果。同时传递reject函数对象给Promise.resolve().then参数，以触发新Promise对象的状态转换。

```js
static all(values) {
  // 校验values参数是否为数组
  if (!isArray(values)) {
    return Promise.reject(new Error('Promise.all must be provided an Array'));
  }

  return new Promise((resolve, reject) => {
    // 如果数组长度为0，直接resolve且结束处理
    if (values.length === 0) {
      resolve([]);
      return;
    }

    const len = values.length;
    // 创建一个数组用来保存values的Promise返回值
    const result = new Array(len);
    let remaining = len;

    // 处理values数组中的值
    function doResolve(index, value) {
      Promise.resolve(value).then(val => {
        // 将解析后的Promise返回值保存在对应索引的结果集中
        result[index] = val;

        // 当values的所有值都解析完后，调用新Promise对象的resolve函数方法，
        // 把所有返回值result传递给后续处理中，且将状态转换为fulfilled。
        if (--remaining === 0) {
          resolve(result);
        }
      }, reject);
    }

    // 迭代values对象，传递其索引位置以确保结果值的顺序
    for (let i = 0; i < len; i++) {
      doResolve(i, values[i]);
    }
  });
}
```


## 总结
`Promise.reject`和`Promise.resolve`通过Promise的构造函数实现状态转变。

`Promise.race`和`Promise.all`使用`Promise.resolve`解析values值，同时通过构造函数的executor参数的函数对象触发Promise的状态转变，其中`Promise.all`使用数组记录返回值、使用索引值以确保其返回值在结果集中的顺序。


## 关键知识点
> [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
>>MDN


## 资源
### [完整代码](https://github.com/ipluser/anole/blob/master/src/promise/index.js)
