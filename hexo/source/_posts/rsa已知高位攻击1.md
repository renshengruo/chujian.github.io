title: rsa已知高位攻击
author: 人生若只如初见
tags: []
categories:
  - 密码学
  - rsa
date: 2020-05-27 11:54:00
---
# 已知高位攻击利用
* 基础讲解：
# 1. 格的基础
* 格的基础概念：设v1,v2,….,vm线性无关，m维格L（v1,v2,….,vm）是指由向量v1,v2,….,vm生成的一个向量集，它的形式为——L（v1,v2,….,vm）=Σ ai * vi, (i=1,2,….,m ai∈Z)
称{v1,v2,….,vm}为格L的一组基，且记Dim(L)=m; m、n分别为格L的维数和秩，当m=n时，称格L是满维的。
# 2. 格基规约
* 同一个格可以使用不同的基来表示，解决格上的问题时，即使使用同一种算法，如果选用不同的基，最后的运行时间和运行结果差别也是十分明显的。选择最优的一组基的过程就是格基规约，并称这样的一组基为格的一组规约基。常见的两种规约方法是Gauss规约和LLL规约。
![](https://note.youdao.com/yws/api/personal/file/3E79235F98824E54871C18C516CCC71D?method=download&shareKey=719882fdcdade620567084447f055b72)
* 其实综上所述，已知高位攻击就是将问题转换到格的数学问题中，转向求格的最短非零向量

* 格基规约算法应用于密码分析的另一个重要方面就是线性同余结尾序列的重构。

# 重点（伪随机序列的还原重构）
## LLL-算法在密码分析中的应用(攻克线性同余结尾序列)
* 众所周知, 线性同余是产生伪随机序列的一个极其简便流行的手段.但是, 若将原序列原样输出,而不做
* 处理的话 ,即使是在不知道模数与乘子的情况下, 序列也能被轻易地重构(即还原出模数与乘子).于是, 为了增强抗攻击力,Knuth 提出了截尾序列生成器
* 该生成器模型如下 :
```
输入初值 x 0 ,递归生成序列{x i }:x i+1 =ax i +b mod m , a , b , m ∈ Z ,输出序列{y i }:y i =[ x i /2 βv ] 
```
* 其中v 为m 的比特数 , β =1 -α而α为输出y i 的比特数占原x i 的比特数之比例.
* 由于对{x i }做了截尾处理, 使重构困难了.早期的计算机上曾广泛使用了类似的方法产生伪随机整数序列 .其方法是将 n 比特数平方后, 丢弃头尾各n/2 比特

# J.S算法
* 功能 　设得到 n +t 长的序列 ,记为 y 1 , y 2 , …, y t , y t+1 , …, y t+n ,其中 n ≈(2 αt log m)1/2,且原序列{xi}中(x 1 -x 0 , m)=1.J.S.算法能求出模数 m 和乘子a , 从而重构了原模型

* 描述：
```
<1> 做Vi=[y(i+1) - yi ,  y(i+2)-y(i+1)....,y(i+t)-y(i+t)-1],  i=1,2,...,n.
<2> 适当选取 k , 考虑 kV 1 , kV 2 , …, kV n 作输入 , 执行 KILL- 算法 , 得到 λ=(λ 1 , λ 2 , …, λ n ), 且∑λ i V i =0   (i=1，2,...,n), λ 较小.
<3> 做多项式 P(x)=Σλi * X^i  (i=1,2,...,n)
<4> 重复(1)～ (3)多次 ,得到多项式序列 p 1 , p 2 , …, p r .将p i 自然映射到Z^(n+1) 上.应用LLL-算法于p 1 , p 2 ,…, p r 上, 得到生成的格 L 的一组 LLL-约基 ,计算 d(L),令m =d(L),输出 m 
<5> 将 L 的基中次数 ≥2的向量乘以大整数 K (K ≥m 2^(n/2) ),再构成子格 L′, 执行 LLL-算法 ,找到 L′的一组 LLL-约基 .运用线性代数的知识,利用 LLL-约基, 找到首一多项式 A(x).令a =A(0), 输出 a .算法结束 .
```
* 注意：在（4）中多项式个数r≤（6α-1）㏒ m ㏒ ㏒ m,就可以满足要求了，算法中执行的LLL-算法其实是LLL-算法在输入任意r个向量(有可能线性相关)情况时的改进算法

# coppersmith的一些定理:
```
<1>定理3.3 对任意的a > 0 ， 给定N = PQR及PQ的高位(1/5)(logN,2)比特，我们可以在多项式时间logN内得到N的分解式。这是三个因式的分解。也就是说我们现在是由理论依据的，已知高位是可以在一定时间内分解N。
<2>已知p高位u多少位菜可以进行攻击呢？定理是在《Mathematics_of_Public_Key_Cryptography》这本数里面提到的，我们将我们上面得到的N的值带入上图的式子中。计算(1/根号2)*N
```
* 由上式得出：

```
if p.bit_length == 1024 ,p的高位需已知约576位

if p.bit_length == 1024 ,p的高位需已知约288位
```
* 1.sage里面的small_roots能实现上述的给出已知的p高位进行分解N的函数方法，利用了LLL算法求解非线性低维度多项式方程小根的方法。

* 2.Coppersmith证明了在已知p和q部分比特的情况下，若q和p的未知部分的上界X和Y满足XY <= N ^ (0.5)则Ｎ的多项式可以被分解。
这里的0.5可以替换成其他的数，具体原因不详。

* 链接：
  [sage里small_roots的具体用法](https://doc.sagemath.org/html/en/reference/polynomial_rings/sage/rings/polynomial/polynomial_modn_dense_ntl.html#sage.rings.polynomial.polynomial_modn_dense_ntl.small_roots)

# 已知高位攻击分为三种情况
```
<1>已知P的高位
<2>已知d的高位
<3>已知m的高位
```
## 已知P的高位

* 知道p的高位为p的位数的约1/2时即可

* 已知e,n爆破 1024的P，至少需要知道前576位二进制，即前144位16进制(特殊情况下，可能所得到的已知位数稍小于144位，需要爆破两三位，然后使用sage脚本)
* 正好已知144位16进制的情况
```
n=
p4=            #已知P的高位
e=
pbits=          #P原本的位数

kbits=pbits - p4.nbits()
print p4.nbits()
p4 = p4 << kbits
PR.<x> = PolynomialRing(Zmod(n))
f = x + p4
roots = f.small_roots(X=2^kbits,beta=0.4)
# 经过以上一些函数处理后，n和p已经被转化为10进制
if roots:
    p= p4 + int(roots([0]))
    print "n",n
    print "p",p
    print "q",n/p
```
* 已知142位16进制的情况
```
n=
p4=            #已知P的高位,最后面8位二进制，也就是两位十六进制要参与爆破，所以要用00补充
e=
pbits=          #P原本的位数


for i in range(0,256):        # 要爆破的8位二进制数，为2**8==256，表示0~255
    p4 =
    p4 = p4 + int(hex(i),16)
#print hex(p4)


kbits=pbits - p4.nbits()
print p4.nbits()
p4 = p4 << kbits
PR.<x> = PolynomialRing(Zmod(n))
f = x + p4
roots = f.small_roots(X=2^kbits,beta=0.4)
# 经过以上一些函数处理后，n和p已经被转化为10进制
if roots:
    p= p4 + int(roots([0]))
    print "n",n
    print "p",p
    print "q",n/p
```
## 已知d的高位

* 如果知道d的低位，低位约为n的位数的1/4就可以恢复d。已知私钥的512位的低位 Partial Key Exposure Attack(部分私钥暴露攻击)
```
def partial_p(p0, kbits, n):
 
    PR.<x> = PolynomialRing(Zmod(n))
 
    nbits = n.nbits()
 
 
 
    f = 2^kbits*x + p0
 
    f = f.monic()
 
    roots = f.small_roots(X=2^(nbits//2-kbits), beta=0.3)  # find root < 2^(nbits//2-kbits) with factor >= n^0.3
 
    if roots:
 
        x0 = roots[0]
 
        p = gcd(2^kbits*x0 + p0, n)
 
        return ZZ(p)
 
 
 
def find_p(d0, kbits, e, n):
 
    X = var('X')
 
 
 
    for k in xrange(1, e+1):
 
        results = solve_mod([e*d0*X - k*X*(n-X+1) + k*n == X], 2^kbits)
 
        for x in results:
 
            p0 = ZZ(x[0])
 
            p = partial_p(p0, kbits, n)
 
            if p:
 
                return p
 
 
 
 
 
if __name__ == '__main__':
 
    n = 0xd463feb999c9292e25acd7f98d49a13413df2c4e74820136e739281bb394a73f2d1e6b53066932f50a73310360e5a5c622507d8662dadaef860b3266222129fd645eb74a0207af9bd79a9794f4bd21f32841ce9e1700b0b049cfadb760993fcfc7c65eca63904aa197df306cad8720b1b228484629cf967d808c13f6caef94a9
 
    e = 3
 
    d = 0x603d033f2ef6c759aec839f132a45215fc8a635b757f3951a731fe60bc6729b3bcf819b57abfcaba3a93e9edef766c0d499cad3f7adb306bcf1645cfb63400e3
 
 
 
    beta = 0.5
 
    epsilon = beta^2/7
 
 
 
    nbits = n.nbits()
 
    print "nbits:%d:"%(nbits)
 
    #kbits = floor(nbits*(beta^2+epsilon))
 
    kbits = nbits - d.nbits()-1
 
    print "kbits:%d"%(kbits)
 
    d0 = d & (2^kbits-1)
 
    print "lower %d bits (of %d bits) is given" % (kbits, nbits)
 
 
 
    p = find_p(d0, kbits, e, n)
 
    print "found p: %d" % p
 
    q = n//p
 
    print d
 
print inverse_mod(e, (p-1)*(q-1))
```
## 已知明文m的高位

* 已知明文的高位，是Stereotyped messages攻击 或 Lattice based attacks 
```
n = 0x2519834a6cc3bf25d078caefc5358e41c726a7a56270e425e21515d1b195b248b82f4189a0b621694586bb254e27010ee4376a849bb373e5e3f2eb622e3e7804d18ddb897463f3516b431e7fc65ec41c42edf736d5940c3139d1e374aed1fc3b70737125e1f540b541a9c671f4bf0ded798d727211116eb8b86cdd6a29aefcc7

e = 3
m = randrange(n)
c = pow(m, e, n)

beta = 1
 
epsilon = beta^2/7
 
nbits = n.nbits()
 
kbits = floor(nbits*(beta^2/e-epsilon))
 
#mbar = m & (2^nbits-2^kbits)
 
mbar = 0xb11ffc4ce423c77035280f1c575696327901daac8a83c057c453973ee5f4e508455648886441c0f3393fe4c922ef1c3a6249c12d21a000000000000000000
 
c = 0x1f6f6a8e61f7b5ad8bef738f4376a96724192d8da1e3689dec7ce5d1df615e0910803317f9bafb6671ffe722e0292ce76cca399f2af1952dd31a61b37019da9cf27f82c3ecd4befc03c557efe1a5a29f9bb73c0239f62ed951955718ac0eaa3f60a4c415ef064ea33bbd61abe127c6fc808c0edb034c52c45bd20a219317fb75
 
print "upper %d bits (of %d bits) is given" % (nbits-kbits, nbits)
 
PR.<x> = PolynomialRing(Zmod(n))
 
f = (mbar + x)^e - c

print m
x0 = f.small_roots(X=2^kbits, beta=1)[0]  # find root < 2^kbits with factor = n1
print mbar + x0
```