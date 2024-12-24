---
title: 一些Js的手写
date: 2022-05-25 23:41:58
tags: 前端
---

**常见的JS手写**

<!--more-->

## 一、场景实用

### 手写节流函数

```javascript
// 节流阀
function throttle(func,wait){
    let timer = null
    return function(...args){
        let context = this
        if(!timer){
            // func.call(context,...args) // 如果需要立即调用的话就写在这,一般节流是立即调用的
         	timer = setTimeout(function(){
            	func.call(context,...args)
                clearTimeout(timer)
            	timer=null
        	},wait)   
        }
    }
}
```

### 手写防抖函数

```javascript
// 防抖函数
function debounce(func,wait){
    let timer = null
    return function(...args){
        let context = this
        if(timer)	clearTimeout(timer)
        timer = setTimeout(function(){
            func.call(context,...args)
            clearTimeout(timer)
            timer = null
        },wait)
    }
}
```

### 手写Js深拷贝

```javascript
const is = {// js中其实万物皆对象,所以其实所有Js内置对象都能通过这种方式拿到类型,包括null与undeinfed
    // 基础类型
    Number:(val)=>Object.prototype.toString.call(val) === '[object Number]',
    String:(val)=>Object.prototype.toString.call(val) === '[object String]',
    Boolean:(val)=>Object.prototype.toString.call(val) === '[object Boolean]',
    Undefined:(val)=>Object.prototype.toString.call(val) === '[object Undefined]',
    Null:(val)=>Object.prototype.toString.call(val) === '[object Null]',
    Object:(val)=>Object.prototype.toString.call(val) === '[object Object]',
    Symbol:(val)=>Object.prototype.toString.call(val) === '[object Symbol]',
    BigInt:(val)=>Object.prototype.toString.call(val) === '[object BigInt]',
    
    // 拓展类型
    Array:(val)=>Object.prototype.toString.call(val) === '[object Array]',
    Function:(val)=>Object.prototype.toString.call(val) === '[object Function]',
    Set:(val)=>Object.prototype.toString.call(val) === '[object Set]',
    Map:(val)=>Object.prototype.toString.call(val) === '[object Map]',
    Date:(val)=>Object.prototype.toString.call(val) === '[object Date]',
    RegExp:(val)=>Object.prototype.toString.call(val) === '[object RegExp]',
    Promise:(val)=>Object.prototype.toString.call(val) === '[object Promise]',
    Error:(val)=>Object.prototype.toString.call(val) === '[object Error]',
    ArrayBuffer:(val)=>Object.prototype.toString.call(val) === '[object ArrayBuffer]',
}
// 需要注意的是,对于非内置对象的实例,运算的结果是原型链上遇到的第一个内置对象
// 深递归,防循环引用
function deepClone(target,weakMap = new WeakMap()){
    if(weakMap.get(target))	return weakMap.get(target);// 如果已经递归过这个target,则直接返回
    if(is.Symbol(target))	return Symbol(target.description);
    if(is.Date(target))	return new Date(target);
    if(is.Set(target)){
        const set = new Set();
        weakMap.set(target,set);
        for(let x of target)	set.add(deepClone(x,weakMap));
        return set;
    }
    if(is.Map(target)){
        const map = new Map();
        weakMap.set(target,map);
        for(let [key,value] of target)
            map.set(deepClone(key,weakMap),deepClone(value,weakMap));
        return map;
    }
    if(is.Function(target)){
    	if (/^function|^\(\)/.test(target.toString())) {
    		return new Function(`return ${target.toString()}`)()
    	} else {
    		return new Function(`return function ${target.toString()}`)()
    	}
    }
    
    // 这一条语句上方是会特殊处理的类型,这一条语句下方是对Object与Array的处理,其余情况都返回自身
	if(!is.Object(target) && !is.Array(target))	return target;
  
 	// 对象与数组的循环引用处理
 	const result = new target.constructor();// 利用其构造函数创建新内容(适用于数组与对象)
 	weakMap.set(target,result);

	// 普通对象与数组的处理
 	const keys = Reflect.ownKeys(target);// Reflect.ownKeys和Object.keys的区别是,前者多返回了不可枚举的私有属性,而for in则会把原型链的也返回
 	for(let key of keys){
        result[key] = deepClone(target[key],weakMap);
	}
	return result;
}
```

