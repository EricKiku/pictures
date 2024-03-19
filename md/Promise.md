
## Promise是什么

1. 抽象表达式
   1. Promise是一门新的技术
   2. Promise是JS中进行异步编程的新的解决方案，旧的是用回调函数
2. 具体表达
   1. 从语法说：Promise是一个构造函数
   2. 从功能说：Promise用来封装一个异步操作，并可以获取成功/失败的值

3. promise的状态

   1. `pending `变成 `resolved`
   2. `pending` 变成 `rejected`

4. Promise对象的值

   1. `PromiseResult`：保存对象成功或失败的结果 

5. Promise.then和Promise.catch

   1. `then`可以处理成功和失败的结果

   2. `catch`只能处理失败的结果

      1. ```js
         let p = new Promise();
         
         p.catch(reason=>{})
         ```

      2. 

## 为什么使用

1. 指定回调函数的方式更灵活
   1. 旧的必须在启动异步任务前指定
   2. promise：启动异步任务=>返回promise对象=>给对象绑定回调函数
2. 支持链式调用，可以解决回调地狱问题
   1. 回调地狱：回调函数嵌套调用，外部函数的结果是内部函数的执行条件
   2. 回调地狱缺点：不便于阅读，不便于异常处理
   3. 解决方案：Promise链式调用



## Promise基本使用

```html
<script>
        function rand(m,n) {
            return parseInt(Math.random()*(n-m+1));
        }
        let btn = document.getElementById('btn');

        btn.addEventListener('click',function () {
            let promise = new Promise((resolve,reject)=>{
                
                setTimeout(() => {
                    let random = rand(0,10);
                    if (random>=7) {
                         resolve(`中奖了,号码是：${random}`)
                    }
                    reject(`没中奖,号码是：${random}`)
                }, 2000);
            });
            
            promise.then((value)=>{
                alert(value);
            },(reason)=>{
                alert(reason);
            })
        })
</script>
```



## Promise-fs模块练习

```js
const fs = require('fs')
const path = require('path')

let promise = new Promise((resolve,reject)=>{
    fs.readFile(path.resolve(__dirname,"./resource/context.txt"),
    'utf-8',
    (err,data)=>{
        if (err) {
            return reject(`读取错误${err}`);
        }
        resolve(data)
    })
})

promise.then((value)=>{
    console.log(value);
},(reason)=>{
    console.log(reason);
})
```



## Promise-ajax练习

`index.html`

```html
<script>
        let btn = document.getElementById('btn');
        let context = document.getElementById('context');
        btn.addEventListener('click',function () {
            let promise = new Promise((resolve,reject)=>{
                const xhr = new XMLHttpRequest();
                xhr.open('GET',"http://localhost:8000/server-get");
                xhr.send();
                xhr.onreadystatechange = function () {
                    if (xhr.readyState ===4) {
                        if (xhr.status >=200 && xhr.status <300) {
                            resolve(xhr.response)
                        }
                        reject('服务器请求失败')
                    }
                }
            });

            promise.then(value=>{
                context.innerText = value;
            },reason=>{
                context.innerText = reason;
            })
        })
    </script>
```

`server.js`

```js
const express = require('express');
const cors = require('cors')
const app = express();
app.use(cors())
//设置跨域访问
// app.all('*', function(req, res, next) {
//     res.header("Access-Control-Allow-Origin", "*");
//     res.header("Access-Control-Allow-Headers", "X-Requested-With");
//     res.header("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,OPTIONS");
//     res.header("X-Powered-By",' 3.2.1')
//     res.header("Content-Type", "application/json;charset=utf-8");
//     next();
// });

app.get('/server-get',(req,res)=>{
    
    res.setHeader("Access-Control-Allow-Origin","*");
    res.setHeader("Access-Control-Allow-Headers","*");
    res.send('success')
})

app.listen(8000,()=>{
    console.log('服务器已启动，端口号 8000...');
})
```



## 使用Promise封装函数

