title: 密码学编码(二)base编码
author: 人生若只如初见
tags: []
categories:
  - 密码学
  - 编码
date: 2020-05-27 15:23:00
---
* 我们接着上一节。
* 上一节我们讲述了ASCII编码，接下来的base系列编码与ASCII之间联系非常紧密。好，我们继续讲解。

* base系列编码：
* ASCII 是用128（2^8）个字符，对二进制数据进行编码的方式，
* base64编码是用64（2^6）个字符，对二进制数据进行编码的方式 
* base32就是用32（2^5）个字符，对二进制数据进行编码的方式
* base16就是用16（2^4）个字符，对二进制数据进行编码的方式

* 这里我们可以清楚的看到，base系列编码之间的不同，在于用于编码的字符数量的多少。

* 那我们如何直接区分出base16、32、64编码呢？
* 那可以从编码字符的数量方面入手，对于base16，用于编码的字符只有：1-9，A-F ,只有简单的15个字符。对于base32而言，编码字符有了明显改变，由base16的类型转变为了A-Z,2-7。作为base系列中最完善的base64编码，是在base32的基础上，增加了”a-z,0,1,8,9,+,/“，以及特殊填充字符”=”
* Base-64编码将一个8位子节序列拆散为6位的片段，并为每个6位的片短分配一个字符，这个字符是Base-64字母表中的64个字符之一。

* 编码解码过程：
* base系列编码过程都类似，所以我们用base64来说明
# base64填充
* base64编码收到一个8位字节序列，将这个二进制序列流划分成6位的块。二进制序列有时不能正好平均地分为6位的块，在这种情况下，就在序列末尾填充零位，使二进制序列的长度成为24的倍数(6和8的最小公倍数)。
* 对已填充的二进制进行编码时，任何完全填充(不包括原始数组中的位)的6位组都有特殊的第65个符号”=”表示。如果6位组是部分填充的，就将填充位设置为0.
* 下面会写一个填充实例。初始输入字符串为”a:a”为3个字节(24位)。24是6和8的倍数，因此按照上面给出的例子计算。无需填充就会得到base64编码为”YTph”。
* 然而，再增加一个字符，输入字符串变为”a:aa”,转换为二进制就会有32位长。而6和8的下一个公倍数为48.因此要添加16为的填充码。填充的前4位是与数据位混合在一起的。得到的6位组01xxxx，会被当作010000、十进制中的16，或者base64编码的Q来处理。剩下的两个6位组都是填充码，用=来表示。

```
a:a -- 011000 010011 101001 100001 -- YTph

a:aa -- 011000 010011 101001 100001 011000 01xxxx xxxxxx xxxxxx -- YTphYQ==

a:aaa -- 011000 010011 101001 100001 011000 010110 0001xx xxxxxx -- YTphYWE=

a:aaaa -- 011000 010011 101001 100001 011000 010110 000101 1000001 -- YTphYWFh
```
* 特别注意，Base64编码后的文本的长度总是4的倍数，但是如果再加上1到2个=不就不是4的倍数了吗？
* 所以并不是先编码，再加上1到2个=，而是编码之后，把最后的1到2个字符（这个字符肯定是A）替换成=
* 改进：

```
1.  标准Base64里是包含 + 和 / 的，在URL里不能直接作为参数，所以出现一种 “url safe” 的Base64编码，其实就是把 + 和 / 替换成 - 和 _ 。
2.  同样的，=也会被误解，所以编码后干脆去掉=，解码时，自动添加一定数量的等号，使得其长度为4的倍数即可正常解码了。
```

