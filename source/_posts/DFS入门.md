---
title: DFS入门
date: 2020-02-08 17:02:58
tags: 算法
---

**——经典例题**

<!--more-->

### 题目一：7-39 求迷宫最短通道 (20分)

<img src="DFS入门\求迷宫最短通道1.png">

<img src="DFS入门\求迷宫最短通道2.png">

### 了解深度搜索

其实深度搜索就是``用递归去遍历``的一种算法，讲究一条路走到黑。

### 思路

递归模板：

```c
void dfs(int step)//深度搜索函数
{
        判断是否满足边界//如迷宫中是否到达终点
        {
            进行相应操作，输出或计数之类的//如迷宫中记下步数
        }
    若超出边界//如迷宫撞墙了
    {
        return ;//结束
    }
        尝试每一种可能//比如迷宫的四个方向，就用i<4的循环
        {
               if（check）//判断语句，也可以没有的，如迷宫中表示各个方向
               标记或计数+1//如迷宫中标记走过了这一步
               继续下一步dfs(step+1)//进入下一步的选择
                //执行到这里就代表前面的路径已经尝试过了，开始走另一条路，如迷宫中走了左，现在开始进入下一个选择，在这一格里走一下右试试
               恢复初始状态（回溯的时候要用到）//其实就是取消之前的操作（如取消标记或计数-1）
        }
}   
```

### 代码区

```c
#include<stdio.h>
void dfs(int x,int y);//定义深搜函数，这里注意：x是行号，y是列号
int maze[10][10];//存放迷宫地图 
int n;//矩阵行列数 
int num=0;//统计步数 
int min=10000;//定义足够大，是用于存储最小步数的 

int main()
{
	int i,j;
	scanf("%d",&n);
	for(i=0;i<n;i++)//给出迷宫 
	{
		for(j=0;j<n;j++)
		{
			scanf("%d",&maze[i][j]);
		}
	}
	dfs(1,1);//以坐标（1，1）开始 
	if(min==10000)	printf("No solution");//如果min值没有改变过，代表并没有走到终点的路径
	else	printf("%d",min);//否则输出最小步数
} 


void dfs(int x,int y)
{
	int i;
	if(x==n-2 && y==n-2)//当到达终点 
	{
		if(num<=min)		min=num;
	}
	if(maze[x][y]==1)	return;//当撞墙了，或者这一格走过了，结束
	for(i=0;i<4;i++)//四个方向 
	{
		if(i==0)//向下 
		{
			num++;//步数+1
			maze[x][y]=1;//表示这一步走过了
			dfs(x+1,y);//尝试往这个方向走（）行号加1
            //开始回溯
			maze[x][y]=0;//当尝试完后要把该位置标为未走过，防止别的尝试走不了此通道 
			num--;//步数减一
		}
		if(i==1)//向右 
		{
			num++;
			maze[x][y]=1;
			dfs(x,y+1);//列号加1，即往右
			maze[x][y]=0;
			num--;
		}
		if(i==2)//向上 
		{
			num++;
			maze[x][y]=1;
			dfs(x-1,y);//行号减1，即往上
			maze[x][y]=0;
			num--;
		}
		if(i==3)//向左 
		{
			num++;
			maze[x][y]=1;
			dfs(x,y-1);//列号-1，即往左
			maze[x][y]=0;
			num--;
		}
	}
}
```

参考链接：https://blog.csdn.net/weixin_43272781/article/details/82959089

### 题目二：7-33 整数分解为若干项之和 (20分)

<img src="DFS入门\整数分解为若干项之和.png">

### 代码区

```c
#include<stdio.h>
/*整数分解为若干项之和（应该属于深度搜索中的全排列）*/ 
void dfs(int x);//深入搜索的函数（递归函数）
int a[40]={0};//定义全局数组，用于存放若干项 
int n;//用于接收数据，表示那个整数 
int k=0;//用于判断啥时候换行
int num=0;//表示是第几项，用于为对应的数组元素赋值
int sum=0;//表示各项的最终和，用于判断是否输出 
int main()
{
	scanf("%d",&n);
	dfs(1);//深入搜索的函数（递归函数）,传入的1其实是各项的起始点 
	return 0;
} 
void dfs(int x)
{
	int i;
	if(sum==n)//若尝试成功了 
	{
		k++;//尝试成功了就开始计数 
		printf("%d",n);
		for(i=0;i<num;i++)//输出赋值到num为止的n项
		{
			if(i==0)	printf("=%d",a[i]);
			else	printf("+%d",a[i]);
		} 
		if(num==1 || k%4==0) printf("\n");//逢4或者最后一个样例出换行
        //最后一个样例是指如7=7时，此时只输出了一项，num值应为1
		else	printf(";");//否则出分号 
	}
    
	if(sum>n)	return;//当sum超出范围的时候，就没有必要再算了 
    
	for(i=x;i<=n;i++)//遍历n项
	{
		a[num++]=i;//为第n项赋值 
		sum=sum+i; //加上第n项之后的总和
		dfs(i);//将i传入函数，这样其实是在尝试这个i能不能继续作为下一项（因为题目要求每一项以小为优先）
		//这里其实是在该项中的每一种数字（1~n）中尝试，
		//举例说就是它在尝试第7项中能填啥，都尝试完之后就开始回溯 
		//接下来是回溯 
		sum=sum-i; //当上一函数执行结束（执行成功输出了或者执行失败了就开始往回走） 
		num--; //举例说此时就是尝试完第7项的所有数字之后，开始尝试第6项能填的数字 
	} 
}
```

### 题目三：求1~8的全排列(这题才是最基础的dfs入门题)

**dfs解法：**

```c++
#include<iostream>
using namespace std;
bool visit[10];//标记数组
int path[10];//路径数组
void deal(int dep,int n)//dep表示现在所处的深度
{
    if(dep==n)//到达8了就输出路径数组
	{
        for(int i=1;i<=n;i++)
		{
            if(i==1)    cout<<path[i];
            else    cout<<" "<<path[i];
        }
        cout<<endl;
        return ;
    }
    for(int i=1;i<=n;i++)
    {
        if(!visit[i])//判断i有没有用过 
        {
            path[dep+1]=i;//下一步尝试用i 
            visit[i]=true;//表明i用过了 
            deal(dep+1,n);
            visit[i]=false;//回溯，i恢复原状 
            path[dep+1]=0;//下一步的状态清空,其实这里没有这句也行，因为状态是覆盖的 
        }
    }
}
int main()
{
	int n=8;
    deal(0,n);
}
```

**STL算法类函数next_permutation()的解法：**

```c++
#include<iostream>
#include<algorithm>
using namespace std;
int a[10];
int main()
{
    int n;
    n=8;
    for(int i=1;i<=n;i++)    a[i]=i;
    do{
        for(int i=1;i<=8;i++){
            if(i==1)    cout<<a[i];
            else    cout<<" "<<a[i];
        }
        cout<<endl;
    }while(next_permutation(a+1,a+n+1));//操作对象是数组,string也行,参数是开头与结尾后一位的地址
    return 0;
}
```

