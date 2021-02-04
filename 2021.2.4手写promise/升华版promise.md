```js
   const MyPromise = (()=>{
       const REJECTED = "reject",
             RESOLVED = "resolved",
             PENDING = "pending",
             PromiseStatus = Symbol("PromiseStatus"),
             PromiseValue = Symbol("PromiseValue"),
             changeStatus = Symbol("changeStatus"),
             thenAbles = Symbol("thenAbles"),
             catchAbles = Symbol("catchAbles"),
             settleHandle = Symbol("settleHandle"),
             linkPormise = Symbol("linkPormise");
       return class MyPromise {
           [changeStatus](status, value, queue) {
                if(this[PromiseStatus] === PENDING){
                    this[PromiseStatus] = status;
                    this[PromiseValue] = value;
                    queue.forEach(handle => handle(value))
                }
            }
           constructor(executor) {
               this[PromiseStatus] = PENDING;
               this[PromiseValue] = undefined;
               this[thenAbles] = [];
               this[catchAbles] = [];
               const resolve = data =>{
                   this[changeStatus](RESOLVED, data, this[thenAbles])
               }
               const reject = err => {
                    this[changeStatus](REJECTED, err, this[catchAbles])
               }
               try{
                    executor(resolve,reject)
               }catch(e){
                   reject(e)
               }
           }
           [settleHandle](handle, status, queue) {
               if(typeof handle !== "function"){
                   return;
               }
               if(this[PromiseStatus] === status) {
                   setTimeout(()=>{
                       handle()
                   },0)
               }else {
                    queue.push(thenAble)
               }
           }
           [linkPormise]() {
               function exec(handle, resolve,reject, data) {
                   try{
                        const result = thenAble(data);
                        if(result instanceof MyPromise){
                            result.then(data=>{
                                resolve(data)
                            },err=>{
                                reject(err)
                            })
                        }else {
                            resolve(result)
                        }
                    }catch(e){
                        reject(e)
                    }
               }
               return new MyPromise((resolve,reject) => {
                   this[settleHandle](data =>{
                       if(typeof thenAble !=="function"){
                           resolve(data);
                           return
                       }
                       exec(thenAble, resolve, reject, data)
                   }, RESOLVED, this[thenAbles]);
                    this[settleHandle](err =>{
                        if(typeof catchAble !== "function"){
                            reject(err);
                            return
                        }
                        exec(catchAble, resolve, reject, err)
                    }, REJECTED, this[catchAbles])
               })
           }
           then(thenAble,catchAble) {
            //    this[settleHandle](thenAble, RESOLVED, this[thenAbles])
            //    this.catch(catchAble)
               this[linkPormise](thenAble, catchAble)
           }
           catch(catchAble) {
            //    this[settleHandle](catchAble, REJECTED, this[catchAbles])
               this[linkPormise](undefined, catchAble)
           }
       }
   })()
```