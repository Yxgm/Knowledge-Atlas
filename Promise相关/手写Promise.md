```js
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class MPromise {
    FULFILLED_CALLBACK_LIST = [];
    REJECTED_CALLBACK_LIST = [];
    _status = PENDING;
    constructor(fn) {
        // 初始状态为pending
        this.status = PENDING;
        this.value = null;
        this.reason = null;
        try {fn(this.resolve.bind(this), this.reject.bind(this))} 
        catch (e) { this.reject(e)}
    }

    get status() {return this._status }
    set status(newStatus) {
        this._status = newStatus;
        switch (newStatus) {
            case FULFILLED: {
                this.FULFILLED_CALLBACK_LIST.forEach(callback => callback(this.value));
                break;
            }
            case REJECTED: {
                this.REJECTED_CALLBACK_LIST.forEach(callback => callback(this.reason));
                break;
            }
        }
    }

    resolve(value) {
        if (this.status === PENDING) {
            this.value = value;
            this.status = FULFILLED;
        }
    }

    reject(reason) {
        if (this.status === PENDING) {
            this.reason = reason;
            this.status = REJECTED;
        }
    }

    then(onFulfilled, onRejected) {//then方法
  		const realOnFulfilled = this.isFunction(onFulfilled) ? onFulfilled : (value) => value
  		const realOnRejected = this.isFunction(onRejected) ? onRejected : (reason) => throw reason
     
        const promise2 = new MPromise((resolve, reject) => {
            const fulfilledMicrotask = () => {
                queueMicrotask(() => {
                    try {
                        const x = realOnFulfilled(this.value);
                        this.resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e)
                    }
                })
            };
            const rejectedMicrotask = () => {
                queueMicrotask(() => {
                    try {
                        const x = realOnRejected(this.reason);
                        this.resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                })
            }

            switch (this.status) {
                case FULFILLED: {
                    fulfilledMicrotask()
                    break;
                }
                case REJECTED: {
                    rejectedMicrotask()
                    break;
                }
                case PENDING: {
                    this.FULFILLED_CALLBACK_LIST.push(fulfilledMicrotask)
                    this.REJECTED_CALLBACK_LIST.push(rejectedMicrotask)
                }
            }
        })
        return promise2

    }

    catch (onRejected) {return this.then(null, onRejected) }

    isFunction(param) {return typeof param === 'function'}

    resolvePromise(promise2, x, resolve, reject) {
        // 如果 newPromise 和 x 指向同一对象，以 TypeError 为据因拒绝执行 newPromise
        // 这是为了防止死循环
        if (promise2 === x) {
            return reject(new TypeError('The promise and the return value are the same'));
        }

        if (x instanceof MPromise) {
            // 如果 x 为 Promise ，则使 newPromise 接受 x 的状态
            // 也就是继续执行x，如果执行的时候拿到一个y，还要继续解析y
            queueMicrotask(() => {
                x.then((y) => {
                    this.resolvePromise(promise2, y, resolve, reject);
                }, reject);
            })
        } else if (typeof x === 'object' || this.isFunction(x)) {
            // 如果 x 为对象或者函数
            if (x === null) {
                // null也会被判断为对象
                return resolve(x);
            }

            let then = null;

            try {
                // 把 x.then 赋值给 then 
                then = x.then;
            } catch (error) {
                // 如果取 x.then 的值时抛出错误 e ，则以 e 为据因拒绝 promise
                return reject(error);
            }

            // 如果 then 是函数
            if (this.isFunction(then)) {
                let called = false;
                // 将 x 作为函数的作用域 this 调用
                // 传递两个回调函数作为参数，第一个参数叫做 resolvePromise ，第二个参数叫做 rejectPromise
                try {
                    then.call(
                        x,
                        // 如果 resolvePromise 以值 y 为参数被调用，则运行 resolvePromise
                        (y) => {
                            // 需要有一个变量called来保证只调用一次.
                            if (called) return;
                            called = true;
                            this.resolvePromise(promise2, y, resolve, reject);
                        },
                        // 如果 rejectPromise 以据因 r 为参数被调用，则以据因 r 拒绝 promise
                        (r) => {
                            if (called) return;
                            called = true;
                            reject(r);
                        });
                } catch (error) {
                    // 如果调用 then 方法抛出了异常 e：
                    if (called) return;

                    // 否则以 e 为据因拒绝 promise
                    reject(error);
                }
            } else {
                // 如果 then 不是函数，以 x 为参数执行 promise
                resolve(x);
            }
        } else {
            // 如果 x 不为对象或者函数，以 x 为参数执行 promise
            resolve(x);
        }
    }

    static resolve(value) {
        if (value instanceof MPromise) return value;
        return new MPromise((resolve) => {
            resolve(value);
        });
    }

    static reject(reason) {
        return new MPromise((resolve, reject) => {
            reject(reason);
        });
    }
//测试代码
}
const test = new MPromise((resolve, reject) => {
    setTimeout(() => {
        reject(111);
    }, 1000);
}).then((value) => {
    console.log('then');
}).catch((reason) => {
    console.log('catch');
})
```

### race方法手写

```js
static race(promiseList) {
        return new MPromise((resolve, reject) => {
            const length = promiseList.length;
            if (length === 0) return resolve();
            else {
                for (let i = 0; i < length; i++) {
                    MPromise.resolve(promiseList[i]).then( (value) => resolve(value), (reason) => reject(reason) );
                }
            }
        });
    }
```

###  all方法手写

```js
	static all(arrayOfPromise) {
  return new Promise((resolve, reject) => {
    let result = [];
    let count = 0;
    for (let i = 0; i++; i < arr.length) {
      Promise.resolve(arr[i]);//防止有不是promise对象进入
      arr[i]
        .then((value) => {
          count++;
          result[i] = value;
          if (count === arr.length) {
            resolve(result);
          }
        })
        .catch((reason) => {
          reject(reason);
        });
    }
  });
};
// Pay attention to this :
// 1.先使用Promise.resolve()包裹一下，因为数组中不一定都是promise，确保万一
// 2.不要使用res.push方法，这样返回的promise结果数组。顺序是乱的
// 3.不要使用res.length===arr.length判断结果数组是否结果全部返回，（如果最后一个promise
// 先返回了，结果数组长度会等于arr.length）
```

### allSelected方法

```js
function allSeltted(promiseArray) {
    return new Promise(function (resolve, reject) {
        if (!Array.isArray(promiseArray)) {
            return reject(new TypeError('arguments muse be an array'))
        }
        let counter = 0;
        let promiseNum = promiseArray.length;
        let resolvedArray = [];
        for (let i = 0; i < promiseNum; i++) {
            Promise.resolve(promiseArray[i])//防止有不是promise对象进入
                .then((value) => {
                    resolvedArray[i] = {
                        value,
                        status: 'fulfilled'
                    };

                }).catch(reason => {
                    resolvedArray[i] = {
                        reason,
                        status: 'rejected'
                    };
                }).finally(() => {
                    counter++;
                    if (counter == promiseNum) {
                        resolve(resolvedArray)
                    }
                })
        }
    })
}
```

