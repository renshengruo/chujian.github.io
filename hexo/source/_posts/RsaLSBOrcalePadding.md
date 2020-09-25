title: RsaLsbOrcalePadding
author: 人生若只如初见
tags: []
categories:
  - 密码学
  - rsa
date: 2020-09-06 09:23:00
---
[参考1](https://crypto.stackexchange.com/questions/11053/rsa-least-significant-bit-oracle-attack)
[参考2](https://zhuanlan.zhihu.com/p/140726869)
[参考3](https://introspelliam.github.io/2018/03/27/crypto/RSA-Least-Significant-Bit-Oracle-Attack/)

## 原理公式

```
攻击者得到密文C=Pe(mod n) ，将其乘以2e(mod N), 并作为密文发送出去，若返回f(2P)
如果f(2P) 返回的最后一位是0，那么2P<N，即P<N/2
如果f(2P) 返回的最后一位是1，那么2P>N，即 P>N/2
接着我们来看看2P 和 4P
如果返回的是（偶，偶），那么有 P<N/4
如果返回的是（偶，奇），那么有N/4<P<N/2
如果返回的是（偶，奇），那么有N/2<P<3N/4
如果返回的是（奇，奇），那么有3N/4<P<N

```
* 数论中有个定理，c = 偶数 a mod 奇数 b，若 c 为奇数，则 a>b，若 c 为偶数，则 a<b

## 推导过程


![](/images/pasted-33.png)



## 脚本：

```
L = 0
H = n
t = pow(2, e, n)
for _ in range(n.bit_length()):
    c = (t * c) % n
    if oracle(c) == 0:
        H = (L + H) // 2
    else:
        L = (L + H) // 2
m = L # plain text
```


* 可忽略
## service.py

```
#!/usr/bin/python -u
from Crypto.Util.number import *
from Crypto.PublicKey import RSA
import random
#from SECRET import flag
flag = "CTF{this_is_my_test_flag}"
m = bytes_to_long(flag)
key = RSA.generate(1024)
c = pow(m, key.e, key.n)
print("Welcome to BACKDOORCTF17\n")
print("PublicKey:\n")
print("N = " + str(key.n) + "\n")
print("e = " + str(key.e) + "\n")
print("c = " + str(c) + "\n")
while True:
    try:
        temp_c = int(raw_input("temp_c = "))
        temp_m = pow(temp_c, key.d, key.n)
    except:
        break
    l = str(((temp_m&5) * random.randint(1,10000))%(2*(random.randint(1,10000))))
    print "l = "+l
```

## solve.py

```
# -*- coding: utf-8 -*-
#/usr/bin/env python
from pwn import *
import libnum
import Crypto
import re
from binascii import hexlify,unhexlify
if len(sys.argv)>1:
    p=remote("127.0.0.1",2334)
else:
    p=remote('127.0.0.1',2333)
#context.log_level = 'debug'
def oracle(c):
    l = []
    for i in range(20):
        p.sendline(str(c))
        s = p.recvuntil("temp_c")
        l.append(int(re.findall("l\s*=\s*([0-9]*)",s)[0]))
    flag0 = 0
    flag2 = 0
    for i in range(20):
        if l[i]%2 != 0:
            flag0 = 1
        if l[i] > 10000:
            flag2 = 1
    return [flag2,flag0]
def main():
    ss = p.recvuntil("temp_c")
    N = int(re.findall("N\s*=\s*(\d+)",ss)[0])
    e = int(re.findall("e\s*=\s*(\d+)",ss)[0])
    c = int(re.findall("c\s*=\s*(\d+)",ss)[0])
    size = libnum.len_in_bits(N)
    print "N=",N
    print "e=",e
    print "c=",c
    c = (pow(2,e,N)*c)%N
    LB = 0
    UB = N
    i = 1
    while LB!=UB:
        flag = oracle(c)
        print i,flag
        if flag[1]%2==0:
            UB = (LB+UB)/2
        else:
            LB = (LB+UB)/2
        c = (pow(2,e,N)*c)%N
        i += 1
    print LB
    print UB
    for i in range(-128,128,0):
        LB += i
        if pow(LB,e,N)==C:
            print unhexlify(hex(LB)[2:-1])
            exit(0)
if __name__ == '__main__':
    main()
    p.interactive()
```