## 二、重写API

### 手写New实例对象

```javascript
function newObj(Fn,...args){
    let obj = Object.create(Fn.prototype)
    // 调用构造函数,注意这个调用会有返回值,构造函数也有可能内部有return语句返回乱七八糟的值
    const result = Fn.call(obj,...args)
    // 返回了对象值的话就以这个对象值为准,否则
    return result instanceof obj?result:obj
}
```

### 手写组合寄生继承

```javascript
function Parent(name,age){
    this.name = name
    this.age = age
    this.say = function(){
        console.log("haha")
    }
}
function Child(name,age,sex){
    Parent.call(this,name,age)
    this.sex = sex
}
Child.prototype = Object.create(Parent.prototype)
Child.prototype.constructor = Child
```

### 用reduce实现map

```javascript
Array.prototype._map = function(func,thisArg){//map接受参数(value,key,arr)
    const result = []
    this.reduce((a,b,index,arr)=> {
        result[index] = func.call(thisArg,arr[index],index,arr)
    },0)
    return result
}
```

### 手写Promise

```javascript
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';
class Promise {
  // 传入构造器,初始化状态与resolve,reject函数
  constructor(excutor){
    this.state = PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.callbacks = [];
    const resolve = (value) => {
      if(this.state === PENDING){
        this.state = FULFILLED;
        this.value = value;
        this.callbacks.forEach(func => func.onResolved());//状态变化,执行所有成功函数(同个Promise可以被多个then添加成功函数)
      }
    }
    const reject = (reason) => {
      if(this.state === PENDING){
        this.state = REJECTED;
        this.reason = reason;
        this.callbacks.forEach(func => func.onRejected());//同上
      }
    }
    try {
      excutor(resolve,reject);//立即执行构造器的内容
    } catch(e){
      reject(e);// 构造器内扔出错误将调用reject()
    }
  }

  // 传入成功函数与失败函数,状态变化时相应执行
  then(onResolved,onRejected){
    if(typeof onResolved !== 'function')  onResolved = (value)=>value; // 不声明成功函数,value值一直下传
    if(typeof onRejected !== 'function')  onRejected = (err)=>{throw(err)};// 异常传透
    // 公用的处理逻辑,根据type执行不同的函数
    const dealFunction = (type,resolve,reject)=> {
      setTimeout(()=> {
        try {
          let x;
          if(type == 'onResolved')  x = onResolved(this.value);
          if(type == 'onRejected')  x = onRejected(this.reason);
          this.resolvePromise(promise2,x,resolve,reject);
        } catch(e) {
          reject(e);
        }
      })
    }
    const promise2 = new Promise((resolve,reject)=> {
      if(this.state === FULFILLED){
        dealFunction('onResolved',resolve,reject);
      }
      if(this.state === REJECTED){
        dealFunction('onRejected',resolve,reject);
      }
      if(this.state === PENDING){//状态还没变化,将成功与失败函数放入异步队列this.callbacks中
        this.callbacks.push({
          onResolved:()=> {
            dealFunction('onResolved',resolve,reject);
          },
          onRejected:()=> {
            dealFunction('onRejected',resolve,reject);
          }
        })
      }
    })
    return promise2;
  }

  // 基于成功/失败函数执行结果生成新的Promise的结果
  resolvePromise(promise2,x,resolve,reject){
    if(promise2 === x){
      reject(new TypeError("循环等待了!"))
    }
    let called; // 用于只让一个函数执行(要么成功要么失败)
    if(x && (typeof x === 'function' || typeof x === 'object')){
      try {
        let then = x.then;
        if(typeof then === 'function'){//这里把有then的函数或对象直接当成Promise了,实例上部分其他库也可以能有存在then属性的对象或函数
          then.call(x,y=> {
            if(called)  return;
            called = true;
            this.resolvePromise(promise2,y,resolve,reject);//成功了继续递归调用
          },z=> {
            if(called)  return;
            called = true;
            reject(z);// 失败了就失败了
          })
        }
        else {
          resolve(x);
        }
      } catch(e) {
        if(called)  return;
        called = true;
        reject(e);
      }
    }
    else{
      resolve(x);
    }
  }
  
  static resolve(value){
    const promise2 = new Promise((resolve,reject)=> {
      setTimeout(()=> {
        Promise.prototype.resolvePromise(promise2,value,resolve,reject);//跟then里的处理一样
      })
    })
    return promise2;
  }

  static reject(reason){
    return new Promise((resolve,reject)=>{
      reject(reason);
    });
  }

  static all(promises){
    return new Promise((resolve,reject)=> {
      let result = [];// 存放promise结果
      let cnt = 0;// 统计已经完成的promise的数量
      let currentIndex = 0;// 因为返回的顺序是依照原始顺序,而不是完成的顺序,所以这里要准确落位
      for(let promise of promises){
        let index = currentIndex;
        currentIndex++;
        promise.then(value=> {
          result[index]=value;
          cnt++;
          if(cnt == promises.length)  resolve(result);//全完成了,成功
        },err=> {
          reject(err);//有一次失败就88
        })
      }
      if(currentIndex === 0)  resolve(result);//promises的长度为0的也会返回成功,对应空数组
    })
  }

  static race(promises){
    return new Promise((resolve,reject)=> {
      for(let promise of promises){
        promise.then(resolve,reject);//以最快反应的promise为结果
      }
    })
  }
}

Promise.defer = Promise.deferred = function () {
  let dfd = {};
  dfd.promise = new Promise((resolve, reject) => {
    dfd.resolve = resolve;
    dfd.reject = reject;
  });
  return dfd;
}

module.exports = Promise;

// 安装测试脚本:npm install -g promises-aplus-tests
// 测试命令:promises-aplus-tests promise.js
```

