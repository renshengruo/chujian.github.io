title: Ecc椭圆曲线基础
author: 人生若只如初见
tags: []
categories:
  - 密码学
  - ECC
date: 2020-05-27 18:16:00
---
* 一般，椭圆曲线可以用以下二元三阶方程的形式来表示：

```
y² = x³ + ax + b，其中a、b为系数。
```

* 它大概的几何形状如下图：

![](/images/pasted-14.png)
* 而本文要介绍的加法和乘法，就是基于这样一个奇怪的几何图形来做到的。
* 椭圆曲线加法（非有限域）：在椭圆曲线上取一点P(Xp,Yp)，再取一点Q(Xq,Yq)，连接P、Q两点作一条直线，这条直线将在椭圆曲线上交于第三点G，过G点作垂直于X轴的直线，将过椭圆曲线另一点R（一般是关于X轴对称的点），R点则被定义为P+Q的结果，既P+Q=R：

![](/images/pasted-15.png)

* 当P=Q的情况下，直线将是椭圆曲线在P（Q）点上的切线，而G点是这条切线和曲线的另一个交点，同样，P+Q=R

![](/images/pasted-16.png)

* 通过上述的图片和文字描述，已经在几何图形上给出了椭圆曲线加法的定义，可是如果要公式化，该如何快速计算呢？
* 这里只提供快速计算公式，不提供证明，证明可以自己再去解方程组推导一下：
* 计算P+Q=R
* 当P！=Q时，两点纵坐标相减的值与横坐标相减的值就是直线的斜率：

```
λ = (Yq - Yp)/(Xq - Xp)

```
* 当P=Q，计算过P（Q）点切线的斜率，既椭圆曲线公式两边求导相除：

```
λ = (3Xp² + a)/2Yp
```
* 斜率计算之后，对点R的坐标进行计算，公式如下：

```
Xr = (λ² - Xp - Xq)
Yr = (λ(Xp - Xr) - Yp)
```
* 通过上述公式，可以快速计算椭圆曲线上任意两点的加法和，这里给出加法实现的python代码：

```
if P == Q:
aaa=(3pow(P[0],2) + a)
bbb=(2G[1])
k=(aaa/bbb)
else:
aaa=(Q[1]-P[1])
bbb=(Q[0]-P[0])
k=(aaa/bbb)

Rx=(pow(k,2)-P[0] - Q[0])
Ry=(k*(P[0]-Rx) - P[1])
椭圆曲线加法（有限域）
```

* 实数范围上光滑的椭圆曲线在密码学应用上并不合适，需要进行有限域下的离散化操作才能使用。

![](/images/pasted-18.png)
* 现在将上述的椭圆曲线加法计算公式适当修改，以适应有限域下的计算：


```
当P！=Q时，两点纵坐标相减的值与横坐标相减的值需要与p进行取余操作：
	λ = (Yq - Yp)/(Xq - Xp) mod p

当P=Q，计算过P（Q）点切线的斜率，既椭圆曲线公式两边求导相除，结果也需要与p进行取余操作：

	λ = (3Xp² + a)/2Yp mod p

斜率计算之后，对点R的坐标进行计算，公式如下：

	Xr = (λ² - Xp - Xq) mod p

	Yr = (λ(Xp - Xr) - Yp) mod p

通过比较，有限域下的计算只是对结果进行了取余操作，上述公式看起来已经解决了有限域下的椭圆曲线加法。

但是如果在编写代码，计算实际的例子时，有很大可能会得到错误的结果，其根源在于

	λ = (Yq - Yp)/(Xq - Xp) mod p或λ = (3Xp² + a)/2Yp mod p
    
在进行取余计算之前，除数和被除数之前可能并不是一个整除的关系。如：1/4 mod 23，如果直接进行处理，将会得到结果0。

但是在分数求模计算中，是如下定义的：

	计算a/b(mod n)

	a/b (mod n)=a*b^-1(mod n)

计算1/b mod n=b^(-1) mod n
就是求y，满足：

	yb = 1 mod n
	y是有限域F(n)上x的乘法逆元素

简单点说，假设需要求上述的1/4 mod 23，可以转化为14（-1次方） mod 23，又可以转化为1(4和23的乘法逆元) mod 23。
而计算乘法逆元，可以通过拓展欧几里得计算得到，这里对拓展欧几里得不作展开，只提供一个简单算法流程描述：
		ExtendedEuclid(d,f) 
	1 （X1,X2,X3):=(1,0,f) 
	2   (Y1,Y2,Y3):=(0,1,d) 
	3  if (Y3=0) then return  d'=null//无逆元 
	4  if (Y3=1) then return  d'=Y2  //Y2为逆元 
	5  Q:=X3 div Y3 
	6  (T1,T2,T3):=(X1-Q*Y1,X2-Q*Y2,X3-Q*Y3) 
	7 （X1,X2,X3):=(Y1,Y2,Y3) 
	8  (Y1,Y2,Y3):=(T1,T2,T3) 
	9  goto 3
```

