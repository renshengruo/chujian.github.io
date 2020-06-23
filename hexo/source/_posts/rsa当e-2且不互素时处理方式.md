title: rsa当e=2且不互素时处理方式
author: 人生若只如初见
tags: []
categories:
  - 密码学
  - rsa
date: 2020-06-23 18:10:00
---
# 整个过程简化为6步
* 1、运用广义Euclid除法，求出整数s和t使得sp+tq=1
* 2、计算u≡c**((p+1)/4) (mod p)
* 3、计算v≡c**((q+1)/4) (mod q)
* 4、计算x≡(tqu+spv)  (mod n)
* 5、计算y≡(tqu-spv)  (mod n)
* 6、同余式x**2≡c (mod n) 的四个根是x,-x(mod n),y,-y(mod n)



*  其中一个根即为密文

```
# -*-coding: utf-8 -*-  
import gmpy  

def n2s(num):  
    t = hex(num)[2:]  
    if len(t) % 2 == 1:  
        return ('0'+t).decode('hex')  
    return t.decode('hex')  

c = int(open('flag.enc','rb').read().encode('hex'),16)  # 密文 c  
p = 275127860351348928173285174381581152299             # 分解后的素数 p  
q = 319576316814478949870590164193048041239             # 分解后的素数 q  
n = p*q                                                 # 公钥 N  

# 根据中国剩余定理求解相应明文  
r = pow(c,(p+1)/4,p)  
s = pow(c,(q+1)/4,q)  
a = gmpy.invert(p,q)  
b = gmpy.invert(q,p)  
x =(a*p*s+b*q*r)%n  
y =(a*p*s-b*q*r)%n  

# 打印明文  
print n2s(x%n)  
print n2s((-x)%n)  
print n2s(y%n)  
print n2s((-y)%n)
```