---
title: 2026面经
date: 2026-01-13 18:00:00
tags: 前端
---

26年初开始的一波面试经历、此时两年半的工作经历

<!--more-->

## 2026.1.12 阿里淘天一面（88VIP团队）
1. 手写节流防抖
2. https链接正则识别,比如完整识别https://www.baidu.com/path?x=1
3. 事件循环的一道题
问简历项目-CR规则生成工具

## 2026.1.13 阿里千问一面
全程问简历：稳定性部分的问题没毛病，但是我的性能基础太薄弱了，需要恶补
- 有没有观测过项目的一些指标
- RN和React的差别
- 如果后端的接口速度不满足你的要求，有没有想过能怎样去优化
- 了解过SSR的运作流程吗
- LCP了解过吗
- 有没有观测过你模块的性能，怎么做性能优化呢

## 2026.1.21 小红书一面
问简历：公会架构、聊天室场景的实现
- 故障预防除了ESLint插件还做了哪些事 —— 智能CR、SDK插件等
- 公会现状架构有什么其他痛点 —— 全局弹窗、基座与子应用双包(组件库、Vue等)
- 基座和子应用通信的方式都有哪些 —— single-spa能力、window、customEvent
- 实现扁平化方法，支持层级参数，比如flatten(arr, 2)表示最多只解构两层

## 2026.1.26 Presence(类海外探探)一面
1. 实现Promise.allsettleed
```javascript
Promise.allsettled = (arr) => {
    const result = [];
    let cnt = 0;
    return new Promise((resolve) => {
        for(let i=0;i<arr.length;i++) {
            arr?.().then((res) => {
                result[i].value = res;
                result[i].status = 'fulfilled';
                cnt++;
                if (cnt === arr.length) resolve(result);
            }, (err) => {
                result[i].value = err;
                result[i].status = 'rejected';
                cnt++;
                if (cnt === arr.length) resolve(result);
            })
        }
    })
}
```

2. 版本号排序
```javascript
const versions = ["1.2.3", "1.2.10", "1.2", "1.0.2", "1.0"];
const versionSort = (arr) => {
    const n = arr.length;
    return arr.sort((a,b) => {
        const versionsA = a.split('.');
        const versionsB = b.split('.');
        for(let i=0;i<3;i++) {
            const a = Number(versionsA?.[i]) || 0;
            const b = Number(versionsB?.[i]) || 0;
            if (a === b)    continue;
            else return a-b;
        }
    })
}
console.log(versionSort(versions)); // 输出: ['1.0', '1.0.2', '1.2', '1.2.3', '1.2.10']
```

3.写一个LazyMan
```javascript
const LazyMan = (name) => {
    console.log(`Hi I am ${name}`);
    let promise = Promise.resolve();
    const obj = {
        eat: (food) => {
            promise.then(() => {
                console.log(`I am eating ${food}`);
            })
            return obj;
        },
        sleep: (time) => {
            promise = new Promise((resolve) => {
                setTimeout(() => {
                    resolve();
                }, time * 1000);
            })
            return obj;
        },
    }
    return obj;
}

LazyMan('Tony');
// Hi I am Tony
 
LazyMan('Tony').sleep(10).eat('lunch');
// Hi I am Tony
// 等待了10秒...
// I am eating lunch
 
LazyMan('Tony').eat('lunch').sleep(10).eat('dinner');
// Hi I am Tony
// I am eating lunch
// 等待了10秒...
// I am eating diner
```

4. 括号字符串校验
```javascript
// 给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。
// 有效字符串需满足：
// 1. 左括号必须用相同类型的右括号闭合。
// 2. 左括号必须以正确的顺序闭合。
// 3. 每个右括号都有一个对应的相同类型的左括号
const isValid = (str) => {
    const n = str.length;
    if (n === 0)   return true;
    if (n % 2 === 1)   return false;
    const a = str[0];
    const b = str[n-1];
    if ((a==='(' && b===')') || (a==='[' && b===']') || (a==='{' && b==='}')){
        const newStr = str.slice(1, n-1);
        return vaildStr(newStr);
    }   else return false;
}
console.log(isValid("({[]})")); // true
console.log(isValid("({])"));
console.log(isValid("({[}])"))
```

## 2026.1.27 阿里淘天一面（业务技术-RPO专场）
1.前端类似于npm包，像树状，后端更多的是一些外置依赖，怎么让AI去完成整个仓库的理解 —— 比如有个支持读取依赖库代码的MCP工具，怎么设计这一整个工作流

2.了解过code wiki、cursor等工具的设计原理吗

3.MCP和Skill的差别，为什么需要SKill

4.请求竞态是其中的一类问题，如何设计一个可插拔的方式去做问题上报

