---
title: 手动实现Promise
date: 2020-03-30 20:23:01
tags: [Promise]
---

#### Promise/A+ 规范

[Promise/A+ 规范](http://malcolmyu.github.io/malnote/2015/06/12/Promises-A-Plus/)

#### Promise 声明

首先，Promise 肯定是一个类，我们用 class 来声明。

- 由于 `new Promise((resolve, reject) => {})`，所以传入一个参数(函数)，Promise/A+ 里叫他 executor (奇怪，我找了半天没找到...)，传入就行了。
- executor 里面有两个参数，一个叫 resolve（成功），一个叫 reject（失败）。
- 由于 resolve 和 reject 可执行，所以都是函数，我们用 let 声明。

```
class Promise {
    // 构造器
    constructor(executor) {
        // 成功
        let resolve = () => {};
        // 失败
        let reject = () => {};
        // 立即执行
        executor(resolve, reject);
    }
}
```

#### 解决基本状态

Promise/A+ 有规定：

- Promise 存在三个状态，pending，fulfilled，rejected
- pending 为初始状态（state），并可以转化为 fulfiied 和 rejected
- fulfilled 时，不可转为其他状态，且必须有一个不可改变的值(value)
- rejected 时，不可转为其他状态，且必须有一个不可改变的原因(reason)
- `new Promise((resolve, reject) =>{ resolve(value); })` resolve 成功，接受参数 value，状态改为 fulfilled，不可再改变。
- `new Promise((resove, reject) => { reject(reason); })` reject 失败，接受参数 reason，状态改为 rejected，不可再改变。
- 若是 executor 函数报错，直接执行 reject();

于是根据规定，我们获得一下代码

```
class Promise{
    constructor(executor) {
        // 初始化状态
        this.state = 'pending';
        // 成功的值
        this.value = undefined;
        // 失败的原因
        this.reason = undefined;

        let resolve = (value) => {
            if (this.state === 'pending') {
                // resolve 调用后，state转为成功状态
                this.state = 'fulfilled';
                // 储存成功的值
                this.value = value;
            }
        }

        let reject = (reason) => {
            if (this.state === 'pending') {
                // reject 调用后，state转为失败
                this.state = 'rejected';
                // 储存失败的原因
                this.reason = reason;
            }
        }

        // 如果 executor 失败，直接调用 reject
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
}
```

#### then 方法

**Promise/A+ 规定：Promise 有一个叫做 then 的方法，里面有两个参数：onFulfilled，onRejected，成功有成功的值，失败有失败的原因。**

- 当状态 state 为 fulfilled，则执行 onFulfilled，传入 this.value。当状态 state 为 rejected，则执行 onRejected，传入 this.reason
- 如果 onFulfilled，onRejectred 是函数，则必须分别在 fulfilled，rejected 后被调用，value 或 reason 依次作为他们的第一个参数

```
class Promise {
    constructor(executor) {
        // 初始化状态
        this.state = 'pending';
        // 成功的值
        this.value = undefined;
        // 失败的原因
        this.reason = undefined;

        let resolve = (value) => {
            if (this.state === 'pending') {
                // resolve 调用后，state转为成功状态
                this.state = 'fulfilled';
                // 储存成功的值
                this.value = value;
            }
        }

        let reject = (reason) => {
            if (this.state === 'pending') {
                // reject 调用后，state转为失败
                this.state = 'rejected';
                // 储存失败的原因
                this.reason = reason;
            }
        }

        // 如果 executor 失败，直接调用 reject
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
    // then 有两个参数 onFulfilled, onRejected
    then(onFulfilled, onRejected) {
        // state 为 fulfilled，执行 onFulfilled，传入成功的值
        if (this.state === 'fulfilled') {
            onFulfilled(this.value);
        }
        // state 为 rejected onRejected，传入失败的原因
        if (this.state === 'rejected') {
            onRejected(this.reason);
        }
    }
}
```

#### 解决异步实现

**现在基本可以实现简单的同步代码，但是当 resolve 在 setTimeout 内执行 then 时，state 还是 pending 状态。这时，我们就需要在 then 调用的时候，将成功和失败存到各自的数组，一旦 resolve 或 reject，就调用它们**

类似于发布订阅，先将 then 里面的两个函数储存起来，由于一个 Promise 可以有多个 then，所以存在同一个数组内。

```
// 多个 then 的情况
let p = new Promise();
p.then();
p.then();
```

成功或失败时，forEach 调用它们

```
class Promise {
    constructor(executor) {
        // 初始化状态
        this.state = 'pending';
        // 成功的值
        this.value = undefined;
        // 失败的原因
        this.reason = undefined;
        // 成功时存放的数组
        this.onResolvedCallbacks = [];
        // 失败时存放的数组
        this.onRejectedCallbacks = [];

        let resolve = (value) => {
            if (this.state === 'pending') {
                // resolve 调用后，state转为成功状态
                this.state = 'fulfilled';
                // 储存成功的值
                this.value = value;
                // 一旦 resolve 执行，调用成功数组的函数
                this.onResolvedCallbacks.forEach(fn => fn());
            }
        }

        let reject = (reason) => {
            if (this.state === 'pending') {
                // reject 调用后，state转为失败
                this.state = 'rejected';
                // 储存失败的原因
                this.reason = reason;
                // 一旦 reject 执行，调用失败数组的函数
                this.onRejectedCallbacks.forEach(fn => fn());
            }
        }

        // 如果 executor 失败，直接调用 reject
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
    // then 有两个参数 onFulfilled, onRejected
    then(onFulfilled, onRejected) {
        // state 为 fulfilled，执行 onFulfilled，传入成功的值
        if (this.state === 'fulfilled') {
            onFulfilled(this.value);
        }
        // state 为 rejected onRejected，传入失败的原因
        if (this.state === 'rejected') {
            onRejected(this.reason);
        }
        // state 为 pending 时
        if (this.state === 'pending') {
            this.onResolvedCallbacks.push(() => {
                onFulfilled(this.value);
            });
            this.onRejectedCallbacks.push(() => {
                onRejected(this.reason);
            });
        }
    }
}
```

#### 解决链式调用

**我们常常用到 `new Promise().then().then()`，这就是链式调用，用来解决回调地狱**
1、为了达成链式，我们默认在第一个 then 里面返回一个 Promise。Promise/A+ 规定了一个方法，就是在 then 里面返回一个新的 Promise，成为 promise2：`promise2 = new Promise((resolve, reject) => {})`

- 将这个 promise2 返回的值传递到下一个 then 中
- 如果返回一个普通的值，则将普通的值传递到下一个 then 中

2、当我们在第一个 then 中 return 了一个参数(参数未知，需判断)。这个 return 出来的新的 promise 就是 onFulfilled() 或 onRejected() 的值

Promise/A+ 规定 onFulfilled() 或 onRejected() 的值，叫做 x，判断 x 的函数叫做 resolvePromise

- 首先，要看 x 是不是 promise
- 如果时 promise，则取它的结果，作为新的 promise2 成功的结果
- 如果是普通值，直接作为 promise2 成功的结果
- 所以要比较 x 和 promise2 是不是相等
- resolvePromise 的参数有 promise2，x，resolve，reject
- resolve 和 reject 是 promise2 的

```
class Promise {
    constructor(executor) {
        // 初始化状态
        this.state = 'pending';
        // 成功的值
        this.value = undefined;
        // 失败的原因
        this.reason = undefined;
        // 成功时存放的数组
        this.onResolvedCallbacks = [];
        // 失败时存放的数组
        this.onRejectedCallbacks = [];

        let resolve = (value) => {
            if (this.state === 'pending') {
                // resolve 调用后，state转为成功状态
                this.state = 'fulfilled';
                // 储存成功的值
                this.value = value;
                // 一旦 resolve 执行，调用成功数组的函数
                this.onResolvedCallbacks.forEach(fn => fn());
            }
        }

        let reject = (reason) => {
            if (this.state === 'pending') {
                // reject 调用后，state转为失败
                this.state = 'rejected';
                // 储存失败的原因
                this.reason = reason;
                // 一旦 reject 执行，调用失败数组的函数
                this.onRejectedCallbacks.forEach(fn => fn());
            }
        }

        // 如果 executor 失败，直接调用 reject
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
    // then 有两个参数 onFulfilled, onRejected
    then(onFulfilled, onRejected) {
        let promise2 = new Promise((resolve, reject) => {
            // state 为 fulfilled，执行 onFulfilled，传入成功的值
            if (this.state === 'fulfilled') {
                let x = onFulfilled(this.value);
                resolvePromise(promise2, x, resolve, reject);
            }
            // state 为 rejected onRejected，传入失败的原因
            if (this.state === 'rejected') {
                let x = onRejected(this.reason);
                resolvePromise(promise2, x, resolve, reject);
            }
            // state 为 pending 时
            if (this.state === 'pending') {
                this.onResolvedCallbacks.push(() => {
                    let x = onFulfilled(this.value);
                    resolvePromise(promise2, x, resolve, reject);
                });
                this.onRejectedCallbacks.push(() => {
                    let x = onRejected(this.reason);
                    resolvePromise(promise2, x, resolve, reject);
                });
            }
        });
        return promise2;
    }
}
```

#### 完成 resovlePromise 函数

Promise/A+ 规定：

1. 如果 x 与 promise2 指向同一对象，会造成循环引用，自己等待自己完成，则以 `TypeError` 为据因拒绝执行 promise

循环引用实例

```
let p = new Promise(resolve => {
    resolve(1);
})
let p2 = p.then(resolve => {
    // 循环引用，自己等待自己完成
    return p2;
})
```

2. 如果 x 为Promise，则使用 promise 接受 x 的状态

3. 否则，x 为对象或函数

- x 不能是 null
- x.then 赋值给 then
- 如果取 x.then 的值时抛出错误 e，则以 e 为据因拒绝 promise
- 如果 then 是一个函数，则用 call 执行 then
- 如果成功的回调还是 promise，就递归继续解析
- x 是普通值，则直接 resolve(x)
- 成功和失败只能调用一个，所以设定一个 called 来防止多次调用

```
function resolvePromise(promise2, x, resolve, reject) {
    // 循环引用直接报错
    if (promise2 === x) {
        return reject(new TypeError('Chaining cycle detected for promise'));
    }

    // 防止多次调用
    let called = false;
    if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
        try {
            let then = x.then;
            if (typeof then === 'function') {
                then.call(x, y => {
                    if (called) return;
                    called = true;
                    resolvePromise(promise2, y, resolve, reject);
                }, r => {
                    if (called) return;
                    called = true;
                    reject(r);
                })
            } else {
                resolve(x);
            }
        } catch(e) {
            if (called) return;
            called = true;
            reject(e);
        }
    } else {
        resolve(x);
    }
}
```

#### 解决其他问题

Promise/A+ 规定 onFulfilled，onRejected 都是函数，如果不是，则直接被忽略。这里其实应该说如果不是则变为函数，成功时将 value 传递给下一个 then 中的 onFulfilled 中，失败时，直接抛出错误。

- onFulfilled 返回一个普通的值，直接等于 value => value
- onRejected 返回一个普通的值，直接抛出错误 reason => throw reason
- onFulfilled 或 onRejected 不能同步调用，所以使用 setTimeout 解决

```
class Promise {
    constructor(executor) {
        // 初始化状态
        this.state = 'pending';
        // 成功的值
        this.value = undefined;
        // 失败的原因
        this.reason = undefined;
        // 成功时存放的数组
        this.onResolvedCallbacks = [];
        // 失败时存放的数组
        this.onRejectedCallbacks = [];

        let resolve = (value) => {
            if (this.state === 'pending') {
                // resolve 调用后，state转为成功状态
                this.state = 'fulfilled';
                // 储存成功的值
                this.value = value;
                // 一旦 resolve 执行，调用成功数组的函数
                this.onResolvedCallbacks.forEach(fn => fn());
            }
        }

        let reject = (reason) => {
            if (this.state === 'pending') {
                // reject 调用后，state转为失败
                this.state = 'rejected';
                // 储存失败的原因
                this.reason = reason;
                // 一旦 reject 执行，调用失败数组的函数
                this.onRejectedCallbacks.forEach(fn => fn());
            }
        }

        // 如果 executor 失败，直接调用 reject
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
    // then 有两个参数 onFulfilled, onRejected
    then(onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
        onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason };
        let promise2 = new Promise((resolve, reject) => {
            // state 为 fulfilled，执行 onFulfilled，传入成功的值
            if (this.state === 'fulfilled') {
                setTimeout(() => {
                    try {
                        let x = onFulfilled(this.value);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch(e) {
                        reject(e);
                    }
                })
            }
            // state 为 rejected onRejected，传入失败的原因
            if (this.state === 'rejected') {
                setTimeout(() => {
                    try {
                        let x = onRejected(this.reason);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch(e) {
                        reject(e);
                    }
                })
            }
            // state 为 pending 时
            if (this.state === 'pending') {
                this.onResolvedCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onFulfilled(this.value);
                            resolvePromise(promise2, x, resolve, reject);
                        } catch(e) {
                            reject(e);
                        }
                    })
                });
                this.onRejectedCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onRejected(this.reason);
                            resolvePromise(promise2, x, resolve, reject);
                        } catch(e) {
                            reject(e);
                        }
                    })
                });
            }
        });
        return promise2;
    }
}
```

顺便实现 catch、resolve、reject、race、all方法

```
class Promise {
    constructor(executor) {
        // 初始化状态
        this.state = 'pending';
        // 成功的值
        this.value = undefined;
        // 失败的原因
        this.reason = undefined;
        // 成功时存放的数组
        this.onResolvedCallbacks = [];
        // 失败时存放的数组
        this.onRejectedCallbacks = [];

        let resolve = (value) => {
            if (this.state === 'pending') {
                // resolve 调用后，state转为成功状态
                this.state = 'fulfilled';
                // 储存成功的值
                this.value = value;
                // 一旦 resolve 执行，调用成功数组的函数
                this.onResolvedCallbacks.forEach(fn => fn());
            }
        }

        let reject = (reason) => {
            if (this.state === 'pending') {
                // reject 调用后，state转为失败
                this.state = 'rejected';
                // 储存失败的原因
                this.reason = reason;
                // 一旦 reject 执行，调用失败数组的函数
                this.onRejectedCallbacks.forEach(fn => fn());
            }
        }

        // 如果 executor 失败，直接调用 reject
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
    // then 有两个参数 onFulfilled, onRejected
    then(onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
        onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason };
        let promise2 = new Promise((resolve, reject) => {
            // state 为 fulfilled，执行 onFulfilled，传入成功的值
            if (this.state === 'fulfilled') {
                setTimeout(() => {
                    try {
                        let x = onFulfilled(this.value);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch(e) {
                        reject(e);
                    }
                })
            }
            // state 为 rejected onRejected，传入失败的原因
            if (this.state === 'rejected') {
                setTimeout(() => {
                    try {
                        let x = onRejected(this.reason);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch(e) {
                        reject(e);
                    }
                })
            }
            // state 为 pending 时
            if (this.state === 'pending') {
                this.onResolvedCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onFulfilled(this.value);
                            resolvePromise(promise2, x, resolve, reject);
                        } catch(e) {
                            reject(e);
                        }
                    })
                });
                this.onRejectedCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onRejected(this.reason);
                            resolvePromise(promise2, x, resolve, reject);
                        } catch(e) {
                            reject(e);
                        }
                    })
                });
            }
        });
        return promise2;
    }
    // 实现 catch 方法
    catch(fn) {
        return this.then(null, fn);
    }
}
function resolvePromise(promise2, x, resolve, reject) {
    // 循环引用直接报错
    if (promise2 === x) {
        return reject(new TypeError('Chaining cycle detected for promise'));
    }

    // 防止多次调用
    let called = false;
    if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
        try {
            let then = x.then;
            if (typeof then === 'function') {
                then.call(x, y => {
                    if (called) return;
                    called = true;
                    resolvePromise(promise2, y, resolve, reject);
                }, r => {
                    if (called) return;
                    called = true;
                    reject(r);
                })
            } else {
                resolve(x);
            }
        } catch(e) {
            if (called) return;
            called = true;
            reject(e);
        }
    } else {
        resolve(x);
    }
}
// 实现 resolve 方法
Promise.resolve = function(val) {
    return new Promise((resolve, reject) => {
        resolve(val);
    })
}
// 实现 reject 方法
Promise.reject = function(reason) {
    return new Promise((resolve, reject) => {
        reject(reason);
    })
}
// 实现 race 方法
Promise.race = function(promises) {
    return new Promise((resolve, reject) => {
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(resolve, reject);
        }
    })
}
// 实现 all 方法(获取所有的 promise，都执行 then，把结果放到数组，一起返回)
Promise.all = function(promises) {
    let arr = [];
    let i = 0;
    function processData(index, data) {
        arr[index] = data;
        i++;
        if (i === promise.length) {
            resolve(arr);
        }
    }
    return new Promise((resolve, reject) => {
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(data => {
                processData(i, data);
            }, reject);
        }
    })
}
```

#### 验证我们的 Promise 是否正确

- 安装 `npm install -g promises-aplus-tests`

- 命令行 `promises-aplus-test [js文件名].js`

- 添加如下代码

```
Promise.defer = Promise.deferred = function () {
  let dfd = {}
  dfd.promise = new Promise((resolve,reject)=>{
    dfd.resolve = resolve;
    dfd.reject = reject;
  });
  return dfd;
}
module.exports = Promise;
```
