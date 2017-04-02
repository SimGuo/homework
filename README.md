#操作系统作业7

---

郭思敏
*1400012963*

---
####作业要求

1、Tracy和Peter与金鱼的故事，理解题意并分析各种解法。

2、试用信号量及PV操作正确模拟两条双向道路的交叉路口的交通情况。需要满足下列条件：
 - 任何给定时刻只能有一辆车通过路口；
 - 当一辆车到达交叉路口并且另一条道路上没有车到来时，应该允许此车通过；
 - 当两个方向上都有车到达时，它们应该轮流通过，以防止在其中一个方向上的无限期延迟。

3、考虑下面两个函数，其中A和B是任意的计算过程。初始值：s1=s2=d=1；c1=c2=0；假设有无限多个进程正在调用函数f1()或者f2()中的某一个。
 - 计算A有多少个调用可以并发进行？此时s1、c1和d的值分别是多少？
 - 当A运行的时候，计算B有多少个调用可以并发进行？此时s2、c2和d的值分别是多少？
 - A或者B会饥饿吗？解释原因。


---

####一、Tracy和Peter与金鱼的故事

**题意：**
> Tracy和Peter需要每天喂金鱼，金鱼每天必须要喂一次，但不能喂一次向上，否则会撑死——即每天有且仅有一人喂金鱼。

**解法一：**
```
Peter：
if(noNote) {
	leave note
	if(noFeed){
		feed fish
	}
	remove note
}

Tracy：
if(noNote) {
	leave note
	if(noFeed){
		feed fish
	}
	remove note
}
```
解法一中Peter和Tracy都是先判断金鱼是否被喂过（`if(noNote)`），然后再通知对方自己在喂金鱼了。
但是可能会出现如下情况：
 - 	在金鱼还没有喂的时候，Peter判断了`noNote`为真，在刚判断完还没发消息之前，Tracy也发现`noNote`为真。

在这种情况下，两个人都会去喂金鱼，金鱼就会撑死。


**解法二：**
```
Peter:
leave notePeter
if(no noteTracy) {
	if(noFeed){
		feed fish
	}
}
remove notePeter

Tracy:
leave noteTracy
if(no notePeter){
	if(noFeed){
		feed fish
	}
}
remove noteTracy
```
在解法二中，Peter和Tracy先互相通知对方自己要喂鱼，然后再看对方是否要喂鱼。如果对方想喂，就自己偷偷懒甩锅给别人喂。
因此，自然而然地，会有如下的问题：
 - 当其中一人（如Peter），Peter通知了Tracy自己要喂鱼(`leave notePeter`)，而在他看Tracy是否也要喂鱼之前，Tracy也客气地先通知Peter自己要喂鱼(`leave noteTracy`)。

在这种情况下，两个人在`if`的判断中，都认为对方会去喂鱼，从而自己就不喂鱼，导致最后鱼会饿死。

**解法三：**
```
Peter:
leave notePeter
while (noteTracy){
	do nothing
}
if(noFeed){
	feed fish
}
remove notePeter

Tracy:
leave noteTracy
if(no notePeter){
	if(noFeed){
		feed fish
	}
}
remove noteTracy
```
解法三中，Peter和Tracy也是先告诉对方自己要喂鱼了(`leave notePeter(Tracy)`)。但这次Peter比Tracy更加细心一些。即：如果Tracy发现Peter也说自己要喂了，那就直接甩锅给Peter，并宣称自己不喂鱼了。但如果Peter发现Tracy说自己要喂鱼，会耐心地等一下Tracy，直到Tracy说自己不喂鱼了(`remove noteTracy`)，也会认真判断一下是否还需要喂鱼。
这个方案是可以保证鱼的合理喂养的，但存在的一个问题是不公平。因为在Peter和Tracy的运行时间相差无几的情况下，大部分情况都会是Peter去喂鱼。

-----
####二、信号量及PV模拟双向道路

**题意：**
>试用信号量及PV操作正确模拟两条双向道路的交叉路口的交通情况。需要满足下列条件：
> - 任何给定时刻只能有一辆车通过路口；
> - 当一辆车到达交叉路口并且另一条道路上没有车到来时，应该允许此车通过；
> - 当两个方向上都有车到达时，它们应该轮流通过，以防止在其中一个方向上的无限期延迟。

**解答：**

```
Semaphore ew = 1, sn = 0; //表示道路的行驶权，默认是东西方向先走
Semaphore ew_cnt = 0, sn_cnt = 0; //计数
Semaphore road = 1; //道路锁

void Road_ew{
	过来一辆车
	V(ew_cnt); //计数

	//判断另外一个方向是否来了车（肯定有一个方向先进行V操作，所以只有如果在这之后只会有一个方向可能进入if条件）
	if(!sn_cnt){
		P(road);
		车开过去
		P(ew_cnt);
		V(road);
	}
	else{
		P(ew); 
		P(road);
		车开过去
		P(ew_cnt);
		V(road);
		V(sn);
	}
}

void Road_sn{
	过来一辆车
	V(sn_cnt);
	if(!ew_cnt){
		P(road);
		车开过去
		P(sn_cnt);
		V(road);
	}
	else{
		P(sn);
		P(road);
		车开过去
		P(sn_cnt);
		V(road);
		V(ew);
	}
}
```
在上面这段代码中：
 - 只有一个方向有车的话，就是同一方向的车之间争抢方向上的道路锁的问题。
 - 如果出现两个方向都有车的话，因为预设的`ew = 1, sn = 0`，所以运行的结果将会是东西方向的车先行，只要东西方向的车发现南北方向也有车（只要南北方向的车进行了`V(sn)`操作），就会将行驶权交给南北方向，然后南北方向的车获得行驶权之后就可以开车了。
 - 因为行使权同时只有一个为1，所以两个方向可以轮流走。

----
####三、

**题意：**
>考虑下面两个函数，其中A和B是任意的计算过程。初始值：s1=s2=d=1；c1=c2=0；假设有无限多个进程正在调用函数f1()或者f2()中的某一个。
> - 计算A有多少个调用可以并发进行？此时s1、c1和d的值分别是多少？
> - 当A运行的时候，计算B有多少个调用可以并发进行？此时s2、c2和d的值分别是多少？
> - A或者B会饥饿吗？解释原因。
```
f1(){
	P(s1);
	c1 = c1 + 1;
	if(c1 == 1) P(d);
	V(s1);

	A;
	
	P(s1);
	c1 = c1 - 1;
	if(c1 == 0)
	V(d);
	V(s1);
}

f2(){
	P(s2);
	c2 = c2 + 1;
	if(c2 == 1) P(d);
	V(s2);
	
	B;
	P(s2);
	c2 = c2 - 1;
	if(c2 == 0)
	V(d);
	V(s2);
}
```

可以知道：

（1）计算A可以有无数个调用并发进行。只要第一个调用抢占了d之后一直睡眠，则其他所有计算A的线程都可以进行计算。
此时s1在执行计算的过程中为1，d因为被抢占了所以为0，而c1则可能是任意不低于1的整数，表示正在执行计算A的任务个数。

（2）当A在运行的时候，计算B不能背调用。此时c2和d都是0。如果有任务在等待执行计算B的话，则s2为0，如果没有任务在等待，则s2为1。

（3）会出现饥饿的问题，因为只要一方抢占了d之后，这一方就可以无限制地进行计算，而另外一方就必须一直等待。

