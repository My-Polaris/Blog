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

## 2026.2.3 京东一面（京东小程序）
1. 个人职业规划，使用AI的经验，有没有用过cursor、Skills和rule、RAG工作原理、是vibe coding还是spec coding、新仓库怎么让AI按照团队的代码规范进行coding、大模型应用的温度是什么原理
2. 挑一个项目讲，我讲了竞态，诊断与治理，axios拦截器、Proxy方案等，然后我提到洋葱模型他反问了下洋葱模型是什么数据结构实现的，是不是能直接使用reduce能实现，我说能
- 这里axios的实现应该是，数组存所有拦截器，然后请求拦截器unshift进去，响应拦截器push进去，最后用reduce串成promise链
- 还问到为什么要用Reflect，Reflect本质是提供原生的调用，避免副作用
3. 笔试题,二选一实现
```javascript
// ## 题目一：
// 给定一个 没有重复 数字的序列，返回其所有可能的全排列。

// 示例:

// 输入: [1,2,3]
// 输出: [ [1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1] ]

// ## 题目二：
// 利用 Promise 使用 Promise.all
```
4. 反问环节直接问面评：整体没什么问题，竞态项目的表述不够清晰、他觉得业内应该有一些成熟的方案、AI方面多了解快手内Kwaipilot的和cursor、Claude Code啥的对标能力
个人体验：面试官基本全程面瘫+苦脸，跟阿里淘天的业务技术有点像，给人感觉就是挂定你了

## 2026.2.4 百度一面（百度文库）
总体比较快，就35分钟的样子
1. 问vue2和vue3的响应式区别
2. 问简历的三个项目
3. 一道笔试题：手写深拷贝
4. 反问环节问面评：逻辑思维整体没问题，挺好的

## 2026.2.4 京东二面（京东小程序）
1. 讲一讲三个项目，以及分别有什么难点、复杂点、遇到过什么问题
第一个项目：基于AI的CR能力增强
- 背景：团队在做故障预防，我作为故障预防工作的主R，其中的一个方向是对同质性问题做预防避免二次发生。具体讲需要分为「经验沉淀」和「工具化」两个步骤，经验沉淀就是找到这些同质性问题，工具化则是以诸如ESLit插件或智能CR的方式预防住。
- 挑战点：在项目初期我们通过人工review团队的CR内容和缺陷单找到这些同质性问题，但由于缺陷单仅有问题描述没有解决方案，大部分缺陷单看不懂，导致经验沉淀效率低，我们需要减少整件事的人力投入成本，提高效率。
- 解决方案：将经验沉淀来源从「问题发现」转向「问题发现」，设计AI工具，辅助我们找到并分析MR中的修复类变更，完成智能CR规则的自动生成。具体讲工具的设计思路是，识别commit标题作为问题描述，commit内容作为解决方案，进行规则生成，工具最终版的落地形式是一个定时工作流，会定期的去找到仓库内的MR并进行分析与规则生成。
- 遇到过什么问题：1. 生成规则效果差 —— 我们会分两种情况，一种是单点问题，这种情况我们需要对规则做单点优化，一种是规则生成的共性问题，这种需要通过调整提示词或优化工作流的方式，比如加入规则质检和去重等步骤；2. 自动化程度差，最开始的形式是本地MCP+自主提问，后续演变成线上的自动化工作流，再变成定时执行的工具。
- 难点：工作流和提示词的设计与优化