5.笔试题

  5.1 请写出一个函数，解析window.location.search

  ```javascript
  // 如?a=1&b=1&c.d=2
  // 解析为{a:1,b:1,c:{d:2}}
  function transferSearchToObj(str) {
    const searchStr = str.includes('?') ? str.slice(1) : str;
    const arr = searchStr.split('&');
    const resultObj = {};
    for (let i = 0; i < arr.length; i++) {
   const item = arr[i];
   if (item.includes('.')) {
     const keys = item.split('.');
     const firstKey = keys?.[0];
     resultObj[firstKey] = transferSearchToObj(item.slice(firstKey.length + 1))
   } else {
     const [a, b] = item.split('=');
     resultObj[a] = b;
   }
    }
    return resultObj;
  }
  console.log(transferSearchToObj("?a=1&b=1&c.d=2"))
  ```

  5.2 国际化翻译。提供了A格式代码，请写出函数将其转化为B格式

  ```javascript
  // A
  const A = {
    hello: "你好",
    page: {
      home: "首页",
      activity: "活动",
      more: {
        a: 1,
        b: 2,
        d: {
          x: {
            y: {
              z: 2
            }
          }
        }
      }
    }
  }
  // B
  const B = {
    hello: "你好",
    "page.home": "首页",
    "page.activity": "活动",
    "page.more.a": 1,
    "page.more.b": 2,
    "page.more.d.x.y.z": 2
  }
  function flattenObj(obj) {
    const result = {};
    for (const key in obj) {
      const value = obj[key];
      if (value !== null && typeof value === 'object') {
        const inObj = flattenObj(value);
        for(const inKey in inObj) {
          const inValue = inObj[inKey];
          result[`${key}.${inKey}`] = inValue;
        }
      } else {
        result[key] = value;
      }
    }
    return result;
  }
  console.log(flattenObj(A));
  ```

  5.3 真实电商场景模拟

```javascript
// 正在为某电商平台开发购物车页面。
  // 平台支持多种优惠类型同时使用，但不同优惠之间存在互斥规则和叠加优先级。
  // 你的任务是：根据用户选择的商品和可用优惠券，计算出订单的最低实付金额。
  // 当前支持的优惠类型如下：
  
  // 满减券（FullReduction）
  // 示例：满300减50
  // 可与其他非互斥优惠叠加
  
  // 折扣券（Discount）
  // 示例：9折
  // 不能与满减券同时使用（互斥）
  
  // 包邮券（FreeShipping）
  // 仅减免运费（固定运费为 ¥10）
  // 可与任意优惠叠加
  
  // 注意：用户最多只能选择一种满减券或一种折扣券（二者互斥），但可以额外选择包邮券。
  
  const calculateMinPayment = (products, coupons) => {
    // dfs，入参是已经用了的卷和现在的价格，函数功能是找到这个链路花的最少的钱
    const dfs = (money, used) => {
      let minMoney = money;
      for(let i=0;i<coupons.length;i++) {
        const item = coupons[i];
        if (used.find(x => x.type === item.type)) continue;
        if (item.type === 'discount' && !used.find(x => x.type === 'full_reduction')) {
          // 满减卷
          const nextMoney = money * 0.9;
          minMoney = Math.min(minMoney, dfs(nextMoney, used.concat(item)));
        }
        if (item.type === 'full_reduction' && !used.find(x => x.type === 'discount')) {
          // 折扣卷
          const nextMoney = money - item.amount * Math.floor(money / item.threshold);
          minMoney = Math.min(minMoney, dfs(nextMoney, used.concat(item)));
        }
        if (item.type === 'free_shipping') {
          // 包邮卷
          const nextMoney = money - products.length * 10;
          minMoney = Math.min(minMoney, dfs(nextMoney, used.concat(item)));
        }
      }
      return minMoney;
    }
    const totalPrice = products.reduce((a,b) => a?.price * a.quantity + b.price * b.quantity, {
      price: 0,
      quantity: 0,
    }) + products.length * 10;
    return dfs(totalPrice, []);
  }
  
  // // 示例1:
  // const products = [
  //   { price: 100, quantity: 2 }, // 总价 200
  //   { price: 150, quantity: 1 }  // 总价 150 → 合计 350
  // ];
  
  // const coupons = [
  //   { type: 'full_reduction', threshold: 300, amount: 50 },
  //   { type: 'discount', amount: 0.9 },
  //   { type: 'free_shipping' }
  // ];
  
  // console.log(calculateMinPayment(products, coupons));
  // // 方案1: 满300减50 + 包邮 → (350 - 50) + 0 = 300
  // // 方案2: 9折 + 包邮 → 350 * 0.9 + 0 = 315
  // // 最优为 300.00
  
  
  // 示例2:
  const products = [{ price: 50, quantity: 1 }]; // 总价 50
  const coupons = [
    { type: 'full_reduction', threshold: 100, amount: 20 },
    { type: 'free_shipping' }
  ];
  console.log(calculateMinPayment(products, coupons));
  // 满减不满足门槛，只能用包邮
  // 实付 = 50 + 0 = 50.00
```

