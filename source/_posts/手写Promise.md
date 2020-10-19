---
title: 手写Promise-ES5函数版与ES6_class版
categories:
  - javascript
tags:
  - 异步
  - ES6
toc: true
abbrlink: 32a89cf3
---

>自定义Promise的全部逻辑，并加入两个延迟执行的方法

<!-- more -->

## 一、ES5函数版

```js
/**
 * 自定义Promise函数模块: IIFE
 * 
 */
(function(window) {
    // 将经常使用的字符串用常量代替
    const PENDING = 'pending'
    const RESOLVED = 'resolved'
    const REJECTED = 'rejected'

    /**
     * Promise构造函数
     * excutor: 执行器函数(同步执行)
     */
    function Promise(excutor) {
        // 将当前Promise保存起来
        const me = this
        me.status = PENDING // 给promise对象指定status属性，初始值为pending
        me.data = undefined // 给promise对象指定一个用于存储结果数据的属性
        me.callbacks = [] // 每个元素的结构: {onResolved() {}, onRejected() {}}

        function resolve(value) {
            // 如果当前状态不是pending，直接结束 --- 因为pending只能改一次
            if (me.status !== PENDING) {
                return
            }
            // 将状态改为resolved
            me.status = RESOLVED
                // 保存value
            me.data = value
                // 如果存在待执行的callback函数，立即异步执行回调函数onResolved
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
            // 如果当前状态不是pending，直接结束 --- 因为pending只能改一次
            if (me.status !== PENDING) {
                return
            }
            // 将状态改为rejected
            me.status = REJECTED
                // 保存value
            me.data = reason
                // 如果存在待执行的callback函数，立即异步执行回调函数onRejected
            if (me.callbacks.length > 0) {
                // 模拟异步，执行放入队列中所有成功的回调函数
                setTimeout(() => {
                    me.callbacks.forEach(callbacksObj => {
                        callbacksObj.onRejected(reason)
                    });
                })
            }
        }

        // 立即同步执行excutor
        // 如果调用失败，则需捕获异常
        try {
            excutor(resolve, reject)
        } catch (error) { // 如果执行器抛出异常，promise对象变为rejected状态
            reject(error)
        }
    }

    /**
     * Promise原形对象的then()
     * 指定成功和失败的回调函数
     * 返回一个新的Promise对象
     */
    Promise.prototype.then = function(onResolved, onRejected) {
        onResolved = typeof onResolved === 'function' ? onResolved : value => value // 向后传递成功的value
            // 指定默认的失败的回调(实现错误/异常传透的关键点)
        onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason } // 向后传递失败的reason

        const me = this

        // 返回一个新的Promise对象
        return new Promise((resolve, reject) => {
            /**
             * 调用指定的回调函数处理，根据执行结果，改变return的promise状态
             */
            function handle(callback) {
                /**
                 * 1.如果抛出异常，return的promise就会失败，reason就是error
                 * 2.如果回调函数返回不是promise，return的promise就会成功，value就是返回值
                 * 3.如果回调函数返回是promise，return的promise结果就是这个promise的结果
                 */
                try {
                    const result = callback(me.data)
                        // 3.如果回调函数返回是promise，return的promise结果就是这个promise的结果
                    if (result.instanceof(Promise)) {
                        // result.then((resolve, reject) => {
                        //     value => resolve(value),
                        //     reason => reject(reason),
                        // })

                        // 简化
                        result.then(resolve, reject)
                    } else {
                        // 2.如果回调函数返回不是promise，return的promise就会成功，value就是返回值
                        resolve(result)
                    }
                } catch (error) {
                    // 1.如果抛出异常，return的promise就会失败，reason就是error
                    reject(error)
                }
            }

            // 当前还是pending状态，将回调函数保存起来
            if (me.status === PENDING) {
                me.callbacks.push({
                    onResolved(value) {
                        handle(onResolved)
                    },
                    onRejected(reason) {
                        handle(onRejected)
                    }
                })
            } else if (me.status === RESOLVED) { // 如果当前是resolved状态，异步执行onResolve并改变return的Promise状态
                // 回调函数需异步执行
                setTimeout(() => {
                    handle(onResolved)
                })
            } else { // 果当前是rejected状态，异步执行onRejected并改变return的Promise状态
                // 回调函数需异步执行
                setTimeout(() => {
                    handle(onRejected)
                })
            }
        })
    }

    /**
     * Promise原形对象的catch()
     * 指定失败的回调函数
     * 返回一个新的Promise对象
     */
    Promise.prototype.catch = function(onRejected) {
        return this.then(undefined, onRejected)
    }

    /**
     * Promise函数对象的resolve方法
     * 返回一个指定结果的成功的Promise
     */
    Promise.resolve = function(value) {
        // 返回一个成功/失败的promise
        return new Promise((resolve, reject) => {
            // value是promise时
            if (value instanceof(Promise)) { // 使用value的结果作为promise的结果
                value.then(resolve, reject)
            } else { // value不是promise时，promise成功，数据是value
                resolve(value)
            }
        })
    }

    /**
     * Promise函数对象的reject方法
     * 返回一个指定reason的失败的Promise
     */
    Promise.reject = function(reason) {
        return new Promise((resolve, reject) => {
            reject(reason)
        })
    }

    /**
     * Promise函数对象的all方法
     * 返回一个promise，只有当所有的promise都成功时才成功，否则只要有一个失败的就失败
     */
    Promise.all = function(promises) {
        // 用来保存所有成功value的数组
        const values = new Array(promises.length)
            // 用来保存成功的promise的数量
        let resolvedCount = 0

        // 返回一个新的promise
        return new Promise((resolve, reject) => {
            // 遍历promises获取每个promise的结果
            promises.forEach((p, index) => {
                Promise.resolve(p).then(
                    value => {
                        resolvedCount++ // 成功的数量加1
                        // p成功，将成功的value保存到values
                        // value.push(p) // 不行,成功的promise的顺序也必须是一一对应的
                        values[index] = value

                        // 如果全部成功了，将return的promise改为成功
                        if (resolvedCount === promises.length) {
                            resolve(values)
                        }
                    },
                    reason => { // 只要一个失败了，return的promise就会失败
                        reject(reason)
                    }
                )
            })
        })
    }

    /**
     * Promise函数对象的race方法
     * 返回一个promise，其结果由第一个完成的promise决定
     */
    Promise.race = function(promises) {
        // 返回一个promise
        return new Promise((resolve, reject) => {
            // 遍历promises获取每个promise的结果
            promises.forEach((p, index) => {
                Promise.resolve(p).then(
                    value => { // 一旦有成功的，将return变为成功
                        resolve(value)
                    },
                    reason => { // 只要一个失败了，return的promise就会失败
                        reject(reason)
                    }
                )
            })
        })
    }

    /**
     * 工具方法 --- resolveDelay(自定义方法)
     * 每返回一个promise对象，它在指定的时间后才确定结果
     */
    Promise.resolveDelay = function(value, time) {
        // 返回一个成功/失败的promise
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                // value是promise
                if (value instanceof(Promise)) { // 使用value的结果作为promise的结果
                    value.then(resolve, reject)
                } else { // value不是promise => promise变为成功，数据是value
                    resolve(value)
                }
            }, time);
        })
    }

    /**
     * 工具方法 --- rejectDelay(自定义方法)
     * 每返回一个promise对象，它在指定的时间后才失败
     */
    Promise.rejectDelay = function(reason, time) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                reject(reason)
            }, time);
        })
    }

    // 向外暴露Promise函数
    window.Promise = Promise
})(window)
```