### 手写ajax

```javascript
// 封装ajax函数,jQuery版
function AJAX(obj){
    if(!obj.type)   obj.type='GET';
    if(!obj.url)    return console.log("未传入url参数")
    if(!obj.data)   obj.data={}
    const p = new Promise((resolve,reject)=> {
        const xhr = new XMLHttpRequest()    //1.新建XMLHttpRequest对象
        xhr.open('GET','https://api.apiopen.top/api/getImages?page=0&size=10')  //2.打开接口
        xhr.send()  //3.发送
        xhr.onreadystatechange = function() {// 4.触发回调函数
            if(xhr.readyState == 4){//readyState到第四步才确保把该拿的东西都拿全了
                if(xhr.status>=200 && xhr.status<=299){
                    resolve(xhr.response)
                }
                else    reject(xhr.status)
            }
        }
    })
    p.then((value)=> {
        obj.success.call(obj,value)
    },(err)=> {
        obj.success.call(obj,err)
    })
}
export {
    AJAX
}
```

### 手写axios

```javascript
// 封装axios函数
function axios({method,url,params,data}) {
    method = method.toUpperCase()
    return new Promise((resolve,reject) => {
        // 1.创建对象
        const xhr = new XMLHttpRequest()

        // 2.打开接口传入url,有params要解析,解析params参数放到url后面,POST也可以带URL参数的
        let urlStr = "?"
        for(let key in params){
            urlStr+=`${key}=${params[key]}&`
        }
        urlStr = urlStr.slice(0,-1) //如果加了参数会去除末尾的&,不加的话会去除?
        xhr.open(method,url+urlStr)

        // 3.发送数据,有data需要解析
        if(method == 'POST' || method == 'PUT' || method == 'DELETE'){//发请求体
            xhr.setRequestHeader('Content-type','application/json')//告诉服务器发来的是json数据
            xhr.send(JSON.stringify(data))
        }
        else    xhr.send()

        // 4.处理返回
        xhr.responseType = 'json'   //预设返回结果是json格式
        xhr.onreadystatechange = function() {
            if(xhr.readyState == 4){
                if(xhr.status>=200 && xhr.status<=299){
                    resolve({
                        status:xhr.status,
                        message:xhr.statusText,
                        body:xhr.response
                    })
                }
                else    reject(new Error('请求发送失败,状态码为'+xhr.status))
            }
        }
    })
}

axios.get = function(url,options){
    return axios(Object.assign(options,{url,method:'GET'}))
}

axios.post = function(url,options){
    return axios(Object.assign(options,{url,method:'POST'}))
}

axios.put = function(url,options){
    return axios(Object.assign(options,{url,method:'PUT'}))
}

axios.delete = function(url,options){
    return axios(Object.assign(options,{url,method:'DELETE'}))
}
```

### 手写call，apply，bind函数