第二个项目：中后台聊天室场景建设
- 背景：在直播聊天室场景中，给到中后台展示的直播流缺少上麦嘉宾的信息，比如有哪些嘉宾在麦上、头像、开闭麦状态、谁在说话等。根因是C端的这些展示是客户端代码绘制出来的，要保持一致的话B端也得画。
- 挑战点：要完成B端的绘制工作，除了要复刻客户端的实现以外，还缺少充足的数据源。1. 不同于C端是客户端，B端是H5，天然缺少Native能力；2. 不同于C端只有直播，B端还需要支持回放。
- 解决方案：解决方案分「绘制思路」和「数据源组合思路」。1.绘制上讲，聊天室直播下有7种细分场景，布局各有不同，可以按特征分为能开摄像头的不能开摄像头的，能开摄像头的，我们的绘制需要一比一与流内容对其，然后在某个嘉宾开摄像头的时候通过「挖孔」的方式把它的视频内容露出来，不能开摄像头的则只需要完全把流内容盖住，纯前端绘制布局就行了。2.数据源组合上需要四类数据源，第一类数据源是C端的直播实时数据（拿到直播基本数据），第二个是直播PB数据（补充非Native拿不到的数据），第三类是直播埋点数据（补充回放场景下的数据），第四类是业务数据（公会专属信息）。按照以上方案我们沉淀了复用物料，快速完成了4个平台的落地。
- 遇到过什么问题：1.数据源可靠性问题，数据源大多是开播工具上报的，不同开播工具上报的形式或字段会有差异，或者是某些数据只在部分客户端版本以后才上报，或者是生产的数据在不同视频流压缩后会丢失，所以需要理清每个数据源的完整生产消费链路，做好兜底避免成为线上问题；2. 数据间的时差问题，比如直播类型数据比直播流快，提前切换绘制方式后和底流内容对不上，所以要明确好数据间是否有时差，有时差时先考虑能否用没有时差的字段做替换，实在没得换就做特殊兜底逻辑避免页面错乱。
- 难点：实现过程比较琐碎，难点在调研与方案设计，因为是司内首例

第三个项目：公会请求竞态问题
- 背景：公会请求竞态问题客诉频发，需要对100+页面300+接口做竞态诊断（偶现性）+治理。
- 挑战点：传统方案成本太高，需要工具化提效。
- 解决方案：设计请求竞态工具，在请求拦截层统一完成识别和治理。
- 遇到过什么问题：1. 轮询场景可能导致饥饿，上一个请求未响应下一个请求已发出，导致始终在等。方案是维护最后一个请求发出的时间，响应时该请求的发出时间小于「最后一次响应的请求的请求发出时间则不响应」
- 难点：诊断的特征提取，处理的多场景覆盖（忽略请求、顺序返回等）

2. 故障预防除了智能CR还有哪些方面能做功
- 故障预防是稳定性体系中的事前环节，在事前环节的每个环节都能做功。
- 开发前：PRD可行性分析/风险评估、技术方案评审。
- 开发中：ESLint插件、CDN容灾。
- 提测前（静态）：人工/智能CR、性能检测、代码质量检测、安全准入。
- 测试中（动态）：功能测试、性能测试、兼容性测试、降级测试、安全测试。
- 上线前：上线计划、变更通知、审批、分级/灰度发布
- 上线后：线上日志观测、报警
3. Vue响应式原理、数组扁平化的思路
4. 除了第一个项目还有什么AI的应用
- 在日常业务中，会有Vibe Coding，也会有基于Skills让AI去做的case
5. 反问环节问面评：一个是讲项目讲的不够清晰，一个是简历上体现最新技术的点比较少

## 2026.2.6 百度二面（百度文库）
1. 平时有没有写webpack或者vite配置
- 基本没有，很少
2. Vue函数式挂载组件怎么写？
- let instance = createApp，instance.mount(某个元素),document.body.appendChild(某个元素);
3. 用户信息怎么不被读取？
- HttpOnly
4. 怎么防止URL中脚本注入
- 识别并过滤script标签
5. 写一个函数，找到URL中某个key的值
- new URL();
6. 在用webpack写项目的时候，这个项目非常大，可能有很多路径的入口，怎么进行一个配置
- 我直接没get到
总结：考察的都是非常具体的知识，细到某个API长什么样，但是面试官的表述非常难评，这些问题都是我反复问几轮才确定她要问的是什么。最后24分钟就结束了，估计她觉得我不具备webpack的能力...