## 二、ES6 class版

```javascript
/**
 * 自定义Promise函数模块: class
 * 
 */
(function(window) {
    // 将经常使用的字符串用常量代替
    const PENDING = 'pending'
    const RESOLVED = 'resolved'
    const REJECTED = 'rejected'

    class Promise {
        /**
         * Promise构造函数
         * excutor: 执行器函数(同步执行)
         */
        constructor(excutor) {
            // 将当前Promise保存起来
            const me = this
            me.status = PENDING // 给promise对象指定status属性，初始值为pending
            me.data = undefined // 给promise对象指定一个用于存储结果数据的属性
            me.callbacks = [] // 每个元素的结构: {onResolved() {}, onRejected() {}}

            function resolve(value) {
                // 如果当前状态不是pending，直接结束 --- 因为pending只能改一次
                if (me.status !== PENDING) {
                    return
                }
                // 将状态改为resolved
                me.status = RESOLVED
                    // 保存value
                me.data = value
                    // 如果存在待执行的callback函数，立即异步执行回调函数onResolved
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
                // 如果当前状态不是pending，直接结束 --- 因为pending只能改一次
                if (me.status !== PENDING) {
                    return
                }
                // 将状态改为rejected
                me.status = REJECTED
                    // 保存value
                me.data = reason
                    // 如果存在待执行的callback函数，立即异步执行回调函数onRejected
                if (me.callbacks.length > 0) {
                    // 模拟异步，执行放入队列中所有成功的回调函数
                    setTimeout(() => {
                        me.callbacks.forEach(callbacksObj => {
                            callbacksObj.onRejected(reason)
                        });
                    })
                }
            }

            // 立即同步执行excutor
            // 如果调用失败，则需捕获异常
            try {
                excutor(resolve, reject)
            } catch (error) { // 如果执行器抛出异常，promise对象变为rejected状态
                reject(error)
            }
        }

        /**
         * Promise原形对象的then()
         * 指定成功和失败的回调函数
         * 返回一个新的Promise对象
         */
        then(onResolved, onRejected) {
            onResolved = typeof onResolved === 'function' ? onResolved : value => value // 向后传递成功的value
                // 指定默认的失败的回调(实现错误/异常传透的关键点)
            onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason } // 向后传递失败的reason

            const me = this

            // 返回一个新的Promise对象
            return new Promise((resolve, reject) => {
                /**
                 * 调用指定的回调函数处理，根据执行结果，改变return的promise状态
                 */
                function handle(callback) {
                    /**
                     * 1.如果抛出异常，return的promise就会失败，reason就是error
                     * 2.如果回调函数返回不是promise，return的promise就会成功，value就是返回值
                     * 3.如果回调函数返回是promise，return的promise结果就是这个promise的结果
                     */
                    try {
                        const result = callback(me.data)
                            // 3.如果回调函数返回是promise，return的promise结果就是这个promise的结果
                        if (result.instanceof(Promise)) {
                            // result.then((resolve, reject) => {
                            //     value => resolve(value),
                            //     reason => reject(reason),
                            // })

                            // 简化
                            result.then(resolve, reject)
                        } else {
                            // 2.如果回调函数返回不是promise，return的promise就会成功，value就是返回值
                            resolve(result)
                        }
                    } catch (error) {
                        // 1.如果抛出异常，return的promise就会失败，reason就是error
                        reject(error)
                    }
                }

                // 当前还是pending状态，将回调函数保存起来
                if (me.status === PENDING) {
                    me.callbacks.push({
                        onResolved(value) {
                            handle(onResolved)
                        },
                        onRejected(reason) {
                            handle(onRejected)
                        }
                    })
                } else if (me.status === RESOLVED) { // 如果当前是resolved状态，异步执行onResolve并改变return的Promise状态
                    // 回调函数需异步执行
                    setTimeout(() => {
                        handle(onResolved)
                    })
                } else { // 果当前是rejected状态，异步执行onRejected并改变return的Promise状态
                    // 回调函数需异步执行
                    setTimeout(() => {
                        handle(onRejected)
                    })
                }
            })
        }

        /**
         * Promise原形对象的catch()
         * 指定失败的回调函数
         * 返回一个新的Promise对象
         */
        catch (onRejected) {
            return this.then(undefined, onRejected)
        }

        /**
         * Promise函数对象的resolve方法
         * 返回一个指定结果的成功的Promise
         * 
         * 注: 给类对象添加方法
         * 		在函数前加上 static
         */
        static resolve = function(value) {
            // 返回一个成功/失败的promise
            return new Promise((resolve, reject) => {
                // value是promise时
                if (value instanceof(Promise)) { // 使用value的结果作为promise的结果
                    value.then(resolve, reject)
                } else { // value不是promise时，promise成功，数据是value
                    resolve(value)
                }
            })
        }

        /**
         * Promise函数对象的reject方法
         * 返回一个指定reason的失败的Promise
         */
        static reject = function(reason) {
            return new Promise((resolve, reject) => {
                reject(reason)
            })
        }

        /**
         * Promise函数对象的all方法
         * 返回一个promise，只有当所有的promise都成功时才成功，否则只要有一个失败的就失败
         */
        static all = function(promises) {
            // 用来保存所有成功value的数组
            const values = new Array(promises.length)
                // 用来保存成功的promise的数量
            let resolvedCount = 0

            // 返回一个新的promise
            return new Promise((resolve, reject) => {
                // 遍历promises获取每个promise的结果
                promises.forEach((p, index) => {
                    Promise.resolve(p).then(
                        value => {
                            resolvedCount++ // 成功的数量加1
                            // p成功，将成功的value保存到values
                            // value.push(p) // 不行,成功的promise的顺序也必须是一一对应的
                            values[index] = value

                            // 如果全部成功了，将return的promise改为成功
                            if (resolvedCount === promises.length) {
                                resolve(values)
                            }
                        },
                        reason => { // 只要一个失败了，return的promise就会失败
                            reject(reason)
                        }
                    )
                })
            })
        }

        /**
         * Promise函数对象的race方法
         * 返回一个promise，其结果由第一个完成的promise决定
         */
        static race = function(promises) {
            // 返回一个promise
            return new Promise((resolve, reject) => {
                // 遍历promises获取每个promise的结果
                promises.forEach((p, index) => {
                    Promise.resolve(p).then(
                        value => { // 一旦有成功的，将return变为成功
                            resolve(value)
                        },
                        reason => { // 只要一个失败了，return的promise就会失败
                            reject(reason)
                        }
                    )
                })
            })
        }

        /**
         * 工具方法 --- resolveDelay(自定义方法)
         * 每返回一个promise对象，它在指定的时间后才确定结果
         */
        static resolveDelay = function(value, time) {
            // 返回一个成功/失败的promise
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    // value是promise
                    if (value instanceof(Promise)) { // 使用value的结果作为promise的结果
                        value.then(resolve, reject)
                    } else { // value不是promise => promise变为成功，数据是value
                        resolve(value)
                    }
                }, time);
            })
        }

        /**
         * 工具方法 --- rejectDelay(自定义方法)
         * 每返回一个promise对象，它在指定的时间后才失败
         */
        static rejectDelay = function(reason, time) {
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    reject(reason)
                }, time);
            })
        }
    }

    // 向外暴露Promise函数
    window.Promise = Promise
})(window)
```