```javascript
// 封装call函数
function call(Fn,obj,...args) {//rest参数(...args),是伪数组
    // 如果传入的对象为空或undefined则默认指向全局对象
    if(obj === undefined || obj === null){
        obj = globalThis;//ES11新特性,指向全局对象
    }
    obj.temp = Fn   //创建临时函数属性
    let result = obj.temp(...args) //执行函数,要解构args传入
    delete obj.temp //删除临时函数属性
    return result
}
// 封装apply函数,与call基本相同
function apply(Fn,obj,args){
    if(obj === null || obj === undefined || typeof obj != 'o'){
        obj=globalThis
    }
    obj.temp = Fn
    let result = obj.temp(...args)
    delete obj.temp
    return result
}
// 封装bind函数
function bind(Fn,obj,...args1){
    return function(...args2){
        return call(Fn,obj,...args1,...args2)
    }
}
```

### 手写数组分块chunk

```javascript
// 数组分块,chunk(arr,size)将arr分为长size的若干块
function chunk(arr,size=1){
    if(arr.length === 0)    return []
    let result = []
    let tmp = []
    arr.forEach(item => {
        if(tmp.length === 0)    result.push(tmp)//空的时候直接把地址压result
        tmp.push(item)
        if(tmp.length === size)  tmp=[]//到达指定长度就创建新的数组
    });
    return result
}
```

### 手写合并对象

```javascript
// 合并对象的函数,如果a和b同时有该属性则合并为数组,而Object.assign是覆盖
function mergeObject(...args){
    let result = {}
    args.forEach(obj => {
        for(let key in obj){
            if(result.hasOwnProperty(key)){
                result[key] = [].concat(result[key],obj[key])
            }
            else    result[key]=obj[key]
        }
    });
    return result
}
```

### 手写instanceof

```javascript
// 与instanceof类似,判断Fn是否在obj的原型链上
function myInstanceof(obj,Fn) {
    let proto = obj.__proto__   //获取对象的隐式原型
    let prototype = Fn.prototype
    while(proto){
        if(proto === prototype)  return true
        proto = proto.__proto__ //获取隐式原型的隐式原型
    }
    return false
}
```

## 三、排序算法

### 手写冒泡排序

```javascript
/* 冒泡排序,挑最大的放右边,通过交换保持j+1是大的值 */
function maoPaoSort(arr){
    const n = arr.length;
    for(let i=0;i<n;i++){
        for(let j=0;j<n-i-1;j++){
            if(arr[j]>arr[j+1])    [arr[j],arr[j+1]]=[arr[j+1],arr[j]];
        }
    }
}
maoPaoSort(a);
```

### 手写选择排序

```javascript
/* 选择排序,遍历一遍找最小,然后放左边,不稳定原因:[3,4,3,1],一开始3和1就换位了,导致两个3原本的顺序发生变化 */
function selectSort(arr){
    const n = arr.length;
    for(let i=0;i<n;i++){
        let index = i;// 最小元素的index
        for(let j=i;j<n;j++){
            if(arr[j]<arr[index])   index=j;
        }
        [arr[i],arr[index]] = [arr[index],arr[i]];
    }
}
selectSort(a);
```

### 手写插入排序

```javascript
/* 插入排序,维护左边为有序区,通过交换放到合适的位置 */
function insertSort(arr){
    const n = arr.length;
    for(let i=1;i<n;i++){// 默认0已经是有序区了,所以从1开始,将其放入有序区中属于自己的位置
        for(let j=i;j>=1;j--){
            if(arr[j]<arr[j-1]) [arr[j],arr[j-1]] = [arr[j-1],arr[j]];
            else    break;
        }
    }
}
insertSort(a);
```

### 手写快速排序

```javascript
/* 快速排序,选基准,比基准小的放基准左,大的放基准右,递归 */
function quickSort(left,right,arr){
    if(left>right)  return;
    const tmp = arr[left];
    let i=left,j=right;
    while(i<j){
        while(i<j && arr[j]>=tmp)    j--;
        arr[i]=arr[j];
        while(i<j && arr[i]<tmp)    i++;
        arr[j]=arr[i];
    }
    arr[i]=tmp;
    quickSort(left,i-1,arr);
    quickSort(i+1,right,arr);
}
quickSort(0,a.length-1,a);
```

### 手写希尔排序

