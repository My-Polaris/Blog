---
title: ES规范整合
date: 2022-03-11 13:02:10
tags: 前端
---

**总结了常见的ES新特性**

<!--more-->

### ES6

### 1. let与const

ES5前只有函数作用域，全局作用域与eval作用域，ES6以后有块级作用域。

let与const均不允许重复声明，不挂载window，无变量提升，块级作用域。

const一定要赋初值，简单类型不可更改值，复杂类型不可更改指向。

建议数组和对象都以const声明，其余以let声明。

### 2. 解构赋值

数组的解构赋值与对象的解构赋值，[]与{}。

### 3. 模板字符串

以``包围，允许换行声明（字符串“ ”需要用+或者\连接），也允许嵌入变量。

### 4. 对象的简化写法

```javascript
let name = 'z3'
const obj = {
	name,
	say(){
		console.log("hello");
	}
}
```

### 5. 箭头函数

箭头函数与普通函数区别：1.this指向，普通函数指向调用对象，箭头函数的this取决于声明时所处的作用域链；2.箭头函数无法做构造器；3.箭头函数无法使用arguments属性。

### 6. 函数允许设置默认参数

```javascript
function Fn(a,b=1){};
```

### 7. 扩展运算符...

```javascript
// 1.这里是作为rest参数,与arguments类似，差别在于它是数组，arguments是伪数组
function Fn(a,...rest){};
```

### 8. Symbol类

用于标识唯一属性。

### 9. 迭代器与生成器

```javascript
// 为obj添加迭代器属性,使得能通过for of去遍历它的friends属性
        let obj = {
            name:'z3',
            friends:[
                'xiaoming',
                'xiaohu',
                'letme',
                'gala',
                'bin'
            ],
            // 迭代器是基于Symbol的
            [Symbol.iterator](){
                // 返回指针对象,指向当前数据结构的初始位置,并且这个对象要带next()属性,该next()属性返回一个{vale:'xx',done:false}类的对象
                let index = -1
                return {
                    //使用箭头函数+闭包,达到遍历的效果
                    next:()=> {
                        // value是值,done表示是否遍历到了结尾
                        index++
                        if(index<this.friends.length)   return { value:this.friends[index],done:false }
                        else    return { value:undefined,done:true }
                    }
                }
            }
        }
        for(let man of obj){
            console.log(man)
        }
```

生成器应用场景为一些异步任务的调度，具体效果可以基于yield测试，为生成器传参时会作为yield的返回值。

### 10.Promise

初步解决回调地狱的问题，resolve()，reject()，Promise.then()，Promise.all()，Promise.race()等等。

仿写jQuery的$.ajax请求↓

```javascript
// 封装ajax函数
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
AJAX({
    type:'GET',
    url:'https://api.apiopen.top/api/getImages?page=0&size=10',
    data:{},
    success(res){
        console.log(res)
    }
})
```

### 11. Set与Map

Set：has()，add()，delete()，clear()

Map：has()，set()，get()，clear()

### 12.Class类声明

constructor构造函数，extends声明子类，super()调用父类构造函数，对属性进行get与set的监听，静态成员的声明static。

其中静态成员就类似于为构造函数挂载方法，用它声明子类的时候并不会为子类加上这些方法。

### 13.数值拓展

0b开头表示二进制,0o开头表示八进制,0x开头表示16进制

```javascript
// 解决0.1+0.2 !== 0.3的问题,运用Number.EPSILON表示Number的最小精度
console.log(0.1+0.2 === 0.3)
function equal(a,b){
    return a-b<Number.EPSILON
}
console.log(equal(0.1+0.2,0.3))
```

还有什么isInteger，isNaN等的方法

### 14.对象方法拓展

Object.is()，与===类似，区别在于能判断两个NaN是不同的。

Object.assign(a1,b1)，用于对象的合并，后者会覆盖前者，常用于浅拷贝。

### 15. ES6模块化

浏览器引入的话要为script标签加上type="module"属性，导入导出太常规不必解释。

## ES7新特性

### 1. Array.includes方法

返回是否找到该元素，先前的indexOf是返回下标。

### 2. 指数运算符

指数运算符[**]

## ES8新特性

### 1.async与await

以趋近于同步的方式处理异步问题，前者是让把函数当成Promise对待，识别resolve和reject；后者是把识别接收到的Promise，解析出resolve和reject。

### 2. 对象方法拓展

Object.values()，Object.entries()将对象的各属性变成[key,value]的数组。

### 3. ES8的拓展运算符

之前的...运算符只用于数组拓展，ES8之后也能拓展对象了。

## ES9新特性

### 1. 正则拓展

原子组可命名，支持反向断言，\s修饰符可以让正则表达式的.把换行也匹配了。

## ES10新特性

#### 1. 拓展方法

Object.fromEntires()把二维数组或Map变为对象，与ES8的Object.entries()类似于逆运算。

trimStart()与trimEnd()

arr.flat(k)，将arr扁平化k次，arr.flatMap()在数组的map方法上多了一层flat扁平。

## ES11新特性

### 1.私有属性

class内部以#开头声明，如#age，外部无法访问。

### 2. Promise.allSettled

与Promise.all类似于，均传入Promise数组，all是要全部通过才resolve并返回resolve的Promise们的值，allSettled无论成功失败，直接返回处理结果数组（比如下标[0]resolve，下标[1]reject）。

### 3. 正则的matchAll()

以前要用while(str.exec(reg))获取细节，现在可以matchAll获取迭代器，遍历即可看到每个的细节。

### 4. 可选链操作符

```
let obj = {
	a:{
		b:{
			c:1
		}
	}
}
```

以前需要，if(obj && obj.a && obj.a.b && obj.a.b.c)，确保不会报错

现在可以，obj.?a.?b.?c即可。

### 5. 动态import

在Vue后台管理系统用过，动态import引入，达到懒加载的目的。

### 6. BigInt类型

无限大的整数，let a = 7182491659812658916956128569125681256891n

### 7. globalThis对象

永远指向全局对象，Node指Global，浏览器指window，省去环境的考虑。