* 得到乘法逆元后，椭圆曲线上的加法运算计算就简单了，实现Python代码如下:

```
#coding:utf-8
#欧几里得算法求最大公约数
def get_gcd(a, b):
    k = a // b
    remainder = a % b
    while remainder != 0:
        a = b 
        b = remainder
        k = a // b
        remainder = a % b
    return b
    
#改进欧几里得算法求线性方程的x与y
def get_(a, b):
    if b == 0:
        return 1, 0
    else:
        k = a // b
        remainder = a % b       
        x1, y1 = get_(b, remainder)
        x, y = y1, x1 - k * y1          
    return x, y

#返回乘法逆元
def yunsle(a,b):
    #将初始b的绝对值进行保存
    if b < 0:
        m = abs(b)
    else:
    m = b
    flag = get_gcd(a, b)

    #判断最大公约数是否为1，若不是则没有逆元
    if flag == 1:   
    x, y = get_(a, b)   
    x0 = x % m #对于Python '%'就是求模运算，因此不需要'+m'
    #print(x0) #x0就是所求的逆元
        return x0
    else:
    print("Do not have!")


if P == Q:
        aaa=(3*pow(P[0],2) + a)
        bbb=(2*P[1])
        if aaa % bbb !=0:
            val=yunsle(bbb,mod)
            y=(aaa*val) % mod
        else:
            y=(aaa/bbb) % mod 
else:
        aaa=(Q[1]-P[1])
        bbb=(Q[0]-P[0])
        if aaa % bbb !=0:
            val=yunsle(bbb,mod)
            y=(aaa*val) % mod
        else:
            y=(aaa/bbb) % mod 

Rx=(pow(k,2)-P[0] - Q[0])  % mod
Ry=(k*(P[0]-Rx) - P[1])  % mod
```

## 椭圆曲线乘法

* 简单介绍完椭圆曲线上定义的加法运算，椭圆曲线上的乘法运算就比较简单了，因为加法可以退化为加法运算，就像算数上的1*3等价与1+1+1。

* 假设我们需要求2P，则可以化简为P+P=2P

* 同理，当我们需要求3P时，可以化简为P+2P=3P，其中2P=P+P

* 最后，我们可以得到规律，当求nP时（n为任意正整数），P+(n-1)P=nP，其中(n-1)P=P+(n-2)P

* 这样，通过上述介绍的椭圆曲线加法公式，完全可以进行椭圆曲线的乘法计算
* 以本文开头的题目为例，给出Python代码实现：

```
#coding:utf-8
#欧几里得算法求最大公约数
def get_gcd(a, b):
    k = a // b
    remainder = a % b
    while remainder != 0:
        a = b 
        b = remainder
        k = a // b
        remainder = a % b
    return b
    
#改进欧几里得算法求线性方程的x与y
def get_(a, b):
    if b == 0:
        return 1, 0
    else:
        k = a // b
        remainder = a % b       
        x1, y1 = get_(b, remainder)
        x, y = y1, x1 - k * y1          
    return x, y

#返回乘法逆元
def yunsle(a,b):
    #将初始b的绝对值进行保存
    if b < 0:
        m = abs(b)
    else:
    m = b
    flag = get_gcd(a, b)

    #判断最大公约数是否为1，若不是则没有逆元
    if flag == 1:   
    x, y = get_(a, b)   
    x0 = x % m #对于Python '%'就是求模运算，因此不需要'+m'
    #print(x0) #x0就是所求的逆元
        return x0
    else:
    print("Do not have!")


mod=15424654874903
#mod=23
a=16546484
#a=1
b=4548674875
#b=1
G=[6478678675,5636379357093]
#G=[3,10]
#次数
k=546768
temp=[6478678675,5636379357093]
#temp=[3,10]
for i in range(0,k):
    if i == 0:
        aaa=(3*pow(G[0],2) + a)
        bbb=(2*G[1])
        if aaa % bbb !=0:
            val=yunsle(bbb,mod)
            y=(aaa*val) % mod
        else:
            y=(aaa/bbb) % mod
    else:
        aaa=(temp[1]-G[1])
        bbb=(temp[0]-G[0])
        if aaa % bbb !=0:
            val=yunsle(bbb,mod)
            y=(aaa*val) % mod
        else:
            y=(aaa/bbb) % mod

    #print y
    Rx=(pow(y,2)-G[0] - temp[0]) % mod
    Ry=(y*(G[0]-Rx) - G[1]) % mod
    temp=[Rx,Ry]
    #print temp

print temp
```
* 参考文献：

[讲解了受限域的曲线下的加法实现计算](http://blog.51cto.com/11821908/2057726)


[只讲解了无受限域下曲线的加法](https://www.jianshu.com/p/2e6031ac3d50)


[分数求模原理介绍](https://wenku.baidu.com/view/6f2879cca1c7aa00b52acb5f.html)


[看雪论坛上的详细介绍，提供了加法运算的验证集](https://www.pediy.com/kssd/pediy06/pediy6014.htm)

[乘法逆元求解的python实现](https://blog.csdn.net/baidu_38271024/article/details/78881031)
