title: RSA  Factor Attack 及常用脚本
author: 人生若只如初见
tags: []
categories:
  - 密码学
  - rsa
date: 2020-05-28 22:31:00
---
# 基本解密

* 以下全文中^代表乘方，例如：2^3=2的3次方=8
* c^d %n  == (m^e)^d %n ==m^ (e*d)%n ==m^(e*d %(φ（n）))%n  ==m^1 %n 

* 下面开始正文

# Factor Attack

## 1、当p==q

```
	n=p*q=p**2
	φ（n）=p**2-p
```

## 2、孪生质数（twin prime）

* 定义：孪生素数就是指相差2的素数对，例如：3和5，5和7，以36N（N+1）为界，孪生素数以波浪形式渐渐增多

```
	n1=p*q,n2=(p+2)*(q+2)
	φ（n1）=（p-1）*(q-1)=n1-(p+q)+1
	φ（n2）=(p+1)*(q+1)=n1+(p+q)+1
	n2=(p+2)*(q+2)=n1+2*(p+q)+4
	p+q=(n2-n1-4)/2
```

## 3、Common Factor Attack(公约数分解)

* p,q  为gcd(ni,nj)=pi=pj，所给N中有最大公因数，即可侧面分解n，也出现在唯密文攻击中，这时一般公约数出现在n与密文c之间

## 4、Pollard Algorithm

* 使用时机：当p-1光滑时
* 例如：

```
p=9132400715036908229752508016230000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001
```
* a,b,n,k属于N，p是素数，满足gcd(a,p)=1和p|n
* 根据费马小定理：

```
a^(p-1)≡1 (mod p)⇨a^k(p-1)≡1 (mod p)⇨p|gcd(a^b-1,n)=p^2
```

* 若p-1|b成立且满足gcd(a^b-1,n)>1,则gcd(a^b-1,n)=p,代入a=2去爆破，但可能会失败。

```
def pollard(n):
		a,b=2,2
		while True:
			a=pow(a,b,n)
			p=sympy.gcd(a-1,n)
			if 1<p<n: return p
			b +=1
```

## 5、Fermat's Factorization method
* 使用时机：|p-q|很小的时候

```
让   a=(p+q)/2  b=(p-q)/2
	 n=(a+b)^a-b)=a^2 -b^2
```
* 因为|p-q|很小，所以n会略等于a的平方，把a 用sqrt(n)代入，测试a^2-n是否为平方数，如果a^2-n是平方数，则(p,q)为(a+b,a-b)

```
def fermat(n):
		a=gmpy2.isqrt(n)+1
		b=a**2-n
		while not gmpy2.iroot(b,2)[1]:
			a += 1
			b = a**2 - n
		b= gmpy2.iroot(b , 2)[0]
		return ( a + b , a - b )
```

# 用到的数学定理

## 1、费马小定理
* a属于N，p是素数

```
a^(p-1)%p=1
a^(p-1)≡1 （mod p）
代入φ函数
推出：a^φ（p）≡1（mod p）
```

## Hastad's Broadcast Attack(中国剩余定理)

* 对加密的指数做攻击
* 使用时机：e固定不变，有多个n和对应的c

```
例子：
		N=3*5*7
	x≡2(mod 3)        N1=5*7=35   n1=3   d1=invert(N1,n1)=2
	x≡3(mod 5)        N2=3*7=21   n2=5   d2=invert(N2,n2)=1
	x≡2(mod 7)        N3=3*5=15   n3=7   d3=invert(N3,n3)=1
	x=(c1*d1*N1+c2*d2*N2+c3*d3*N3)%N
	之后对x使用gmpy2.iroot（）,开e次方
```
* 特例：2020网鼎杯
* 使用中国剩余定理求出X后，gmpy2.iroot()开次方数为17，这个需要观察rsa中密文长度、明文长度和模数长度之间的关系考虑。

## Wiener's Attack(维纳攻击)

* 对解密指数做攻击
* 使用时机：e非常大，d很小的时候
* 当d<(1/3)(N^(1/4))和|p-q|<max(p,q)条件符合时，可以利用(e,n)来估计(d,φ(n))

```
e*d≡1(mod φ（n）)
	→e*d=k*φ（n）+1                       k属于N
	→e/φ（n）=k/d++1/(d*φ（n）)        divide by d*φ（n）
	→e/φ（n）≈k/d
	→e/n≈k/d
	方法：连分数  #如何做连分数
```

## Common Factor Attack(共模攻击)

* 使用时机：相同明文，不同e，相同的N, 有对应的密文

```
m**e1%n=c1
m**e2%n=c2
若满足gcd(e1,e2)=1，则有线性方程满足s1*e1+s2*e2=1，其中s1=invert(e1,e2)且s2=invert(e2,e1)
c1**s1 * c2**s2→(m**(e1*s1))*(m**(e2*s2))%n→m**(e1*s1+e2*s2)%n
注意到(s1,s2)，他们必须是线性方程的一组解，所以分开算invert并不是一组的，所以算出s1后，因为s1*e1+s2*e2=1,所以s2=(1-s1*e1)/e2
但python的pow报错：
		ValueError:pow()2nd argument cannot be negative when 3rd argument specified
		若c2**s2≡x**(-s2)  (mod n)
		  (c2**s2)(x**s2)≡1 (mod n)
		  (c2*x)**s2≡1 (mod n)
```

* 脚本

```
import sympy
n=
e1=
e2=
c1=
c2=
c2=int(sympy.invert(c2,n))  #作为上面公式中的x
s1=int(sympy.invert(e1,e2))
s2=(s1*e1-1)//e2
m=(pow(c1,s1,n)*pow(c2,s2,n))%n
```

# 一些自己常用的自定义函数和脚本
## 1、自定义的invert函数

```
def egcd(a,b):
	if a==0: return (b,0,1)
	else:
		g,x,y=egcd(b%a,a)
		return (g, x - (b // a)*y,y)

def invert(a,m):
	g,x,y=egcd(a,m)
	if g !=1: print ('modular inverse does not exist')
	else: return x%m

e= 
phi=   #即φ（n）
d=invert(e,phi)
```

## 2、维纳攻击脚本

```
import sympy

def fractions(x,y):
	ans=[y//x]
	if y%x==0: return ans
	else:
		ans.extend(fractions(y%x,x))
		return ans

def continued_fractions(e,n):
	ans=[]
	x= fractions(e,n)
	for i in range(1,len(x)):
		k, d= 1, x[i-1]
		for j in x[:i-1][::-1]:
			k, d = d, d*j+k
		ans.apped((k, d))
	return ans
	
def Wiener(e,n):
	for k, d in continued_fractions(e,n):
		phi=(e*d-1)//k
		#x**2 -(n-phi+1)x+n=0
		if d == int(sympy.invert(e,phi)):
			return d
			break
```