```javascript
/* 希尔排序,插入排序的一种 */
function xierSort(arr){
    const n = arr.length;
    for(let k=n/2;k>0;k=Math.floor(k/2)){
        for(let i=k;i<n;i++){
            for(let j=i;j>=k;j-=k){
                if(arr[j]<arr[j-k]) [arr[j],arr[j-k]] = [arr[j-k],arr[j]];
                else    break;
            }
        }
    }
}
xierSort(a);
```

### 手写归并排序

```javascript
/* 归并排序,确保左右有序,然后以类似于双指针的方式进行合并 */
function guiBingSort(left,right,arr){
    if(left>=right) return;
    const mid = left+((right-left)>>1);
    // 递归让左右有序
    guiBingSort(left,mid,arr);
    guiBingSort(mid+1,right,arr);
    // 合并两个有序数组,这里是现拷贝了一份原数组,实际上可以在全局拷贝一份原数组然后共用
    const leftArray = [],rightArray = [];
    for(let i=left;i<=mid;i++)  leftArray.push(arr[i]);
    for(let i=mid+1;i<=right;i++)   rightArray.push(arr[i]);
    const n1 = leftArray.length,n2 = rightArray.length;
    let i=0,j=0,k=left;
    while(i<n1 && j<n2) arr[k++] = leftArray[i]<rightArray[j]?leftArray[i++]:rightArray[j++];
    while(i<n1) arr[k++] = leftArray[i++];
    while(j<n2) arr[k++] = rightArray[j++];
}
guiBingSort(0,a.length-1,a);
```

### 手写堆排序

```javascript
/* 堆排序,建最大堆,然后从后开始遍历一遍,每次交换堆顶和堆尾,然后让新堆顶下沉,新堆尾进入有序列 */
function heapSort(arr){
    let n = arr.length;// 堆的长度
    const heapify = (index)=> {// 将index作为堆顶的堆变成大顶堆,其实就是下沉:有两种方式,递归or迭代
        // 1.迭代法
        // let i = index;
        // while(i<n){
        //     let tmp = i,left = i*2+1,right = i*2+2;
        //     if(left<n && arr[left]>arr[tmp])    tmp=left;
        //     if(right<n && arr[right]>arr[tmp])    tmp=right;
        //     if(i!=tmp){// 最大的不是顶部,则交换
        //         [arr[i],arr[tmp]] = [arr[tmp],arr[i]];
        //         i=tmp;
        //     }
        //     else    break;// 最大的是顶部的话就不用继续下沉了
        // }
        // 2.递归法
        let left = 2*index+1,right = 2*index+2,largest = index;
        if(left<n && arr[left]>arr[largest])  largest=left;
        if(right<n && arr[right]>arr[largest])  largest=right;
        if(largest!=index){
            [arr[largest],arr[index]] = [arr[index],arr[largest]];// 交换堆顶
            heapify(largest);// 落位点要继续递归
        }
    }
    // 建堆,从非叶子结点开始,确保每个堆都是大顶堆
    for(let i=Math.floor(n/2);i>0;i--) heapify(i);
    // 排序,交换堆顶和堆尾,堆的长度-1,直至把堆完全毁掉
    for(let i=n-1;i>=0;i--){
        [arr[0],arr[i]] = [arr[i],arr[0]];
        n--;
        heapify(0);
    }
}
```

## 四、其他

### 手写Js函数柯里化

```javascript
function add(...args1){
    const arr = [...args1]
	const inner = function(...args2){
		arr.push(...args2)
		return inner
	}
	// 重写toString方法,让所有参数相加
	inner.toString = () => arr.reduce((a,b)=>a+b)
	return inner
}
```

### 手写发布订阅者模式

```javascript
class pubSub{
    constructor(){
        this.callbacks={}
    }
    subscribe(type,func){
        if(this.callbacks[type])	this.callbacks[type].push(func)
        else	this.callbacks[type]=[func]
    }
    unsubscribe(type,func){
        if(!this.callbacks[type])	return
        if(func === undefined){
            delete this.callbacks[type]
            return
        }
        this.callbacks[type].filter(x=> x!=func)
    }
}
```

### 手写观察者模式

```javascript
class Observer {
    constructor(){
        this.message = []
    }
    subscribe(func){
        this.message.push(func)
    }
    unsubscribe(func){
        this.message.filter(x => x!=func)
    }
    publish(...args){
        this.message.forEach(func=> {
            func(...args)
        })
    }
}
```