## 2026.2.10 京东三面（京东小程序）
1. 智能CR这个事，怎么对知识库做的存储与查找
- 有成熟基建，我们就是直接把规则给到他们的数据库，同时回答了RAG
2. 故障预防还做了哪些事
- 从研发流程各个阶段，做公司基建的接入，没有对应基建则做自己实现，比如性能准入、安全测试等
3. 上线后灰度的观测怎么做的
- 做px链路上报，自定义事件报警
4. 报警策略怎么定
- 看重要程度，核心链路可能有pv立刻上报，有些可能有符合预期的case则做一定的阈值提高
5. 单测这个点，是测试给你们的单测吗，怎么做单测覆盖率
- 没做单测覆盖率，方案只能想到给到PRD和单测规范，让AI做单测用例生成。（面试官回答说可以在执行位置坐行打点）
6. 性能指标了解过吗，有哪些，分别有什么差别
- FMP、FP、FCP、LCP、TTI
7. FCP和LCP的差别
- 加载vue元素就上报，渲染区域到达一定占比后才上报
8. LCP会上报多次吗
- 会（蒙的），每次有更大元素时就会再触发一次，以用户交互前的最后一次LCP为最终值
8. 浏览器从输入到渲染完成发生了什么
- DNS域名解析 -> 建立TCP链接 -> TLS握手 -> 请求资源 -> 响应资源 -> 渲染进程加载HTML（DOM树渲染、CSSOM树渲染，渲染树渲染）-> 加载js资源 -> 之后就是vue的执行逻辑了
9. 做性能优化有哪些策略
- 网络层：网络带宽、CDN、缓存与离线包、HTTP2.0
- 资源大小层：图片webp、图片压缩、字体压缩、gzip、js资源压缩（terser、treeShaking）
- 加载策略层：预请求、懒加载、按需加载、分包、SSR
10. 你说到强缓存，哪些资源适合做强缓存，项目里是怎么做的强缓存
- 除了HTML和JSON类外其他资源都适合做缓存，尤其是css资源、js资源、图片资源等，基建部署平台做的项目Owner不关注（我理解就是直接给资源加响应头Cache-Control就好了）
11. 聊天室能力建设，面试官误解了聊天室的意思，问我聊天室怎么做
12. 了解过直播推拉流的一些协议
- 没有，都集成到成熟播放器基建内了
13. 客户端怎么做的数据获取和页面适配
- 数据获取我说是通过websocket，他说是从流里取的，我说因为流生产来源于很多开播工具，所有只要用的是信令数据，流数据是做辅助（是否打开摄像头），他说要从流里取分辨率啥的
- 页面适配我理解是用vwvh、rem等方式做的宽度适配吧
14. 微前端架构你有了解吗，都怎么做的沙箱
- 了解single-spa，qiankun、wujie，沙箱基于shadow dom、也有基于iframe的

面评：
1. 对于一些高可用架构设计接触的比较少，全局视野的一些缺失，更多的是专注在自己那一块，比如对页面渲染流程有些点接触的不够深入，直播推拉流协议的不了解等
2. 本身vue技术栈+PC的组合在市场的需求比较少，市场更多的需要React
也不是说不过，总体的一些基础和学习能力还行，等面试结果通知吧

## thinking
收到offer后，了解清楚团队架构、薪资结构、福利待遇等等，然后直接接或者思考一两天后接
然后入职时间尽量往后推，可以用办理离职、工作交接、把公司年假休完等等原因，期间可以继续面试，然后有成的就继续接，尽量在「完全确定」后或者「完成一家公司的入职」后才做反悔的动作，反悔的时候用「个人职业规划需要重新考虑机会(比如创业或休息一段时间之类的)+希望未来还能有机会共事，保持联系」或者「临时收到了另一家公司的offer+个人职业规划考量+后续会推荐更合适的人选」
