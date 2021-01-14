---
title: 手写Promise-ES5函数版与ES6_class版
categories:
  - javascript
tags:
  - 异步
  - ES6
toc: true
donates: false
abbrlink: 32a89cf3
---

>自定义 Promise 的全部逻辑，并加入两个延迟执行的方法

<!-- more -->

## 一、ES5函数版

```js
/**
 * 自定义 Promise 函数模块: IIFE
 * 
 */
(function(window) {
    // 将经常使用的字符串用常量代替
    const PENDING = 'pending'
    const RESOLVED = 'resolved'
    const REJECTED = 'rejected'

    /**
     * Promise 构造函数
     * excutor: 执行器函数(同步执行)
     */
    function Promise(excutor) {
        // 将当前 Promise 保存起来
        const me = this
        me.status = PENDING // 给 promise 对象指定 status 属性，初始值为 pending
        me.data = undefined // 给 promise 对象指定一个用于存储结果数据的属性
        me.callbacks = [] // 每个元素的结构: {onResolved() {}, onRejected() {}}

        function resolve(value) {
            // 如果当前状态不是 pending，直接结束 --- 因为 pending 只能改一次
            if (me.status !== PENDING) {
                return
            }
            // 将状态改为 resolved
            me.status = RESOLVED
                // 保存 value
            me.data = value
                // 如果存在待执行的 callback 函数，立即异步执行回调函数 onResolved
            if (me.callbacks.length > 0) {
                // 模拟异步，执行放入队列中所有成功的回调函数
                setTimeout(() => {
                    me.callbacks.forEach(callbacksObj => {
                        callbacksObj.onResolved(value)
                    });
                })
            }
        }

        function reject(reason) {
            // 如果当前状态不是 pending，直接结束 --- 因为 pending 只能改一次
            if (me.status !== PENDING) {
                return
            }
            // 将状态改为 rejected
            me.status = REJECTED
                // 保存 value
            me.data = reason
                // 如果存在待执行的 callback 函数，立即异步执行回调函数 onRejected
            if (me.callbacks.length > 0) {
                // 模拟异步，执行放入队列中所有成功的回调函数
                setTimeout(() => {
                    me.callbacks.forEach(callbacksObj => {
                        callbacksObj.onRejected(reason)
                    });
                })
            }
        }

        // 立即同步执行 excutor
        // 如果调用失败，则需捕获异常
        try {
            excutor(resolve, reject)
        } catch (error) { // 如果执行器抛出异常，promise 对象变为 rejected 状态
            reject(error)
        }
    }

    /**
     * Promise 原形对象的 then()
     * 指定成功和失败的回调函数
     * 返回一个新的 Promise 对象
     */
    Promise.prototype.then = function(onResolved, onRejected) {
        onResolved = typeof onResolved === 'function' ? onResolved : value => value // 向后传递成功的 value
            // 指定默认的失败的回调(实现错误/异常传透的关键点)
        onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason } // 向后传递失败的 reason

        const me = this

        // 返回一个新的 Promise 对象
        return new Promise((resolve, reject) => {
            /**
             * 调用指定的回调函数处理，根据执行结果，改变 return 的 promise 状态
             */
            function handle(callback) {
                /**
                 * 1.如果抛出异常，return 的 promise 就会失败，reason 就是 error
                 * 2.如果回调函数返回不是 promise，return 的 promise 就会成功，value 就是返回值
                 * 3.如果回调函数返回是 promise，return 的 promise 结果就是这个 promise 的结果
                 */
                try {
                    const result = callback(me.data)
                        // 3.如果回调函数返回是 promise，return 的 promise 结果就是这个 promise 的结果
                    if (result.instanceof(Promise)) {
                        // result.then((resolve, reject) => {
                        //     value => resolve(value),
                        //     reason => reject(reason),
                        // })

                        // 简化
                        result.then(resolve, reject)
                    } else {
                        // 2.如果回调函数返回不是 promise，return 的 promise 就会成功，value 就是返回值
                        resolve(result)
                    }
                } catch (error) {
                    // 1.如果抛出异常，return 的 promise 就会失败，reason 就是 error
                    reject(error)
                }
            }

            // 当前还是 pending 状态，将回调函数保存起来
            if (me.status === PENDING) {
                me.callbacks.push({
                    onResolved(value) {
                        handle(onResolved)
                    },
                    onRejected(reason) {
                        handle(onRejected)
                    }
                })
            } else if (me.status === RESOLVED) { // 如果当前是 resolved 状态，异步执行 onResolve 并改变 return 的Promise 状态
                // 回调函数需异步执行
                setTimeout(() => {
                    handle(onResolved)
                })
            } else { // 果当前是 rejected 状态，异步执行 onRejected 并改变 return 的 Promise 状态
                // 回调函数需异步执行
                setTimeout(() => {
                    handle(onRejected)
                })
            }
        })
    }

    /**
     * Promise 原形对象的 catch()
     * 指定失败的回调函数
     * 返回一个新的 Promise 对象
     */
    Promise.prototype.catch = function(onRejected) {
        return this.then(undefined, onRejected)
    }

    /**
     * Promise 函数对象的 resolve 方法
     * 返回一个指定结果的成功的 Promise
     */
    Promise.resolve = function(value) {
        // 返回一个成功/失败的 promise
        return new Promise((resolve, reject) => {
            // value 是 promise时
            if (value instanceof(Promise)) { // 使用 value 的结果作为 promise 的结果
                value.then(resolve, reject)
            } else { // value 不是 promise 时，promise 成功，数据是 value
                resolve(value)
            }
        })
    }

    /**
     * Promise 函数对象的 reject方法
     * 返回一个指定 reason 的失败的 Promise
     */
    Promise.reject = function(reason) {
        return new Promise((resolve, reject) => {
            reject(reason)
        })
    }

    /**
     * Promise 函数对象的 all 方法
     * 返回一个 promise，只有当所有的 promise 都成功时才成功，否则只要有一个失败的就失败
     */
    Promise.all = function(promises) {
        // 用来保存所有成功 value 的数组
        const values = new Array(promises.length)
            // 用来保存成功的 promise 的数量
        let resolvedCount = 0

        // 返回一个新的 promise
        return new Promise((resolve, reject) => {
            // 遍历 promises 获取每个 promise 的结果
            promises.forEach((p, index) => {
                Promise.resolve(p).then(
                    value => {
                        resolvedCount++ // 成功的数量加1
                        // p 成功，将成功的 value 保存到 values
                        // value.push(p) // 不行,成功的 promise 的顺序也必须是一一对应的
                        values[index] = value

                        // 如果全部成功了，将return的promise改为成功
                        if (resolvedCount === promises.length) {
                            resolve(values)
                        }
                    },
                    reason => { // 只要一个失败了，return 的 promise 就会失败
                        reject(reason)
                    }
                )
            })
        })
    }

    /**
     * Promise 函数对象的 race 方法
     * 返回一个 promise，其结果由第一个完成的 promise 决定
     */
    Promise.race = function(promises) {
        // 返回一个 promise
        return new Promise((resolve, reject) => {
            // 遍历 promises 获取每个 promise 的结果
            promises.forEach((p, index) => {
                Promise.resolve(p).then(
                    value => { // 一旦有成功的，将 return 变为成功
                        resolve(value)
                    },
                    reason => { // 只要一个失败了，return 的 promise 就会失败
                        reject(reason)
                    }
                )
            })
        })
    }

    /**
     * 工具方法 --- resolveDelay(自定义方法)
     * 每返回一个 promise 对象，它在指定的时间后才确定结果
     */
    Promise.resolveDelay = function(value, time) {
        // 返回一个成功/失败的 promise
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                // value 是 promise
                if (value instanceof(Promise)) { // 使用 value 的结果作为 promise 的结果
                    value.then(resolve, reject)
                } else { // value 不是 promise => promise 变为成功，数据是 value
                    resolve(value)
                }
            }, time);
        })
    }

    /**
     * 工具方法 --- rejectDelay(自定义方法)
     * 每返回一个 promise 对象，它在指定的时间后才失败
     */
    Promise.rejectDelay = function(reason, time) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                reject(reason)
            }, time);
        })
    }

    // 向外暴露 Promise 函数
    window.Promise = Promise
})(window)
```



