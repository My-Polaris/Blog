---
title: JavaScript中的那些细节
date: 2022-07-4 22:01:38
tags: 前端
---

**一些容易忽略的点**

<!--more-->

#### 二、HTML中的JavaScript

- `<script src="xx" />在有些浏览器是不支持的，最好还是用双标签`
- 页面的渲染时刻是在浏览器解析到`<body>`标签开始。
- defer和async都只对外部脚本文件有效，并且都会进行不阻塞的下载（新开线程）；defer脚本文档渲染后才按原来顺序执行，async脚本加载完就执行且谁快谁执行，因此最好别在async里操作DOM，不然边渲染边操作DOM容易出性能问题。
- creatElement创建的script标签默认添加了async属性。
- 除非script标签要带非js代码，否则最好不要指定type（因为可能导致脚本被忽略）。
- 引用外部js文件的好处，可维护性、缓存后可复用。

#### 三、语言基础

- 声明时直接不用关键字可以创建全局变量（即使你在函数内），但严格模式下会报错。

- 暂时性死区：块内声明了i，那在块内声明i的语句前，你既不能拿到外面的i，也不能拿到即将声明的i。

- const和let差别只有：需要给初值、不能修改。

- 严格模式下delete未声明的变量会出错，非严格模式下不会。

- 最佳实践：用于保存对象的变量，未赋值时以null填充，这也是typef null == ‘object’的原因，因为null的含义就是空对象指针。

- NaN转换为Boolean是false。

- 赋值时数字字面量加前缀0表示八进制数字（严格模式下不支持），但若数字存在8以上(包含8)的数字，则依旧视为十进制。

- js除法中，0做分子返回0，0做分母返回正负无穷（Infinity），0同时做分子分母返回NaN。

- isNaN函数接收不能转为数字的变量就返回true。（注意区分isNaN与Number.isNaN，后者没有隐式类型转换）

- js获取对象的值的时候，先调用对象的ValueOf()方法，再调用toString()方法。

- parseInt与Number差别：parseInt面对空字符串直接返回NaN，而Number则返回0；parseInt遇到非数字字符后会直接截断，而Number则返回NaN；parseInt接收第二个参数指定进制，不传或传0的话就判断开头的0x和0（非严格模式下依旧不支持八进制）。（注意parseInt第二个参数传0的话是和不传第二个参数一个效果的）

- parseFloat（对标parseInt）：会识别第一个'.'，只解析十进制值（会忽略开头的0，遇到0x会返回0）。

- Symbol不支持构造函数，即 new Symbol()会报错。

- Symbol.for(str)与普通Symbol的差别：Symbol.for()会返回符号（依据str标识查找注册表，有就返回符号，没有就注册并返回符号），而普通Symbol只是声明了一块内存。