```js
const fs = require('fs')
const path = require('path')

function mineReadFile(pathFile) {
    let promise = new Promise((resolve,reject)=>{
        fs.readFile(path.resolve(__dirname,pathFile),'utf-8',
        (err,data)=>{
            if (err) {
                reject(`错误${err}`)
            }else{
                resolve(data);
            }
        })
    });

    return promise
}

mineReadFile('./resource/context.txt')
.then(
    (value)=>{
        console.log(value);
    }
    ,(reason)=>{
        console.log(reason);
    }
)
```



## util.promisify

```js
//引入util模块
const util = require('util');
const fs = require('fs');

let fun = util.promisify(fs.readFile);

fun('./resource/context.txt').then(value=>{
    log(value)
})
```





## Promise方法

#### 1. Promise.resolve()

只有传入失败的Promise时，结果才是失败的Promise，其他值都是成功的Promise

#### 2. Promise.reject()

无论传入什么值都返回失败的Promise，结果就是传入的值

#### 3. Promise.all()

```js
let p1 = new Promise(...)
let p2 = new Promise(...)
let p3 = new Promise(...)
Promise.all([p1,p2,p3])=>{}
```

参数是包含了多个promise的数组，只有全部的promise全部是成功的，才返回成功，否则就是失败的Promise。

成功的结果是所有Promise成功的结果的数组

失败的结果就是失败的那个Promise失败的结果

#### 4. Promise.race()

```js
let p1 = new Promise(...)
let p2 = new Promise(...)
let p3 = new Promise(...)
Promise.race([p1,p2,p3])=>{}
```

 参数也是多个Promise，返回的结果是`第一个成功的Promise的结果或失败的结果`

也就是赛跑，第一个完成的Promise的结果就是方法的结果





## p.then()

多次指定Promise的`then`方法，都会执行，

```js
let p = new Promise(..)

p.then(value=>{})
p.then(value=>{})
p.then(value=>{})
```

都会执行





## Promise异常穿透

在一个使用`then`链式调用中，可以在最后指定失败的回调

前面任意一个链式失败了或者异常了，都会传到最后失败的回调中

```js
let p = new Promise({...})
p.then().then().then().catch(...)
```





## 中断Promise链

在链式调用中，如果想在某个链停止向下调用，成为中断链调用

有且只有一种方法：返回一个处在`pending`中的Promise对象

```js
let p = new Promise({...})
p.then(value=>{
   ...
   return new Promise(()=>{})	//返回一个pending状态的Promise对象
}).then().then().catch(...)
```





## Promise手写



