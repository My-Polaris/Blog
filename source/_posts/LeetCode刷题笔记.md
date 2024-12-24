---
title: LeetCode刷题笔记
date: 2021-07-24 00:26:26
tags: 算法
---

**削微做下笔记**

<!--more-->

## 算法入门康复训练

题集：leetcode每日一题，[leetcode算法入门与基础](https://leetcode-cn.com/study-plan/algorithms/?progress=xeb1gac)，[leetcode剑指offer第2版](https://leetcode-cn.com/problem-list/xb9nqhhg/)

C++1s跑1e8~1e9，Leetcode默认1s跑完；INT_MAX略大于1e10，大于2e9的数据开long。二进制左移防越界得开unsigned int，long不好使，具体参考[剑指 Offer 65. 不用加减乘除做加法](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)。

#### 1.关于二分查找中left的定位

其实二分查找已经是典中典了，和之前的排序一样，二分的写法太多了，所以确定自己的写法标准很重要。

标准模板：
```c++
int searchInsert(vector<int>& nums, int target) {
    int mid,left=0,right=nums.size()-1;
    while(left<=right)//①(left<right)
    {
        mid=left+(right-left)/2;
        //之所以不用(left+right)/2的写法是因为left+right是有可能会造成溢出的,而用减法则避免了溢出
        if(nums[mid]<target)    left=mid+1;//left此前的数都<target
        else    right=mid-1;//②right=mid;
    }
    return left;
}
```
**left的定位：不满足判断条件的第一个元素。**

需要注意的是，

1.如果整个数组都<target时，那么left是位于r+1的，这个时候如果用left访问数组的话就会越界，要小心。

2.搜索区间为： [left, right]，left返回值[0,right+1],right返回值[-1,right],mid取值[0,right-1]

**拓展**

例题：[寻找峰值](https://leetcode-cn.com/problems/find-peak-element/) [剑指 Offer 11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

在某些时候，我们会用到``(left<right)``的写法，不同处参考标准模板代码块的①和②

特定的场景诸如：1.判断条件实时改变，为防止越界。2.某些情况需要应用right=mid以避免错过目标值。

这一模板的搜索区间为[left,right)，若初始化right=nums.size()-1，那么当整个数组中都<target时,返回值依旧是最后一个元素。

**背板子是走不到最后的，深刻理解+灵活运用才是王道。**

#### 2.双指针

例：[让数组的元素均向右移动k位](https://leetcode-cn.com/problems/rotate-array/)。

O(1)方法：**环状替换**，**旋转数组**。

例：[删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

方法：**快慢指针**。

例：[环形数组是否存在循环](https://leetcode-cn.com/problems/circular-array-loop/)

方法：**快慢指针**

理解：类似于判断链表是否有环。快指针要么走到终点——无环，要么走入循环——快慢一定会在环里相遇。

C++11新标准：[Lambda表达式创建匿名函数](https://www.cnblogs.com/DswCnblog/p/5629165.html) [C++之function函数](https://www.cnblogs.com/wanghao-boke/p/12239959.html)

例：[剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)

理解：先走一遍摸清楚两链表长度，接着长的提前走x步，这样就能同时到达交点。

例：[11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

理解：这题其实关键是不好想到双指针，提前告知label就没有难度了。

例：[475. 供暖器](https://leetcode-cn.com/problems/heaters/)

理解：排序+二分 || 排序+双指针，一开始会被以供暖器为核心的思路带偏，想着要让供暖器去覆盖某些离他近的房子，但是“近”这个关键词是需要通过比较的，没有两个供暖器的比较根本不知道要由哪个供暖器来覆盖这个房子，只有以房子为核心，找寻离它最近的供暖器，再去计算供暖器需要给出的半径，这才是正确的思路。

#### 3.滑动窗口

例：[在s中找出无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

特点：连续性，去重——unordered_set容器。

方法：用容器的find()方法查看right元素是否存在于set中，存在则左指针右移直至不存在再插入；维护right-left+1。

例：[判断s2是否包含s1的排列](https://leetcode-cn.com/problems/permutation-in-string/)

特点：连续性、记录数组。

错解：全排列会超时。

方法：滑动窗口固定长度为s1.size()，当t(滑动窗口)记录数组==s1记录数组时，包含。

例：[最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

特点：连续性、记录数组——unordered_map容器。

方法：（简便描述：右滑——右指针右移并记录，左滑——左指针右滑删去记录）

右滑，遇必需字符(hs[s[i]]<=t[s[i]])时计数cnt，cnt==t.size()有解；有解时，左滑产生当下最优解后记录解，保持有解的状态继续右滑即可，因为非当下最优解的时候会左滑产生最优解。

#### 4.STL容器

例：[ 矩阵中战斗力最弱的 K 行](https://leetcode-cn.com/problems/the-k-weakest-rows-in-a-matrix/)

特点：优先队列priority_queue+二分，pair容器

方法：以vector<pair<int,int>> record记录军队人数与索引，以priority_queue q(greater<pair<int,int>>(),move(record))赋值小顶堆(小的排前面),其中move函数用于强制将参数转换为右值引用。(可以用sort()代替优先队列)

细节：

1.STL容器——pair，头文件：#include<utility>

类似于结构体，以.first，.second访问俩元素，以make_pair(a,b)正式形成容器，排序时默认先排first再排second。

2.C++11新标准——emplace_back

类似于push_back，优点在于可以直接构造容器并建立右值引用，而不像push_back需要创建临时对象+拷贝。

如可以record.emplace_back(a,b)用a,b创建pair容器，而用push_back则需record.push_back(make_pair(a,b))。

3.C++11新标准——std::move()

把传进来的参数强制转换成右值引用。

4.优先队列——priority_queue，头文件：#include<queue>

其实就是最大堆 与最小堆，定义：``priority_queue<Type, Container, Functional>``，对应数据类型、容器、比较方式。

```c++
//升序队列,最小堆
priority_queue <int,vector<int>,greater<int> > q;
//降序队列,最大堆
priority_queue <int,vector<int>,less<int> >q;
//方法与队列基本相同
top 访问队头元素
empty 队列是否为空
size 返回队列内元素个数
push 插入元素到队尾 (并排序)
emplace 原地构造一个元素并插入队列
pop 弹出队头元素
swap 交换内容
```

#### 5.最短路

例：[求最短路中的最长路径](https://leetcode-cn.com/problems/network-delay-time/)

特点：Dijkstra算法,*max_element()函数

方法：建立邻接矩阵后求得起点k至各点的最短路dist[]，再从最短路中找出最长的路径。

#### 6.动态规划

> 动态规划适用范围：计数、求最大最小值、求存在性
>
> dp解题四部曲：
>
> 1.确定状态——即确定dp数组的含义。
>
> 2.确定转移方程——从上一步怎么得出下一步
>
> 3.确定初始条件——转移方程无法求出，但又需要的数据
>
> 4.确定遍历顺序——确保转移方程运行
>
> 方法：由最后一问得出子问题。
>
> 如：最少用多少枚2,5,7硬币凑出27。则最后一步为f（27 - ak）+ ak(ak为2或5或7)，那么f（27 - ak）即为最少用多少枚硬币凑出27 - ak，这就是子问题；
>
> 所以：dp[x]——最少用多少枚硬币凑出x；转移方程——``f[x]=min(f[x-2]+1,f[x-5]+1,f[x-7]+1)``；初始条件：f(0)=0，0元由0枚硬币组成；遍历顺序：由小到大。

##### 基础

例题：[剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

##### 拓展

例题：[最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

题型：序列DP

最后一问与子问题：区间[i,j]的最长回文子序列由[i+1,j-1]得出，若s[i]==s[j]则+2，否则单独加入i或j取最大值。

```c++
1.确定状态:dp[i][j]——区间[i,j]的最长回文子序列的长度
2.转移方程:if(s[i]==s[j])	dp[i][j]=dp[i+1][j-1]+2;
		 else	dp[i][j]=max[dp[i][j-1],dp[i+1][j]];//加入i或加入j，取最大值
3.初始条件:dp[i][i]=1,其余=0——单个字符的回文长度为1(转移方程无法得出)，其余一开始应该都为0
4.遍历顺序：首先j=i+1,接着i是由i+1赋值的，所以i得从n-1开始。
```

同序列DP：[最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

例题：[优美的排列](https://leetcode-cn.com/problems/beautiful-arrangement/)

题型：状态压缩DP（暴力DFS也能过）

最后一问与子问题：向第n位填入合理数据,同时加入未填入这第n位时的方案数。

tips：__builtin_popcount()函数用于统计数字在二进制下“1”的个数。

```c++
1.确认状态:f[x]表示状态为x时的可行方案数
2.状态压缩:x=10100表示用到了5和3，x=11..11表示数字全都用到了,即最终状态
3.转移方程:循环n位，若遇到1且满足因数倍数条件就加上"尝试将它去除"的状态,如f[100110](向第三位填入数据)=f[000110](填入6)+f[100010](填入3)【+f[100100](填入2)】;3,6可,2不行
4.初始条件:f[0]=1,因为f[1]=f[0]=1
5.遍历顺序:二进制串遍历，即从0~(1<<n)
```

区间DP：[ 猜数字大小 II](https://leetcode-cn.com/problems/guess-number-higher-or-lower-ii/)

这一题其实是小博弈,在1~n猜哪个数会使花费最少,如果作为博弈题,那么这个选数x是有规律的;而这里没有规律,那只好直接遍历x的选择,走记忆化dfs | 区间dp。

```c++
1.确定状态:dp[i][j]——区间[i,j]中确保获胜的最小现金数
2.转移方程:dp[i][j] = min(dp[i][j], max(dp[i][x-1], f[x+1][j])+x);
3.初始条件:dp[i][i]=1,其余=0——只有1个数的话直接猜对，不需要花钱
4.遍历顺序：最终目标是dp[1,n],所以考虑i=n-1倒序,j=i+1顺序,确保了i到j之间的所有子区间已经赋值。
```

> 在我的理解中,不少DP是可以由记忆化DFS代替的,前者是遍历,后者则是递归。

概率DP：[剑指 Offer 60. n个骰子的点数](https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/)

```c++
1.基本思路：扔一个骰子，会得到1~6的点数；扔两个骰子，会得到2~12的点数；扔n个骰子，会得到n~6*n的点数。
2.☆最后一步：当我扔第n个骰子的时候，得到的点数总和 = 扔第n-1个骰子得到的点数总和 + 这次得到的点数。
3.假设我扔完后点数为N，那么最终点数为N的概率就为：
	上个人扔完后为N-1，我扔出1的概率 + ... + 上个人扔完后为N-6，我扔出6的概率。
4.所以转移方程为：dp[n][N] = (dp[n-1][N-1]+dp[n-1][N-2]+...+dp[n-1][N-6])*1/6
```

#### 7.字典树(又称前缀树)

> 前缀树 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。
>
> 这一数据结构有相当多的应用情景，例如自动补完和拼写检查。

```c++
class Trie {
private:
    vector<Trie*> child;//子树,字典树的话长度为26('a'-'z')
    bool isEnd;//确定是不是这个单词

    Trie* searchPrefix(string word){//找前缀字符串,找到就返回最后字符位置,找不到就返回空
        Trie* node = this;
        for(auto c:word){
            if(node->child[c-'a']==nullptr){
                return nullptr;
            }
            node=node->child[c-'a'];
        }
        return node;
    }
public:
    Trie() :child(vector<Trie*>(26)),isEnd(false){//初始化构造函数
    }
    
    void insert(string word) {//向前缀树中插入字符串 word
        //根据其word的字符找到最后一个字符,并将其isEnd赋值为true
        Trie* node = this;
        for(auto c :word){
            if(node->child[c-'a']==nullptr){
                node->child[c-'a'] = new Trie();
            }
            node=node->child[c-'a'];
        }
        node->isEnd=true;
    }
    
    bool search(string word) {//如果字符串 word 在前缀树中，返回 true
        if(this->searchPrefix(word)==nullptr) return false;//前缀都找不到就false
        return this->searchPrefix(word)->isEnd;//找到了就返回他的isEnd,因为有可能存在abcd但不存在abc
    }
    
    bool startsWith(string prefix) {//如果之前已经插入的字符串 word 的前缀之一为 prefix ,返回 true
        return this->searchPrefix(prefix) != nullptr;//找到前缀就true,不用管到底存不存在这个单词
    }
};
```

例题：[实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)    [添加与搜索单词 - 数据结构设计](https://leetcode-cn.com/problems/design-add-and-search-words-data-structure/)

#### 8.奇怪的知识

在遍历二维vector数组时，用for(auto &a:b)将增快速度，关键在于这个&可以省去构造新vector的时间，而且也比for i,j的用法要快，在[搜索二维矩阵 II](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/)这一题中竟然卡了这个点的超时，按理来说这题是不会卡O(mn)算法的,因为最大时间也只是O(300*300)，而这题的O(mn)算法只有用到&（而且要用在第一层遍历）才能过，不知道是不是评测机的问题..

之前也有遇到过卡多重vector的情况，[学生出勤记录 II](https://leetcode-cn.com/problems/student-attendance-record-ii/)的记忆化搜索解法用到三重vector会超时，而用普通的三维数组却能过，所以如果在复杂度较接近临界时，要尽量避开用多重vector。

#### 9.单调栈

用于解决`下一个更大元素`问题，偏冷门。

例题：[下一个更大元素 I](https://leetcode-cn.com/problems/next-greater-element-i/)

简单来说就是：从后往前遍历，把比当前元素小的都出栈。

类比于：看后面比自己高的人(考虑遮挡)——显然,两个高的人之间是不需要有人的,因为他们不再会被看到；同时，栈的特性也让自己能一眼看到比自己高的第一个人。

#### 10.位运算经典

例题：[只出现一次的数字 III](https://leetcode-cn.com/problems/single-number-iii/)

挺经典的，用位运算找出数组中的非重复元素，但这题关键在于有俩非重复元素，已知这俩的异或值，怎么在数组里找到这俩数呢？这里的处理挺巧妙的，找出这个异或值的最低位1，显然这个1肯定由该位为1的数与该位为0的数异或得出，那么就可以在此基础上分类，让所有该位为1的数进行异或运算就会得到A数，所有该位为0的数进行异或运算就会得到B数。

#### 11.二叉树的前/中/后序遍历

例题:[剑指 Offer 07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

算是回顾很久以前学的知识了,但之所以记下这题主要还是因为这题的预处理挺精髓的，正常来讲我们是每次都需要遍历中序遍历找根，但是如果我做个预处理，以unordered_map<int,int>记录key与value的反转，那么当我想找对应value的key时,就可以直接通过下标找到。

#### 12.弗雷耶兹(Fisher-Yates)随机算法

例题：[打乱数组](https://leetcode-cn.com/problems/shuffle-an-array/)

保证概率均等的打乱数组方法。

#### 13.需要扣细节的模拟题

例题：[剑指 Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

记下来只是因为自己在细节模拟上一直很菜,这道题第一次写也花了好久,希望日后自己回来写这些题会更迅速、得心应手。

例题:[剑指 Offer 29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

螺旋矩阵青回了,不一样的是这次是矩阵而不是方阵,总结来说自己的处理有两点没做到位。

1.它不是方阵了,那么只剩下一行与只剩下一列的处理就很关键,我的解法是直接用visit数组处理,显然这不够优雅。

2.自己解法的边界划的不清晰,没有想到while(left<=right && top<=bottom)而是直接用了min(m,n)/2

#### 14.快速幂的拓展

例题：[超级次方](https://leetcode-cn.com/problems/super-pow/)

求a^b次方，但b是大数。由于mod是针对于^运算的，所以关键点就在于将a^(b\*10)变为a^10^b,将\*运算变为^运算之后就可以理想应当的取模啦。

#### 15.map与unordered_map的差别

使用场景上，当你需要实现**映射**或**离散化**时，这俩都会大有帮助。

差别上来讲，map的底部实现机制为红黑树，红黑树内部自然是**有序**的，它也默认会以key的大小进行排序(从小到大)，当然可以加入第三个参数``greater<int>``实现从大到小排。map的查找、插入、添加基本都为O(logn)。

而unordered_map是直接建立一张散列表存放对应key与value，既然是哈希表那肯定是**无序**的，建立上也会比map略高一些，但是查询的复杂度可以达到O(1)，就像给你下标去访问数组一样。

在算法题上，我想并不会卡你建立与查询的时间，使用场景更多的是要考虑到你需不需要插入的数据是有序的。

参考博文：https://blog.csdn.net/BillCYJ/article/details/78985895

#### 16.浅记一下有限状态自动机

例题：[剑指 Offer 56 - II. 数组中数字出现的次数 II](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)

在这题中，我的理解就是给每个二进制位加状态，但由于有三种状态，所以需要一个中间变量two来区分00状态与10状态；而由于二进制的处理都是一致的，所以直接对整数进行位运算处理即为相应的32次二进制位运算处理。

具体的实现就是不停的状态转换，最后two=0,one=1的二进制位就是所求的结果。

#### 17.快速乘

例题：[剑指 Offer 64. 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/)

假设A\*B，其实就是将 B 二进制展开，如果 B 的二进制表示下第 i 位为 1，那么这一位对最后结果的贡献就是 A\*(1<<i) ，即 A<<i，我们遍历 B 二进制展开下的每一位，将所有贡献累加起来就是最后的答案。

这题还有一个关键，就是利用``逻辑与运算``代替``条件判断语句``。

#### 18.atoi()与stoi()的区别

atoi接受char *类型的参数，需要用c_str()配合使用将string转为char *；stoi则接受string类型的参数。

使用atoi好处是可以防止越界报错，越界会输出边界(INT_MAX或INT_MIN)；而stoi则只是使用方便。