## 二、ES6 class版

```javascript
/**
 * 自定义 Promise 函数模块: class
 * 
 */
(function(window) {
    // 将经常使用的字符串用常量代替
    const PENDING = 'pending'
    const RESOLVED = 'resolved'
    const REJECTED = 'rejected'

    class Promise {
        /**
         * Promise 构造函数
         * excutor: 执行器函数(同步执行)
         */
        constructor(excutor) {
            // 将当前 Promise 保存起来
            const me = this
            me.status = PENDING // 给 promise 对象指定 status 属性，初始值为 pending
            me.data = undefined // 给 promise 对象指定一个用于存储结果数据的属性
            me.callbacks = [] // 每个元素的结构: {onResolved() {}, onRejected() {}}

            function resolve(value) {
                // 如果当前状态不是 pending，直接结束 --- 因为 pending 只能改一次
                if (me.status !== PENDING) {
                    return
                }
                // 将状态改为 resolved
                me.status = RESOLVED
                    // 保存 value
                me.data = value
                    // 如果存在待执行的 callback 函数，立即异步执行回调函数 onResolved
                if (me.callbacks.length > 0) {
                    // 模拟异步，执行放入队列中所有成功的回调函数
                    setTimeout(() => {
                        me.callbacks.forEach(callbacksObj => {
                            callbacksObj.onResolved(value)
                        });
                    })
                }
            }

            function reject(reason) {
                // 如果当前状态不是 pending，直接结束 --- 因为 pending 只能改一次
                if (me.status !== PENDING) {
                    return
                }
                // 将状态改为 rejected
                me.status = REJECTED
                    // 保存 value
                me.data = reason
                    // 如果存在待执行的 callback 函数，立即异步执行回调函数 onRejected
                if (me.callbacks.length > 0) {
                    // 模拟异步，执行放入队列中所有成功的回调函数
                    setTimeout(() => {
                        me.callbacks.forEach(callbacksObj => {
                            callbacksObj.onRejected(reason)
                        });
                    })
                }
            }

            // 立即同步执行 excutor
            // 如果调用失败，则需捕获异常
            try {
                excutor(resolve, reject)
            } catch (error) { // 如果执行器抛出异常，promise 对象变为 rejected 状态
                reject(error)
            }
        }

        /**
         * Promise 原形对象的 then()
         * 指定成功和失败的回调函数
         * 返回一个新的 Promise 对象
         */
        then(onResolved, onRejected) {
            onResolved = typeof onResolved === 'function' ? onResolved : value => value // 向后传递成功的 value
                // 指定默认的失败的回调(实现错误/异常传透的关键点)
            onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason } // 向后传递失败的 reason

            const me = this

            // 返回一个新的 Promise 对象
            return new Promise((resolve, reject) => {
                /**
                 * 调用指定的回调函数处理，根据执行结果，改变 return 的 promise状态
                 */
                function handle(callback) {
                    /**
                     * 1.如果抛出异常，return 的 promise 就会失败，reason 就是 error
                     * 2.如果回调函数返回不是 promise，return 的 promise 就会成功，value 就是返回值
                     * 3.如果回调函数返回是 promise，return 的 promise 结果就是这个 promise 的结果
                     */
                    try {
                        const result = callback(me.data)
                            // 3.如果回调函数返回是 promise，return 的 promise 结果就是这个 promise 的结果
                        if (result.instanceof(Promise)) {
                            // result.then((resolve, reject) => {
                            //     value => resolve(value),
                            //     reason => reject(reason),
                            // })

                            // 简化
                            result.then(resolve, reject)
                        } else {
                            // 2.如果回调函数返回不是 promise，return 的 promise 就会成功，value 就是返回值
                            resolve(result)
                        }
                    } catch (error) {
                        // 1.如果抛出异常，return 的 promise 就会失败，reason 就是 error
                        reject(error)
                    }
                }

                // 当前还是 pending 状态，将回调函数保存起来
                if (me.status === PENDING) {
                    me.callbacks.push({
                        onResolved(value) {
                            handle(onResolved)
                        },
                        onRejected(reason) {
                            handle(onRejected)
                        }
                    })
                } else if (me.status === RESOLVED) { // 如果当前是 resolved 状态，异步执行 onResolve 并改变 return的 Promise 状态
                    // 回调函数需异步执行
                    setTimeout(() => {
                        handle(onResolved)
                    })
                } else { // 果当前是 rejected 状态，异步执行 onRejected 并改变 return 的 Promise 状态
                    // 回调函数需异步执行
                    setTimeout(() => {
                        handle(onRejected)
                    })
                }
            })
        }

        /**
         * Promise 原形对象的 catch()
         * 指定失败的回调函数
         * 返回一个新的 Promise 对象
         */
        catch (onRejected) {
            return this.then(undefined, onRejected)
        }

        /**
         * Promise 函数对象的 resolve 方法
         * 返回一个指定结果的成功的 Promise
         * 
         * 注: 给类对象添加方法
         * 		在函数前加上 static
         */
        static resolve = function(value) {
            // 返回一个成功/失败的 promise
            return new Promise((resolve, reject) => {
                // value 是 promise时
                if (value instanceof(Promise)) { // 使用 value 的结果作为 promise 的结果
                    value.then(resolve, reject)
                } else { // value 不是 promise 时，promise 成功，数据是 value
                    resolve(value)
                }
            })
        }

        /**
         * Promise 函数对象的 reject 方法
         * 返回一个指定 reason 的失败的 Promise
         */
        static reject = function(reason) {
            return new Promise((resolve, reject) => {
                reject(reason)
            })
        }

        /**
         * Promise 函数对象的 all 方法
         * 返回一个 promise，只有当所有的 promise 都成功时才成功，否则只要有一个失败的就失败
         */
        static all = function(promises) {
            // 用来保存所有成功 value 的数组
            const values = new Array(promises.length)
                // 用来保存成功的 promise 的数量
            let resolvedCount = 0

            // 返回一个新的 promise
            return new Promise((resolve, reject) => {
                // 遍历 promises 获取每个 promise 的结果
                promises.forEach((p, index) => {
                    Promise.resolve(p).then(
                        value => {
                            resolvedCount++ // 成功的数量加1
                            // p 成功，将成功的 value 保存到 values
                            // value.push(p) // 不行,成功的 promise 的顺序也必须是一一对应的
                            values[index] = value

                            // 如果全部成功了，将 return 的 promise 改为成功
                            if (resolvedCount === promises.length) {
                                resolve(values)
                            }
                        },
                        reason => { // 只要一个失败了，return 的 promise 就会失败
                            reject(reason)
                        }
                    )
                })
            })
        }

        /**
         * Promise 函数对象的 race 方法
         * 返回一个 promise，其结果由第一个完成的 promise 决定
         */
        static race = function(promises) {
            // 返回一个 promise
            return new Promise((resolve, reject) => {
                // 遍历 promises 获取每个 promise 的结果
                promises.forEach((p, index) => {
                    Promise.resolve(p).then(
                        value => { // 一旦有成功的，将 return 变为成功
                            resolve(value)
                        },
                        reason => { // 只要一个失败了，return 的 promise 就会失败
                            reject(reason)
                        }
                    )
                })
            })
        }

        /**
         * 工具方法 --- resolveDelay(自定义方法)
         * 每返回一个 promise 对象，它在指定的时间后才确定结果
         */
        static resolveDelay = function(value, time) {
            // 返回一个成功/失败的 promise
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    // value 是 promise
                    if (value instanceof(Promise)) { // 使用 value 的结果作为 promise 的结果
                        value.then(resolve, reject)
                    } else { // value 不是 promise => promise 变为成功，数据是 value
                        resolve(value)
                    }
                }, time);
            })
        }

        /**
         * 工具方法 --- rejectDelay(自定义方法)
         * 每返回一个 promise 对象，它在指定的时间后才失败
         */
        static rejectDelay = function(reason, time) {
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    reject(reason)
                }, time);
            })
        }
    }

    // 向外暴露 Promise 函数
    window.Promise = Promise
})(window)
```