```js
class Promise{
    //构造方法
    constructor(executor){
        //添加属性：Promise对象的状态
        this.PromiseState = "pending";
        this.PromiseResult = null
        //保存回调函数
        this.callbacks = [];
        //保存一下实例对象的this值
        const _this = this;
        //resolve函数
        function resolve(data) {
            //判断状态是否发生更改
            if(_this.PromiseState !== 'pending') return;
            //设置对象的状态
            _this.PromiseState = "fulfilled";
            //设置对象的结果值
            _this.PromiseResult = data
            //异步调用的回调函数
            setTimeout(() => {
                //setTimeou的作用是把then方法变成异步异步的
                _this.callbacks.forEach(item=>{
                    item.onResolve()
                })
            });
        
        }
        //reject函数
        function reject(data) {
            //判断状态是否发生更改
            if(_this.PromiseState !== 'pending') return;
            _this.PromiseState = "reject";
            _this.PromiseResult = data
            //异步调用的回调函数
            setTimeout(() => {
                _this.callbacks.forEach(item=>{
                    item.onReject()
                })
            });
            
        }

        //同步调用 执行器函数
        try{
            executor(resolve,reject);
        }catch(e){
            reject(e)
        }
    }

    //then 方法封装
    then(onResolve,onReject){
        const _this = this;
        //判断回调函数，异常穿透
        if (typeof onReject !== 'function') {
            onReject = reason=>{
                throw reason;
            }
        }
        //两个参数都不写，创建一个默认的
        if (typeof onResolve !== 'function') {
            onResolve = value => value
        }
        return new Promise((resolve,reject)=>{
            //封装函数
            function callback(funType) {
                try{
                    //判断返回的是否是Promise对象
                    let result = funType(_this.PromiseResult);
                    if(result instanceof Promise){
                        //如果是，则返回的Promise是成功的，则返回成功的Promise
                        //返回的Promise是齿白的，则返回失败的Promise
                        result.then(v=>{
                            resolve(v);
                        },r=>{
                            reject(r)
                        })
                    }else{
                        //如果不是Promise对象，则状态为成功
                        resolve(result)
                    }
                }catch(e){
                    reject(e)
                }
            }
            if (this.PromiseState === 'fulfilled') {
                setTimeout(() => {
                    //settimeout作用是把then方法变成异步的
                    callback(onResolve);
                });
                
            }
            if(this.PromiseState === 'reject'){
                setTimeout(() => {
                    callback(onReject)
                });
               
            }
            if (this.PromiseState === 'pending') {
                this.callbacks.push({
                    onResolve:function () {
                        callback(onResolve)
                    },
                    onReject:function () {
                        callback(onReject)
                    }
                })
            }
        })
    }

    //catch方法封装
    catch(onRejected){
        return this.then(undefined,onRejected)
    }


    //resolve方法封装，注意，方法是类的，不是实例的，需要static
    static resolve(value){
        //返回Promise对象
        return new Promise((resolve,reject)=>{
            if (value instanceof Promise) {
                value.then(v=>{
                    resolve(v)
                },r=>{
                    reject(r)
                })
            }else{
                resolve(value)
            }
        })
    }

    //reject方法封装
    static reject(reason){
        return new Promise((resolve,reject)=>{
            reject(reason);
        })
    }

    //all方法封装
    static all(promises){
        return new Promise((resolve,reject)=>{
            //声明变量
            let count = 0;
            let arr = [];
    
            for (let i = 0; i < promises.length; i++) {
                promises[i].then(v=>{
                    //必须是所有的promise都成功结果才成功
                    count++;
                    //为结果添加数据
                    arr[i] = v;
                    if (count === promises.length) {
                        resolve(arr);
                    }
                },r=>{
                    reject(r)
                })
            }
        })
    }

    //race方法封装
    static race(promises){
        return new Promise((resolve,reject)=>{
            for (let i = 0; i < promises.length; i++) {
                promises[i].then(v=>{
                    resolve(v);
                },r=>{
                    reject(r)
                })
            }
        })
    }
}

```



## async与await



### 1.async函数

1. 函数的返回值为promise对象
2. promise对象的结果由async函数执行的返回值决定

```js
async function main() {
    //1. 如果返回值不是promise类型的数据，则返回成功
    return 521
    //2.如果返回值是一个promise对象，则对象成功async函数成功
    //对象失败，async函数失败
    return new Promise((resolve,reject)=>{
        // resolve('OK')
        reject('NO OK')
    })
    //3.抛出异常,则async函数失败
    throw 'Oh NO'
}
let result = main();
console.log(result);
```



### 2. await 表达式

1. await右侧的表达式一般为promise对象，但是也可以是其他的值
2. 如果表达式是promise对象，await返回的是promise成功的值
3. 如果表达式是其他值，直接将此值作为await的值

### 3.async和await

1. `await`必须写在`async`函数中，但是`async`中可以没有`await`
2. 如果aiait的promise失败了，就会抛出异常，需要通过try-catch捕获

```js
async function main() {
    let p = new Promise((resolve,reject)=>{
        // resolve('OK')
        resolve('OK')
    })
    let result = await p;
    console.log(result);
}
main()
```





### 4.async+await+fs读取文件

`util.promisify`是在`node.js 8.x`版本中新增的一个工具，用于将老式的`Error first callback`转换为`Promise`对象，让老项目改造变得更为轻松。

以下分别是，用了util的和没有用的注释的

```js
const fs = require('fs')
const util = require('util')
const mineReadFile = util.promisify(fs.readFile)
async function main() {
    try {
        // let data1 = await new Promise((resolve,reject)=>{
        //     fs.readFile('./resource/context.txt'
        //     ,'utf-8'
        //     ,(err,data)=>{
        //         if (err) {
        //             reject(err)
        //         }
        //         resolve(data)
        //     })
        // })
        let data1 = await mineReadFile('./resource/context.txt')
        console.log(data1.toString());
    } catch (e) {
        console.log(e);
    }
}
main()
```