- 即使JS的数字是64位存储，但JS的位运算会将数字转换为32位再进行位运算，再将结果转为64位。（也就说位运算无法得出超过INT_MAX，小于INT_MIN的数，对超出INT范围的数字位运算也得不到理想的结果）

  ![](https://webvoobssdl.kugou.com/ba6560a8e81ac4c91e6ce06cd32a8aca.png)

  上图中，a的初始值超出INT了，位运算时强行截断至32位，因此超出32位的1被无视了。

- 位运算：左移`<<`只有一种，会保留数字符号，有符号右移`>>`填充符号位，无符号右移`>>>`填充0。（数字以补码形式存储）

- +-操作符会隐式调用Number()，!操作费会隐式调用Boolean()。

- 无穷大+无穷大=无穷大，无穷小+无穷小=无穷小，无穷大+无穷小=NaN，操作数有NaN结果就是NaN。

- 加法中字符串是大哥，减法与关系运算中数字是大哥，相等判断和比较运算中数字是大哥(且出现bool值时都会先把它变为数字)。(与大哥运算的时候要被大哥同化)

- undefined和null在和字符串相加时，会变为'undefined'和'null'并进行拼接；它俩在相等运算符时不转换成其他类型。（但是注意undefined==null为true，是因为规范中规定了相等，而不是因为发生了类型转换，当然他们不全等）

- for in会枚举非符号类的属性，for of的使用前提是可迭代对象，一般对象能被for in但不能被for of。

- js的标签语句有点陌生的，简单来说就是能给for语句下一个标签，然后配合break和continue关键字使用，可以做到直接break或continue到外部循环。

  ```javascript
  let num = 0; 
  outermost: 
  for (let i = 0; i < 10; i++) { 
  	for (let j = 0; j < 10; j++) { 
  		if (i == 5 && j == 5) { 
  			break outermost; 
   		} 
  		num++; 
  	} 
  } 
  console.log(num); // 55
  ```

- with语句也挺陌生的，简单来说就是创建一个对象作用域。

  ```javascript
  with(location) { //实际应该是location.hostname,location.search等
  	let qs = search.substring(1); 
  	let hostName = hostname; 
  	let url = href; 
  }
  ```

- switch 语句在比较每个条件的值时会使用全等操作符。

#### 四、变量、作用域与内存

- ECMAScript所有函数都是按值传递的。

```javascript
function setName(obj) { 
 obj.name = "Nicholas"; 
 obj = new Object(); 
 obj.name = "Greg"; 
} 
let person = new Object(); 
setName(person); 
console.log(person.name); // "Nicholas"
// 可以看到person的指向并没有变,因此即使引用类型也是按值传递的,只是传的是"地址"
```

- 引用计数的回收机制会有循环引用的问题，所以现在基本都是依据标记清理，特别的，IE9以前的DOM采用的是引用计数，所以有时会有小问题。

#### 五、基本引用类型

- Date类存储形式为自1970.1.1开始的毫秒数，因此可以直接比较大小，构造函数的初始化会隐式调用Date.parse()和Date.UTC()，常见的格式化方法为`toLocaleDateString()`返回年月日(/分隔)，`toLocaleTimeString()`返回时分秒(:分隔)。

  ```javascript
  new Date("May 23, 2019");//Date.parse("May 23, 2019")
  new Date(2005, 4, 5, 17, 55, 55);//Date.UTC(2005, 4, 5, 17, 55, 55)
  time.toLocaleTimeString()
  '17:23:56'
  time.toLocaleDateString()
  '2022/7/11'
  ```

- 注意点：getTime获取毫秒表示，getDate获取日，getDay()获取周几(0~6)。

- RegExp类的使用场景一般是需要根据字符串生成动态正则，由于传参的字符串，字符串本身也带有`\`转移，所以需要二次转义，比如正则里的`\d`在字符串里要是`\\d`。（参数里1传//里的，2传flags表示匹配模式，如`new RegExp("[bc]at", "i")`）

- exec返回值，带\g的话会基于上次匹配的lastIndex继续匹配。（lastIndex是指向匹配到的字符的最后一位的下一位）

  ```
  {
  	Array:0是匹配的字符串,之后是捕获组匹配的字符串
  	index:匹配到的位置(起始)
  	input:要查找的字符串
  }
  ```

- 为什么基本类型不是对象却可以调用自有方法呢，比如str.slice()，其实是底层会去创建相应的对象实例（Number，Boolean，String），调用特定方法后再销毁；不过即便如此，非特殊情况也不推荐显示用对象声明，免得开发者分不清它们到底是原始值还是引用值。（特殊情况是指需要把它们当对象处理，挂载一些属性啥的）

  ```javascript
  let s1 = "some text"; 
  let s2 = s1.substring(2);
  第二行相当于=====》
  let s1 = new String("some text"); 
  let s2 = s1.substring(2); 
  s1 = "some text";
  ```

- new String("sss")与new Object("sss")是同样效果的，后者instanceof String会为true。

- 在布尔表达式中对象会自动转换为true，即使是new Boolean(false)也不例外。

- 截取字符串的方法：slice和substring都是表示截取a\~b，而substr表示截取a\~a+b，不传第二个参都默认是末尾；负数处理中，一种是处理为字符串长度+该负数，第二种是处理为0，slice都为方式1，substring都为方式2，substr第一个参数是方式1，第二个参数是方式二。

- 查找字符串位置的方法：indexOf从头开始找，lastIndexOf从结尾开始找，找不到都返回-1，可以传第2个参数确认起点下标；ES7还多了个includes方法，差别就是能返回bool值表示找没找到，并且includes能找NaN。

- 冷门方法之——填充方法

  ```javascript
  let stringValue = "foo";
  console.log(stringValue.padStart(6)); // "   foo" 
  console.log(stringValue.padStart(9, ".")); // "......foo" 
  console.log(stringValue.padEnd(6)); // "foo   " 
  console.log(stringValue.padEnd(9, ".")); // "foo......"
  ```

- match和replace都是字符串的方法，exec和test才是RegExp的。

- eval是执行时加载，也就说在里面用声明变量和函数的话，在eval外是不能用的。

#### 六、集合引用类型

- Array构造允许不使用new，效果不变。

- Array.from()：类数组->数组；Array.of()：一组参数->数组。

- 修改数组的length属性会切实影响到数组，设短会删元素，设长填充undefined元素。

- fill()是ES6的数组新方法，参数为：填充的值，开始索引，结束索引的后一位。

- join()不传参数会默认为`,`分隔。

- unshift()方法可以用于在数组头部加元素。

- sort()默认情况下会将每一项都当成字符串进行大小比较，因此如果对数值sort()的话要自己指定排序规则。（从小到大：sort((a,b)=>a-b)，负数a排前，正数b排前）

- splice()常用于删除某些元素的同时插入元素，第一个参数是起始位置，第二个参数是删除的元素个数，后面的N个参数表示要插入的值。

- 断言函数，听起来很高大上，其实就是一个判断是否匹配的函数；在find和findIndex函数时的传参就是断言参数，(element,index,array)=>boolean。（楼下some，every，filter的第一个参数应该都算断言函数）

- 数组的5种迭代方式传参都是((item,index,array),thisArg)，所以重写的时候需要注意，第一个参数是处理函数，第二个参数是上下文环境。

  > some有一项符合就true，every每项符合才true，filter返回函数处理为true的项构成的数组，map返回每个函数处理结果构成的数组，forEach无返回值随便造。
  >
  > 五种方式都不改变原数组。

- 咱也不知道为啥reduce和reduceRight要被叫成归并函数，`reduce((a,b,index,arr),initValue)=>any;`，reduceRight就是逆序求。

- **定型数组我理解为Js为了和某些类型严格的库（多为图形库）进行数据交互时提供的一种便捷性的类型，相关知识点我没细究**。

- Map与Object的差别，Object只能用数值（数值其实也会转为字符串）、字符串与符号作为键值，Map可以用任意类型；Map会维护插入顺序，可以遍历其.entires()或[Symbol.iterator]，直接用forEach也行；（Set同理，Set其实就是key和value都是一个值的Map）

#### 七、迭代器与生成器

- 可迭代对象的本质，暴露一个[Symbol.iterator]属性，值为相应的迭代器工厂函数，函数返回一个新迭代器。

  ```javascript
  [Symbol.iterator]() { 
  	let count = 1, 
  	limit = this.limit; 
  	return { 
  		next() { 
  			if (count <= limit) { 
   				return { done: false, value: count++ }; 
  			} else { 
  				return { done: true, value: undefined }; 
   			} 
   		} 
   	}; 
   }
  ```

- 一个有意思的点：维护同一个迭代器

  ```javascript
  let a = [1, 2, 3, 4, 5]; 
  let iter = a[Symbol.iterator](); 
  for (let i of iter) { 
   console.log(i); 
   if (i > 2)	break;
  } 
  // 1 
  // 2 
  // 3 
  for (let i of iter) { 
   console.log(i); 
  } 
  // 4 
  // 5
  ```

- 生成器函数`function* generatorFunction(){}`，箭头函数不能用来定义生成器函数，生成器函数的特性在于能用yield，即支持在函数块内暂停和恢复代码执行。

#### 八、对象与类

- 对象的属性是拥有一些[特性]的，它们标志了这个属性的特点；以特性为基础，可以分为两种类型的属性，分别为数据属性和访问器属性。

- 数据属性

  - 以普通方式声明时（比如`let a = {name:'z3'}`，这个name属性就是普通方式声明的），前三个参数都默认为true；
  - 以Object.defineProperty方式声明时（比如`Object.defineProperty(person, "name", { configurable: false,value: "Nicholas"}); `），声明时前三个参数中未被指定的都默认为false。

  > Configurable：是否可以通过 delete 删除并重新定义，是否可以修改它的特
  >
  > 性，以及是否可以把它改为访问器属性。
  >
  > Enumerable：是否可以被for in获取。
  >
  > Writable：是否可更改
  >
  > Value：值，默认值为undefined。

- 访问器属性

  - 只能以Object.defineProperty方式声明，声明时前两个参数未被指定则默认为false，未指定get则表示只可写，未指定set则表示只可读。

  > Configurable：是否可以通过 delete 删除并重新定义，是否可以修改它的特
  >
  > 性，以及是否可以把它改为访问器属性。
  >
  > Enumerable：是否可以被for in获取。
  >
  > Get()：读取属性的时候会调用
  >
  > Set()：设置属性的时候会调用

- 需要注意的是，一个属性要么为数据属性要么为访问器属性，同名的话会互相覆盖。

- 静态方法Object.getOwnPropertyDescriptors()可以查看对象的所有属性的特性，它实际上是依赖对象身上的getOwnPropertyDescriptor()方法完成的。

- Object.defineProperties可以用于定义一个对象的多个属性及其特性。

  ```javascript
  let book = {}; 
  Object.defineProperties(book, { 
  	year_: { 
  		value: 2017 
   	}, 
   	edition: { 
  		value: 1 
   	}, 
   	year: { 
   		get: function() { 
   			return this.year_; 
   		}, 
   		set: function(newValue){ 
   			if (newValue > 2017) { 
   				this.year_ = newValue; 
   				this.edition += newValue - 2017; 
   			} 
   		} 
   	} 
  });
  ```

- Object.assign的实现机制是，调用源对象的get方法，将返回结果作为目标对象set方法的参数，调用目标对象的set方法。

- ES6常见：属性值简写，可计算属性，简写方法名，解构赋值，类。

- 类构造函数不用new调用会报错，而普通构造函数不会（默认this为global对象）。

- 继承类构造函数中，必须调用super()【其实强行返回一个对象也不会报错】，并且在super前不能调用this。（不写构造函数会默认调用并传入收到的所有参数）

  > tips：可以用super.xxx调用父类的xxx方法。

- 构造函数继承->原型链继承->组合继承->寄生组合继承。构造函数：变量写this里，方法挂prototype原型对象上；类：变量写this里，方法跟constructor同级声明。

- 默认情况下，类定义中的代码都在严格模式下执行。

- ES5实现抽象基类（new.tartget是ES6新属性，它指向调用new 后跟着的构造函数）

  ```javascript
  // 抽象基类
  class Vehicle { 
   constructor() { 
  	if (new.target === Vehicle) {// 构造函数不能是自己,只能是子类
  		throw new Error('Vehicle cannot be directly instantiated'); 
  	} 
   	if (!this.foo) {// 表示一定要声明voo这个方法
   		throw new Error('Inheriting class must define foo()'); 
   	} 
   } 
  }
  ```

#### 九、代理与反射

- Proxy代理其实就是类似于，在原对象上套一层代理对象，可以做一些拦截，代理的意义就是为基本操作嵌入一些额外行为。

- Reflect反射与Proxy代理对应，存放一些原始行为，如get，set之类的，一般都是拦截后做一些自己的处理，然后直接调用全局Reflect对象上的相应方法即可。

  ```javascript
  const target = { 
  	foo: 'bar', 
  	baz: 'qux' 
  }; 
  const handler = { 
  	get(trapTarget, property, receiver) {
          // 目标对象,查询的属性,代理对象
  		let decoration = ''; 
  		if (property === 'foo') { 
  			decoration = '!!!'; 
  		} 
  		return Reflect.get(...arguments) + decoration; 
   	} 
  }; 
  const proxy = new Proxy(target, handler); 
  console.log(target.foo); // bar
  console.log(target.baz); // qux
  console.log(proxy.foo); // bar!!!
  console.log(proxy.baz); // qux 
  ```

- Proxy相较于Object.defineProperty的好处：1.分离了目标对象与代理对象，只有处理代理对象的时候才会触发相应规则；2.对整个对象拦截而不是某个属性。

- 如果有取消代理的需求，可以参考如下。

  ```javascript
  // 直接调用静态方法revocable建立代理关系,返回的proxy是代理对象,revoke是解除方法
  const { proxy, revoke } = Proxy.revocable(target, handler);
  // 需要取消的时候调用一下revoke()即可
  revoke(); 
  ```

- 值得一提的是有个construct()捕获器，会在new操作的时候被调用。

#### 十、函数

- ECMAScript的函数参数只是为了方便才写出来的，本质上都是从arguments对象取出。

- 箭头函数不能使用 arguments、super 和new.target，也不能用作构造函数。此外，箭头函数也没有 prototype 属性。

- 非严格模式下，修改arguments的值会同步到相应的命名参数。（严格模式下不会）

- arguments对象只反应传进来的值，修改命名参数并不会同步到arguments对象，因此默认参数设置的时候也自然不会影响到arguments对象。

- arguments.callee会指向arguments对象所在的函数，可以用于解决函数表达式可能会有的问题。（用函数声明就没这问题）

  ```javascript
  let anotherFactorial = factorial;// 假设factorial是一个匿名递归函数
  factorial = null; 
  console.log(anotherFactorial(4)); // 报错,因为函数内找不到factorial了
  // 递归这样写就没问题,类似于解耦的作用
  factorial = function(num) { 
  	if (num <= 1)	return 1;
      else	return num * arguments.callee(num - 1); 
  }
  ```

- ES6的尾调用优化可以了解一下，写递归的时候可以在一定程度上优化栈空间。（严格模式下）

  其实就是在递归调用的时候，如果这个函数本身没必要继续存储了，那就直接弹出栈空间，比如下述代码中，优化前fib(n-1)的结果要一直存着，所以当前fib不能直接出栈，优化后由于当前fibImpl的返回结果只取决于下个fibImpl的返回结果，当前fibImpl不影响结果，因此当前fibImpl可以直接弹出栈空间，从而节省内存。

  ```javascript
  /* 优化前的斐波那契数列递归函数 */
  function fib(n) { 
  	if (n < 2) return n; 
  	return fib(n - 1) + fib(n - 2); 
  }
  /* 优化后 */
  "use strict"; 
  // 基础框架
  function fib(n) { 
  	return fibImpl(0, 1, n); 
  } 
  // 执行递归
  function fibImpl(a, b, n) { 
  	if (n === 0)	return a;
  	return fibImpl(b, a + b, n - 1); 
  }
  ```

- 经常有人讲闭包引起内存泄漏，其实不全面；准确来说是低版本浏览器+引用计数+闭包引起的，IE9以前的DOM的垃圾回收机制是依靠引用计数的，而引用计数有循环引用的风险，闭包函数与DOM元素形成循环引用就会导致内存泄漏，即内存永远不被释放。

  ```javascript
  function assignHandler() {// 内存泄漏的样例
  	let element = document.getElementById('someElement'); 
  	element.onclick = () => console.log(element.id);
  }
  ```

#### 十一、期约

- 幂等方法：同参重复执行这个函数会得到同一个结果。

  ```javascript
  p1 === Promise.resolve(p1)//false,resolve是幂等方法
  p2 === Promise.reject(p2)//false,reject不是幂等方法
  ```

- try，catch抓不住内部异步行为内的报错，要以异步形式去捕捉异步的报错。

- Promise.prototype.catch()实际只是Promise.prototype.then(undefined,onRejected)的语法糖。

- Promise.finally()无论Promise状态变成功还是拒绝都会执行，一般用作成功失败的公用处理或清理处理。

- Promise.all()均成功会返回对应的Promise数组，Promise.race()返回第一个处理完的Promise，两者失败后都是返回第一个失败的Promise，但都会静默处理所有拒绝操作。

- await会先暂停异步函数后的代码，尝试解包，然后再恢复执行。

- 一道普通的Promise题

  ```javascript
  async function foo() { 
   console.log(2); 
   console.log(await Promise.resolve(8)); 
   console.log(9); 
  } 
  async function bar() {
      console.log(4); 
  	console.log(await 6); 
  	console.log(7); 
  } 
  console.log(1); 
  foo(); 
  console.log(3); 
  bar(); 
  console.log(5);
  // 旧版标准打印顺序是1-9，因为await后跟Promise会有俩异步任务生成
  // 后来TC39改了一次标准，现在只会生成一个异步任务了，也就是说，新版标准的打印顺序为:123458967
  ```

#### 十二、BOM

- window对象有两重身份，一是作为Global对象提供全局方法（如parseInt），二是提供控制浏览器窗口的接口。

- top对象指向顶层窗口，parent对象指向当前窗口的父级窗口，self对象指向window。

- window.devicePixelRatio表示物理像素与逻辑像素的比例，如为3则代表会用3个物理像素去显示一个逻辑像素（CSS像素，即px）。

- outerHeight、outerWidth是整个浏览器的大小，innerHeight、innderWidth是浏览器的视口的大小。

- setTimeout和setInterval都是不保证执行时间的，只能保证在这个时段加入到任务队列内。（JavaScript是单线程的，不过浏览器是多线程的，setTimeout的最小时间间隔是4ms）

- alert，confirm，prompt会阻塞代码执行，直到这些对话框消失。

- window.find()跟用户自己按Ctrl+F搜索是同一个功能。

- 对location.search，即查询字符串处理，可以考虑用`URLSearchParams`的API（不过即使手动处理也不麻烦）。

- Location对象对应浏览器导航栏的相关信息，Navigator对应浏览器本身的一些相关信息（如浏览器版本）。

- History对象出于安全考虑，不会允许你去获取访问过的URL，但可以通过为go()或forword()方法传入字符串，匹配到最近的页面，如`history.go("wrox.com") `，导航到最近的 wrox.com 页面。

- 单页面应用中，为防止多余的页面刷新，最好只改动location.hash（用location改变其他任何都会引起页面刷新）。

- H5新增了一种方法叫histroy.pushState，作用是不刷新页面的去改变url，并支持回退，它与普通路由跳转的区别在于：1.能真的改URL，而不只是hash；2. 更新后的url要确实能找到内容，否则一刷新就白给。

  > 具体的应用可以参考 [ajax与HTML5 history pushState/replaceState实例 « 张鑫旭-鑫空间-鑫生活 (zhangxinxu.com)](https://www.zhangxinxu.com/wordpress/2013/06/html5-history-api-pushstate-replacestate-ajax/)