* 对于base系列的编码解码，可在下面的在线网站进行解码： [base16，base32,base64编码在线](https://www.qqxiuzi.cn/bianma/base.php)
* base16相当于是16进制，可以直接16进制转字符串
* base64编码图


![](/images/pasted-0.png)

## c语言实现base64：


* base64编码、解码实现
* C语言源代码


* 使用说明：
* 命令行参数说明：若有“-d”参数，则为base64解码，否则为base64编码。
* 若有“-o”参数，后接文件名，则输出到标准输出文件。
* 输入来自标准输入stdin，输出为标准输出stdout。可重定向输入输出流。

* base64编码：输入任意二进制流，读取到文件读完了为止（键盘输入则遇到文件结尾符为止）。输出纯文本的base64编码。

* base64解码：输入纯文本的base64编码，读取到文件读完了为止（键盘输入则遇到文件结尾符为止）。输出原来的二进制流。

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <io.h>
#include <fcntl.h>
#include <stdbool.h>

#ifndef MAX_PATH
#define MAX_PATH 256
#endif

const char * base64char = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

char * base64_encode( const unsigned char * bindata, char * base64, int binlength )
{
    int i, j;
    unsigned char current;

    for ( i = 0, j = 0 ; i < binlength ; i += 3 )
    {
        current = (bindata[i] >> 2) ;
        current &= (unsigned char)0x3F;
        base64[j++] = base64char[(int)current];

        current = ( (unsigned char)(bindata[i] << 4 ) ) & ( (unsigned char)0x30 ) ;
        if ( i + 1 >= binlength )
        {
            base64[j++] = base64char[(int)current];
            base64[j++] = '=';
            base64[j++] = '=';
            break;
        }
        current |= ( (unsigned char)(bindata[i+1] >> 4) ) & ( (unsigned char) 0x0F );
        base64[j++] = base64char[(int)current];

        current = ( (unsigned char)(bindata[i+1] << 2) ) & ( (unsigned char)0x3C ) ;
        if ( i + 2 >= binlength )
        {
            base64[j++] = base64char[(int)current];
            base64[j++] = '=';
            break;
        }
        current |= ( (unsigned char)(bindata[i+2] >> 6) ) & ( (unsigned char) 0x03 );
        base64[j++] = base64char[(int)current];

        current = ( (unsigned char)bindata[i+2] ) & ( (unsigned char)0x3F ) ;
        base64[j++] = base64char[(int)current];
    }
    base64[j] = '\0';
    return base64;
}

int base64_decode( const char * base64, unsigned char * bindata )
{
    int i, j;
    unsigned char k;
    unsigned char temp[4];
    for ( i = 0, j = 0; base64[i] != '\0' ; i += 4 )
    {
        memset( temp, 0xFF, sizeof(temp) );
        for ( k = 0 ; k < 64 ; k ++ )
        {
            if ( base64char[k] == base64[i] )
                temp[0]= k;
        }
        for ( k = 0 ; k < 64 ; k ++ )
        {
            if ( base64char[k] == base64[i+1] )
                temp[1]= k;
        }
        for ( k = 0 ; k < 64 ; k ++ )
        {
            if ( base64char[k] == base64[i+2] )
                temp[2]= k;
        }
        for ( k = 0 ; k < 64 ; k ++ )
        {
            if ( base64char[k] == base64[i+3] )
                temp[3]= k;
        }

        bindata[j++] = ((unsigned char)(((unsigned char)(temp[0] << 2))&0xFC)) |
                ((unsigned char)((unsigned char)(temp[1]>>4)&0x03));
        if ( base64[i+2] == '=' )
            break;

        bindata[j++] = ((unsigned char)(((unsigned char)(temp[1] << 4))&0xF0)) |
                ((unsigned char)((unsigned char)(temp[2]>>2)&0x0F));
        if ( base64[i+3] == '=' )
            break;

        bindata[j++] = ((unsigned char)(((unsigned char)(temp[2] << 6))&0xF0)) |
                ((unsigned char)(temp[3]&0x3F));
    }
    return j;
}

void encode(FILE * fp_in, FILE * fp_out)
{
    unsigned char bindata[2050];
    char base64[4096];
    size_t bytes;
    while ( !feof( fp_in ) )
    {
        bytes = fread( bindata, 1, 2049, fp_in );
        base64_encode( bindata, base64, bytes );
        fprintf( fp_out, "%s", base64 );
    }
}

void decode(FILE * fp_in, FILE * fp_out)
{
    int i;
    unsigned char bindata[2050];
    char base64[4096];
    size_t bytes;
    while ( !feof( fp_in ) )
    {
        for ( i = 0 ; i < 2048 ; i ++ )
        {
            base64[i] = fgetc(fp_in);
            if ( base64[i] == EOF )
                break;
            else if ( base64[i] == '\n' || base64[i] == '\r' )
                i --;
        }
        bytes = base64_decode( base64, bindata );
        fwrite( bindata, bytes, 1, fp_out );
    }
}

void help(const char * filepath)
{
    fprintf( stderr, "Usage: %s [-d] [input_filename] [-o output_filepath]\n", filepath );
    fprintf( stderr, "\t-d\tdecode data\n" );
    fprintf( stderr, "\t-o\toutput filepath\n\n" );
}

int main(int argc, char * argv[])
{
    FILE * fp_input = NULL;
    FILE * fp_output = NULL;
    bool isencode = true;
    bool needHelp = false;
    int opt = 0;
    char input_filename[MAX_PATH] = "";
    char output_filename[MAX_PATH] = "";

    opterr = 0;
    while ( (opt = getopt(argc, argv, "hdo:")) != -1 )
    {
        switch(opt)
        {
        case 'd':
            isencode = false;
            break;
        case 'o':
            strncpy(output_filename, optarg, sizeof(output_filename));
            output_filename[sizeof(output_filename)-1] = '\0';
            break;
        case 'h':
            needHelp = true;
            break;
        default:
            fprintf(stderr, "%s: invalid option -- %c\n", argv[0], optopt);
            needHelp = true;
            break;
        }
    }
    if ( optind < argc )
    {
        strncpy(input_filename, argv[optind], sizeof(input_filename));
        input_filename[sizeof(input_filename)-1] = '\0';
    }

    if (needHelp)
    {
        help(argv[0]);
        return EXIT_FAILURE;
    }

    if ( !strcmp(input_filename, "") )
    {
        fp_input = stdin;
        if (isencode)
            _setmode( _fileno(stdin), _O_BINARY );
    }
    else
    {
        if (isencode)
            fp_input = fopen(input_filename, "rb");
        else
            fp_input = fopen(input_filename, "r");
    }
    if ( fp_input == NULL )
    {
        fprintf(stderr, "Input file open error\n");
        return EXIT_FAILURE;
    }

    if ( !strcmp(output_filename, "") )
    {
        fp_output = stdout;
        if (!isencode)
            _setmode( _fileno(stdout), _O_BINARY );
    }
    else
    {
        if (isencode)
            fp_output = fopen(output_filename, "w");
        else
            fp_output = fopen(output_filename, "wb");
    }
    if ( fp_output == NULL )
    {
        fclose(fp_input);
        fp_input = NULL;
        fprintf(stderr, "Output file open error\n");
        return EXIT_FAILURE;
    }

    if (isencode)
        encode(fp_input, fp_output);
    else
        decode(fp_input, fp_output);
    fclose(fp_input);
    fclose(fp_output);
    fp_input = fp_output = NULL;
    return EXIT_SUCCESS;
}
```
* 除了以上的三种编码外，base系列还有其他的几种
* base58,base85,base91,base92，base128

* base58的应用：比特币、Monero、Ripple、Flickr都在用这个Base58的编码方式

* base58编码表：

![](/images/pasted-1.png)
* 也就是字符1代表0，字符2代表1,字符3代表2…字符z代表57。然后回一下辗转相除法。
* 如要将1234转换为58进制；
* 第一步：1234除于58，商21，余数为16，查表得H
* 第二步：21除于58，商0，余数为21，查表得N
* 所以得到base58编码为：NH
* 如果待转换的数前面有0怎么办？直接附加编码1来代表，有多少个就附加多少个（编码表中1代表0）
* 大值思路为：

```
    将数据转换为大整数x，
    依次将（x % 58）的值表示的编码添加到输出字符串末尾；
    令x = ( x / 58 );
    重复2-3，直到x等于0；
    将数据前所有0的编码（即“1”）添加到输出字符串末尾；
    将输出字符串反转，即为Base58编码字符串。
    由于编码前数据可以视为256进制编码数据，所以转换为58进制编码数据后，数据长度为变长。
```
* 即：
* Base58的本质就是把256进制的值转成58进制的值。
* 所以它在编码时不需要考虑补\x00的问题，直接转换即可。
* 把字节流转成一个256进制的大数，然后不断除以58，保留余数，最后余数当作索引，再倒序，即为转换后的结果。

* 特殊处理：
* 不同于一个普通的数字转成某个进制，普通数字最高位是不会为0的，而我们要编码的对象是字节流，那么如果字节流的最前面是0（\x00），那么就会丢失这个信息。所以编码时要特殊记录一下，字节流的开端有多少个\x00，就直接在转换后的编码前面加上多少个b58Alphabet[0]，同理，解码的时候先记录一下前面的b58Alphabet[0]的个数，然后解码之后再在解码的前面加上相同数量的0x00。

* 假设原长度为Len256，转换为长度为Len58，则：
* Len58 = Len256 * ( log256 / log58 ) + 1
* 使用C语言进行base58的加密：

![](/images/pasted-2.png)

![](/images/pasted-3.png)

![](/images/pasted-4.png)

![](/images/pasted-5.png)

![](/images/pasted-6.png)

* 关于base64和base58的总结：
* 不管是Base64还是Base58，都会造成信息的冗余，使得需要传输的数据量增大，所以不会用在很大的数据上。

```
    使用Base64最普遍的是URL、邮件文本、图片；
    相比于Base64直接切割比特的方法（3个比特变为4个比特），Base58采用的大数进制转换，效率更低，所以使用场景的数据更少，例如上面提到的比特币的地址的编码。
```
* 下一节我们讲述Unicode编码