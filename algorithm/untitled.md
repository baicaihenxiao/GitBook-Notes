# 差分数组是个啥？能干啥？怎么用？（差分详解+例题）\_网络\_From now on...的Blogs-CSDN博客

## [https://blog.csdn.net/qq\_44786250/article/details/100056975](https://blog.csdn.net/qq_44786250/article/details/100056975)

## 差分数组是个啥

差分数组很明显**就是个数组**呗，，，

本菜鸡学的比较浅，先说一下我自己认识的差分数组吧！

先解释一下什么是 差分：

差分其实就是数据之间的差，什么数据的差呢？就是**上面所给的原始数组的相邻元素之间的差值**，我们令 **d\[i\]=a\[i+1\]-a\[i\]**，一遍for循环即可将差分数组求出来。

下面给你一个栗子，给出一个差分数组先

![](https://img-blog.csdnimg.cn/20190825105026886.PNG)

## 差分数组怎么求

其实差分数组是一个**辅助数组**，从侧面来表示给定某一数组的变化，一般用来对数组进行**区间修改**的操作

还是上面那个表里的栗子，我们需要进行以下操作：

1、将区间【1，4】的数值全部加上3

2、将区间【3，5】的数值全部减去5

很简单对吧，你可以进行枚举。但是如果给你的数据量是1e5，操作量1e5，限时1000ms你暴力枚举能莽的过去吗？**T**到你怀疑人生直接。这时我们就需要使用到差分数组了。

其实**当你将原始数组中元素同时加上或者减掉某个数，那么他们的差分数组其实是不会变化的。**

利用这个思想，咱们将区间缩小，缩小的例子中的区间 【1,4】吧这是你会发现只有 d\[1\]和d\[5\]发生了变化，而d\[2\],d\[3\],d\[4\]却保持着原样，

![](https://img-blog.csdnimg.cn/20190825105034307.PNG)

在进行下一个操作，

![](https://img-blog.csdnimg.cn/2019082510504048.PNG)

> **这时我们就会发现这样一个规律，当对一个区间进行增减某个值的时候，他的差分数组对应的区间左端点的值会同步变化，而他的右端点的后一个值则会相反地变化，其实这个很好理解**

其实也就这么一点代码就ok了

```text
while(m--){//操作次数
	cin>>left>>right>>change;//左右端点及其变化的值
	d[left]+=change;
	d[right+1]-=change;
}
```

## 差分数组能干啥

既然我们要对区间进行修改，那么差分数组的作用一定就是求多次进行区间修改后的数组喽

**注意** 只能是区间元素同时增加或减少相同的数的情况才能用

因为我们的差分数组是由原始数组的相邻两项作差求出来的，即 d\[i\]=a\[i\]-a\[i-1\]；那么我们能不能反过来，求得一下修改过后的a\[i\]呢？

**直接反过来即得  a\[i\]=a\[i-1\]+d\[i\]** 

事实证明这是正确的，具体证法就不再推广，有空再补上吧；

更新数组a的方式则是下面的那一点点代码，这样我们就求出来了更新后的数组 a，是不是比线段树快多了呢？

```text
for(int i=1;i<=n;i++)
    a[i]=a[i-1]+b[i];
```

## 差分数组怎么用

翻来覆去还是那句，区间修改，当然了，有时候要结合树状数组来使用。直接看题目吧

### **HDU-1556 Color the Ball**  [_**http://acm.hdu.edu.cn/showproblem.php?pid=1556**_](http://acm.hdu.edu.cn/showproblem.php?pid=1556)

这个题果的不能再果了吧，看懂上面的，闭着眼也能敲出来

直接附上代码吧

```text
#include
#include
#include
#include
#include
#include
#define ll long long
#define mem(a,b) memset(a,b,sizeof(a))
using namespace std;
const int inf=0x3f3f3f3f;
const int mm=1e5+10;

int a[mm],b[mm];
int x,y;
int main()
{
    int n;
    while(scanf("%d",&n)&&n){
        mem(a,0);
        mem(b,0);
        for(int i=1;i<=n;i++){
            scanf("%d%d",&x,&y);
            b[x]++;
            b[y+1]--;
        }
        for(int i=1;i<=n;i++)
            a[i]=a[i-1]+b[i];
       
        for(int i=1;i
```

`再看下一个稍微进阶一点的题目`

### _**`POJ -3263 Tallest Cow`**_  [_**`http://poj.org/problem?id=3263`**_](http://poj.org/problem?id=3263)

`这道题他要你求出每头牛的最大可能的高度，我们的思路就是`**`先让所有的牛都和最高的一样高，关于他给到的牛能看到的区间进行修改，让中间的数至少减一，注意这里修改的一定是开区间，区间端点不能修改。再一个就是关于去重问题`**`。。。谁会想到他还有这么一个坑呢？？？？`

`只要看透了是个差分也很简单，代码附上`

```text
#include
#include
#include
#include
#include
#include
#define ll long long
#define mem(a,b) memset(a,b,sizeof(a))
using namespace std;
const int inf=0x3f3f3f3f;
const int mm=1e4+10;

int a[mm],b[mm];
int vis[mm][mm];
int main()
{
	int n,pos,h,r;
	scanf("%d%d%d%d",&n,&pos,&h,&r);
	for(int i=0;i<=h+1;i++)
		a[i]=h; 
	int x,y; 
	for(int i=1;i<=r;i++){
		scanf("%d%d",&x,&y);
		if(x>y) swap(x,y);// 
		if(vis[x][y])//判重 
			continue;
		vis[x][y]=1;
		b[x+1]--;//后面的减少 
		b[y]++;//前面的增加 
	}
	for(int i=1;i<=n;i++){
		a[i]=a[i-1]+b[i];
		printf("%d\n",a[i]);
	} 
	return 0;
}
```

今天测试碰到了一个二阶的差分题，我居然不会，用一阶差分直接被T了就很难受，听了旁边的大佬才知道，可以用两个差分来做，膜拜一下Orz

### CodeForces - 296C  Greg and Array? 

### [_https://cn.vjudge.net/problem/CodeForces-296C_](https://cn.vjudge.net/problem/CodeForces-296C)

首先说一下，这道题的题意吧，反正我一开始模拟了半年也没看懂，太难受了。

这道题意的坑在于那k个询问：

![](https://img-blog.csdnimg.cn/20190825192602713.PNG)

> **其实他给的k个询问并不是直接让你修改的区间，而是一个问题编号的区间，他给的\[L,R\]是告诉你要执行编号为 L~R 的操作**
>
> **这就告诉我们要把问题存起来先，也算是一个离线操作吧。**
>
> 这道题显然可以使用线段树，但是我觉得要打好多行代码，就选择了刚学的差分，我是用一阶差分做的，**直接两层大循环**，然后T死，这里有个代码，来纪念一下我自己被**T惨**的下场，不具备参考价值 嘻嘻嘻。[_**https://paste.ubuntu.com/p/zzSr63cM5J/**_](https://paste.ubuntu.com/p/zzSr63cM5J/)
>
> 这时我们就需要 用两个差分，**第一个差分来维护要进行的操作的区间端点，就是那K个询问**，这是有点难受的，反正我一上来没有想到，是旁边的大佬偷偷告诉我的，**第二个差分就是来维护我们的原始数组，进行修改**，修改的方法可以往上看↑↑↑↑↑↑
>
> 下面是可以参考的代码了：
>
> ```text
> #include
> #include
> #include
> #include
> #include
> #include
> #define ll long long
> #define mem(a,b) memset(a,b,sizeof(a))
> using namespace std;
> const int inf=0x3f3f3f3f;
> const ll mm=1e5+10;
>
> ll a[mm],d[mm];
> ll op[mm][5];
> ll n,m,k;
> ll cnt[mm];
> ll num[mm];
> int main()
> {
> 	cin>>n>>m>>k;
> 	for(ll i=1;i<=n;i++)
> 		cin>>a[i];
> 	for(ll i=1;i<=n+1;i++)
> 		d[i]=a[i]-a[i-1];	
>
> 	for(ll i=1;i<=m;i++)
> 		cin>>op[i][1]>>op[i][2]>>op[i][3];
> 		
> 	ll x,y;
> 	while(k--){//lixian
> 		cin>>x>>y;
> 		cnt[x]++;
> 		cnt[y+1]--;
> 	}
> 	
> 	for(int i=1;i<=m;i++)
> 		num[i]=num[i-1]+cnt[i];
> 	for(ll i=1;i<=m;i++){
> 		d[op[i][1]]+=op[i][3]*num[i];
> 		d[op[i][2]+1]-=op[i][3]*num[i];
> 	}
> 	
> 	for(ll i=1;i<=n;i++)
> 		a[i]=a[i-1]+d[i];
> 	for(ll i=1;i<=n;i++)
> 		cout<
> ```
>
> `还有一道高阶题目：`
>
> ## _**`POJ 3468-A Simple Problem with Integers`**_   [_**`http://poj.org/problem?id=3468`**_](http://poj.org/problem?id=3468)
>
> `这个题也果的一批，让你进行区间修改与区间求和查询，刚学线段树的时候用线段树写的，那是一个受罪啊，也就写了`**`两千多个字符`**`吧，是真的累。。。下面是一片关于这道题线段树解法的blog`
>
> ## [_**`https://blog.csdn.net/qq_44786250/article/details/98474701`**_](https://blog.csdn.net/qq_44786250/article/details/98474701)
>
> `这里我们讲一个高深一点的做法，那就是差分。但是有一点，差分对区间修改好用，但是区间求和还是需要暴力啊，依旧让你TTT这就是这道题的高深之处了，那就是结合树状数组，差分负责区间修改，树状数组进行区间求和，下面的代码也就`**`888个字符`**`，不过有点费脑，或许用的多了就好了吧！`
>
> ```text
> #include
> #include
> #include
> #include
> #include
> #include
> #define ll long long
> #define mem(a,b) memset(a,b,sizeof(a))
> using namespace std;
> const int inf=0x3f3f3f3f;
> const ll mm=1e5+10;
>
> ll n,m;
> ll a[mm],b[mm],c[mm];
> ll sum[mm];
>
> ll lbt(int x){
> 	return x&-x;
> }
>
> void update(int pos,int k){
> 	for(int i=pos;i<=n;i+=lbt(i)){
> 		b[i]+=k;
> 		c[i]+=k*(pos-1);
> 	}
> }
>
> ll getsum(int pos){
> 	ll res=0;
> 	for(int i=pos;i>0;i-=lbt(i)){
> 		res+=pos*b[i]-c[i];
> 	}
> 	return res;
> }
>
> int main()
> {
> 	scanf("%lld%lld",&n,&m);	
> 	for(ll i=1;i<=n;i++){
> 		scanf("%lld",&a[i]);
> 		sum[i]=sum[i-1]+a[i];
> 	}			
> 	char op[2];
> 	ll x,y,z;
> 	while(m--){
> 		scanf("%s",op);
> 		if(op[0]=='Q'){
> 			scanf("%lld%lld",&x,&y);
> 			printf("%lld\n",sum[y]-sum[x-1]+getsum(y)-getsum(x-1));
> 		}
> 		else {
> 			scanf("%lld%lld%lld",&x,&y,&z);
> 			update(x,z);
> 			update(y+1,-z);	
> 		}	
> 	}
> 	return 0;
> }
> ```

