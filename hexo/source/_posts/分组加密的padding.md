title: 分组加密的padding
author: 人生若只如初见
tags:
  - ''
categories:
  - 密码学
  - 填充方式
date: 2020-09-24 19:55:00
---
# 基本规则
* 在分组加密中，明文长度不满足要求时，经常进行padding，即使恰好为块长度的整数倍，依旧需要padding（填充）。

# 各种填充规则
## pkcs5

```
DES INPUT BLOCK  = f  o  r  _  _  _  _  _
(IN HEX)           66 6F 72 05 05 05 05 05
KEY              = 01 23 45 67 89 AB CD EF
DES OUTPUT BLOCK = FD 29 85 C9 E8 DF 41 40
```

* pkCS5是8字节填充的，即填充一定数量的内容，使得成为8的整数倍，而填充的内容取决于需要填充的数目。例如，串0x56在经过PKCS5填充之后会成为0x56 0x07 0x07 0x07 0x07 0x07 0x07 0x07因为需要填充7字节，因此填充的内容就是7。当然特殊情况下，如果已经满足了8的整倍数，按照PKCS5的规则，仍然需要在尾部填充8个字节，并且内容是0x08,目的是为了加解密时统一处理填充。
* pkcs5总结来说就是在需要填充的位置，8字节缺少多少就填充多少，例如上面缺少5字节，所以填充0x05。

## pkcs7




## OneAndZeroes Padding

```
DES INPUT BLOCK  = f  o  r  _  _  _  _  _
(IN HEX)           66 6F 72 80 00 00 00 00
KEY              = 01 23 45 67 89 AB CD EF
DES OUTPUT BLOCK = BE 62 5D 9F F3 C6 C8 40
```

* 这里其实就是和 md5 和 sha1 的 padding 差不多。


## Pad with zeroes except make the last byte equal to the number of padding bytes

```
DES INPUT BLOCK  = f  o  r  _  _  _  _  _
(IN HEX)           66 6f 72 00 00 00 00 05
KEY              = 01 23 45 67 89 AB CD EF
DES OUTPUT BLOCK = 91 19 2C 64 B5 5C 5D B8
```

* 按照上面的例子，依旧缺失5字节，中间全部填充0x00，在最后一字节填充填充了多少，上面一共需要填充5字节，所以结尾为0x05。

## Pad with zero (null) characters（零填充）

```
DES INPUT BLOCK  = f  o  r  _  _  _  _  _
(IN HEX)           66 6f 72 00 00 00 00 00
KEY              = 01 23 45 67 89 AB CD EF
DES OUTPUT BLOCK = 9E 14 FB 96 C5 FE EB 75
```

* 在所有需要填充的地方填充0x00，直到满足填充原则。

## Pad with spaces（空格填充）

```
DES INPUT BLOCK  = f  o  r  _  _  _  _  _
(IN HEX)           66 6f 72 20 20 20 20 20
KEY              = 01 23 45 67 89 AB CD EF
DES OUTPUT BLOCK = E3 FF EC E5 21 1F 35 25
```

* 所有需要填充的地方都使用空格填充。