### 手写优先队列(最小堆)

```javascript
class MinHeap{
    constructor(){
        this.data = [];
        this.cnt = 0;
    }
    push(x){// 直接放尾部,然后开始上滤
        let index = this.cnt;
        this.data[this.cnt++]=x;
        while(index && this.data[(index-1)>>1]>this.data[index]){//边界：index为根节点 || 父节点比子节点小,>>1是整除,index-1是因为根为0
            this.swap(index,(index-1)>>1);
            index=(index-1)>>1;
        }
    }
    pop(){//出顶部,尾部放顶部,然后开始下滤
        let top = this.data[0];
        this.data[0]=this.data[--this.cnt];// 注意this.cnt是数组长度,访问下标[this.cnt]是越界
        this.data.length = this.cnt;// 抛出尾元素
        let n = this.cnt,index = 0,tmp = index;//tmp维护(当前根左右中)最小的值所在的下标
        while(index*2+1<n){//边界1：index无子节点了(由于堆是完全二叉树,因此无左=无子)
            if(this.data[index*2+1]<this.data[index])	tmp = index*2+1;
            if(index*2+2<n && this.data[index*2+2]<this.data[tmp])	tmp = index*2+2;
            if(tmp===index)	break;//边界2：index的值是最小的
            this.swap(index,tmp);
            index=tmp;
        }
        return top;
    }
    swap(a,b){
        [this.data[a],this.data[b]] = [this.data[b],this.data[a]];
    }
    top(){
        return this.data[0];
    }
    empty(){
        return this.cnt === 0;
    }
}
```

测试题：[设计数字容器系统](https://leetcode.cn/problems/design-a-number-container-system/)

### 手写数组转树

```javascript
const arr = [
  {id: 3, name: '部门3', pid: 1},
  {id: 5, name: '部门5', pid: 4},
  {id: 1, name: '部门1', pid: 0},
  {id: 2, name: '部门2', pid: 1},
  {id: 4, name: '部门4', pid: 3},
]
// 优化成一次遍历的思路,通俗点说其实就是如果还没遍历到俺的父亲,那就先声明一个假父亲,让它知道我是他的儿子
// 等遍历到俺的父亲了,假父亲告知我的儿子都有谁,然后他退场
function arrayToTree(arr){
  const map = new Map()
  for(let x of arr){
    const {id,pid} = x
    // 先看这个id有没有记录过,记录过的话说明已经有chidren了,将暂时对象替换成自己
    if(map.has(id)){
      const obj = map.get(id);
      x.children = obj.children;
      map.set(id,x);
    }

    // 再去找它的父亲
    if(map.has(pid)){// 记录过则push
      const obj = map.get(pid);
      obj.children.push(x);
    }
    else{// 否则新建对象然后push
      const obj = {};
      obj.children = [x];
      map.set(pid,obj);
    }
  }
  return map.get(0).children;
}
console.log(arrayToTree(arr));
```

### 手写数组扁平化

```javascript
const array = [[1, 2, 3], [4, [5, [6, [7, [8, 9]]]]], [10, [11, 12]]];
/* 方法1: 原生方法 */
// console.log(array.flat(Infinity));

/* 方法2:递归 */
// function flatten(arr){
//   let result = [];
//   for(let x of arr){
//     if(Array.isArray(x))  result = result.concat(flatten(x));
//     else  result.push(x);
//   }
//   return result;
// }

/* 方法3:拓展运算符+concat */
// function flatten(arr){
//   let result = arr;
//   while(result.some(item=>Array.isArray(item))){
//     result = [].concat(...result);
//   }
//   return result;
// }

/* 方法4:toString()+split,缺点比较明显,元素的类型均为toString()转换后的,需要特殊处理 */
// function flatten(arr){
//   const str = arr.toString();
//   return str.split(',');
// }

/* 方法5:正则去除所有的左右括号,最后在最外层加上括号,缺点就是JSON.stringify的缺点,非法值都会按null处理,对象中值为undefined的属性会丢失(数组处理为null) */
function flatten(arr) {
  const str = JSON.stringify(arr);
  const result = '[' + str.replace(/\[|\]/g, '') + ']';
  return JSON.parse(result);
}
console.log(flatten(array));
```

