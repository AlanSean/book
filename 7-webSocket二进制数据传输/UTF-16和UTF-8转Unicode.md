# 前言
今天来学习UTF8转Unicode，UTF16转Unicode以达成UTF8,UTF16之间的互转。
提炼成函数的公式我并没有放出来，我的目的只是为了更加理解 字符编码之间的关系。
如果你需要转码方式，可以找其他的库，或者根据我文章来进行提炼。
基本利用按位操作符 符号运算符就可以完成。

今天这里只做UTF8转Unicode，UTF16转Unicode， 后续转换可以看前面的文章。

[1.基础准备工作](https://juejin.im/post/5ca43ed2f265da30b3409645)
[2.Unicode转UTF16和UTF8](https://juejin.im/post/5ca5c689f265da30ca24a960)


## UTF16转Unicode
为了更好的理解，我们来使用上一期[Unicode转UTF-16](https://juejin.im/post/5ca5c689f265da30ca24a960#heading-4)的结果
来进行UTF16转Unicode，U+22222转UTF-16 = [0xd848,0xde22] = '𢈢'(这个字的长度为二，所以要获取他所有的charCodeAt)

```
function charCodeAt(str){

    var length = str.length,
        num = 0,
        utf16Arr = [];

    for(num; num < length; num++){
        utf16Arr[num] = '0x'+str[num].charCodeAt().toString(16);
    }

    return utf16Arr;
}
charCodeAt('𢈢');//['0xD848', '0xDE22']


```
### 计算utf-16 4字节的取值范围
上面代码获得了，这个字符的UTF-16编码数组，JS的字符串全部使用的UTF-16编码格式
回顾一下UTF-16的编码方式
**将Unicode值减去0x10000,得到20位长的值，再将其分为高10位和低10位，分别为2个字节，高10位和低10位的范围都在 0 ~ 0x3FF，高10位加0xD800,低十位加0xDC00**

首先我们先看字节问题，Unicode值在U+10000 ~ U+10FFFF时，会分为 两个2 字节,二进制 8位为一个字节,所以
UTF-16的四个字节的字符是两个 16位的二进制
并且根据UTF-16的编码方式的高位加0xD800  低位加0xDC00得出最小范围值
高10位最小值为0xD800，低10为最小值为0xDC00
再根据 高10位和低10位的范围都在 0 ~ 0x3FF得出最大范围值
高10位最大值为0xD800+0x3FF，低10为最大值为0xDC00+0x3FF

**所以高10位的取值范围为 0xD800 ~ 0xdbff**
**低10位的取值范围为 高10位的取值范围为 0xDC00 ~ 0xdfff**

我们已经得知了UTF16编码的高10位和低10位的取值范围所以可以进行判断 是否需要进行逆推转换

```
var strCode = charCodeAt('𢈢'),
    strCode0 = strCode[0],
    strCode1 = strCode[1];

if(strCode0 >= 0xD800 && strCode0 <= 0xDBFF){
    if(strCode1 !=undefined && strCode1 >= 0xDC00 && strCode1 <= 0xDFFF){
        //到了这里说明这个字符是四个字节就可以开始进行逆推了
        //高10位加0xD800,低十位加0xDC00，所以减去
        strCode0 = strCode0 - 0xD800 = 0xd848 - 0xD800  = 0x48 = 72;
        strCode1 = strCode1 - 0xDC00 = 0xDE22 - 0xDC00 = 0x222 = 546;
        //高10位和低10位进行拼接。 字符串或者乘法都行

        //1 字符串的方式拼接 我用抽象的方式来展现过程
        strCode0.toString(2)+strCode1.toString(2) = '1001000' + '1000100010' = '10010001000100010'
        parseInt(Number('10010001000100010'),2).toString(16)//74274 = 0x12222
        //Unicode转utf16时 将Unicode值减去0x10000，所以再进行加法
        0x12222 + 0x10000 = 0x22222; //答案是不是昨天选择的值呢

        //2 利用数学的方式进行转换
        //先给高10位从末位补10个0，也就是乘以10000000000(二进制) = 0x400(16进制) = 1024(十进制)
        strCode0*0x400 = 0x48*0x400 = 1001000*10000000000 = 1001000 10000000000 = 0x12000
        //再加上减去0xDC00后的低10位
        0x12000+0x222 = 0x12222
        //加上 Unicode转utf16时 将Unicode值减去的0x10000
        0x12222+0x10000 = 0x22222;
        //Unicode U+22222 = '𢈢';
        return;
    }
}
//不满足上面条件时，说明UTF16转Unicode 等于原值。不懂为什么就回顾上期的表格


```


## UTF8转Unicode
这里一样 使用[上期](https://juejin.im/post/5ca5c689f265da30ca24a960#heading-2)例子运算出的结果[0xe4, 0xb8,0x80]进行转换

由于JS环境的字符串是UTF16编码所以我这里直接使用十六进制串来进行转换


### 怎么判断二进制数据的字符是utf8中的几字节
根据数据的第一个字节来进行判断 这个字符是几个字节。
根据[表格](https://juejin.im/post/5ca5c689f265da30ca24a960#heading-1)找到编码规则,用来区分这个数据串的字符是几字节

**js是使用小端存储的，小端存储是符合我们的逻辑，大端是逻辑相反**
**[大小端模式](https://baike.baidu.com/item/%E5%A4%A7%E5%B0%8F%E7%AB%AF%E6%A8%A1%E5%BC%8F/6750542?fr=aladdin)**

比如 小端存储是0xxx xxxx 大端存储就是相反的 xxxx xxx0

##### utf8编码规则
1 字节 0xxx xxxx

2 字节 110x xxxx 10xxxxxx

3 字节 1110 xxxx 10xxxxxx 10xxxxxx

4 字节 1111 0xxx 10xxxxxxx 10xxxxxx 10xxxxxx

js是小端存储所以只需要按字节位进行对比即可。

utf8各字节编码规则鲜明差异比较大的是首个字节，所以只需要对比首个字节，就可得知是几个字节。


#### 对比规则
根据 按位与的特性，将原码的x对应，编码规则的位值转为0其他位保持不变（若有更好的判断方法，非常期待您的留言）

也可以使用 **[带符号右移操作符 >>>]()**（js并不会转换正负符号 所以可以进行放心使用）

对应编码规则右移n个位来进行判断值是否为0，110，1111。(只是猜想之一，并没有进行实际验证，目前仅实践了下面的方式)

#### 推导过程

 **根据[按位与](https://juejin.im/post/5ca43ed2f265da30b3409645#heading-3)用 1 来保留原码对应的编码规则位值以及x位值全部转换为0 来进行判断是几字节**

                二进制                        将x替换为0                   十六进制
1字节 char0 & 1xxx xxxx = 0xxx xxxx   char0 & 1000 0000 = 0000 0000  char0 & 0x80 = 0

2字节 char0 & 111x xxxx = 110x xxxx   char0 & 1110 0000 = 1100 0000  char0 & 0xE0 = 0xC0

3字节 char0 & 1111 xxxx = 1110 xxxx   char0 & 1111 0000 = 1110 0000  char0 & 0xF0 = 0xE0

4字节 char0 & 1111 1xxx = 1111 0xxx   char0 & 1111 1000 = 1111 0000  char0 & 0xF8 = 0xF0

上面的判断规则已经非常明了。

下面的转码 我就只进行三字节的转码规则，其他 若有兴趣，可自行参考3字节的方式进行推算（动手才是理解最好的方式）
```
var buffer = new ArrayBuffer(6);
var view = new DataView(buffer);
view.setUint8(0,0xe4);
view.setUint8(1,0xb8);
view.setUint8(2,0x80);
view.setUint8(3,0xe4);
view.setUint8(4,0xb8);
view.setUint8(5,0x80);
//[[Uint8Array]]: Uint8Array(6) [228, 184, 128, 228, 184, 128]

var byteOffset = 0,//起点从1开始
    char0,
    length = view.byteLength;//获取数据的字节数

while(byteOffset <length){
    var char0 = view.getUint8(byteOffset),char1,char2,char3;
    if((char0 & 0x80) == 0){
        //代表是一个字节
        byteOffset++;
        continue;
    }
    if((char0 & 0xE0) == 0xC0){
        //代表是2个字节
        byteOffset+=2;
        continue;
    }
    if((char0 & 0xF0) == 0xE0){
        //代表是3个字节
        //3 字节编码规则 1110 xxxx 10xxxxxx 10xxxxxx
        //进入这个区间时，char0是符合 1110 xxxx的规则的
        //利用按位与来进行截取char0对应编码规则x的位值 也就是 0000 1111  0xF
        //我们先转换第一个字节,二进制 速算法 先将二进制进行转换16进制(4位二进制为一位十六进制)
        //228 & 0xF  1110 0100 & 0000 1111 = 100 = 0x4 = 4

        char0 = char0 &  0xF = 4

        //第二字节进行转换
        //第二字节编码规则 10xx xxxx 同理利用按位与 0011 1111 0x3F
        //184 & 0x3F 1011 1000 & 0011 1111 = 11 1000 = 0x38 = 56
        char1 = view.getUint8(byteOffset++);
        char1 = char1 & 0x3F = 56

        //第三字节进行转换
        //第三字节编码规则 10xx xxxx 同理利用按位与 0011 1111 0x3F
        //128 & 0x3F 1000 0000 & 0011 1111 = 00 0000 = 0x00 = 0

        char2 = view.getUint8(byteOffset++)  
        char2 = char2& 0x3F = 0

        //下面才是重点,我们已经按字节转码完成 那么如何进行组合呢。
        //第一种方法，利用字符串进行拼接。
        //'100' + '11 1000' + '00 0000' = '0100 1110 0000 0000'
        //parseInt(100111000000000,2) = 19968
        //String.fromCharCode(19968) = '一'
        //上面 我抽象的用二进制的过程来展现的，但是实际转换中 是看不到二进制的。
        //parseInt(Number(char0.toString(2)  +  char1.toString(2) +  char2.toString(2)),2)

        //第二种方式，利用左移操作符 <<
        //编码规则 1110 xxxx 10xxxxxx 10xxxxxx 第一字节后面有12个x 所以第一字节末位补12个0
        //char0 >> 12 = 4 >> 12
        //00000000 00000000 00000000 00000100 >> 12 = 0100 0000 0000 0000 = 0x4000 = 16384
        //第二自己后面有6个x 所以第二字节补6个0
        //char1 >> 6 = 56 >> 12
        //00000000 00000000 00000000 00111000 >> 6 =       1110 0000 0000 = 0xE00 = 3584
        //第三字节为最后一个字节所以不需要末位补0                   
        //利用按位或 进行组合
        16384 | 3584 | 0 =                            0100 1110 0000 0000 = 0x4e00 = 19968
        //19968 < 0x10000(U+10000),不需要进行转码,调用String.fromCharCode即可
        //Unicode码就转换完成了。
        continue;
    }

    if( (char0 & 0xF8) == 0xF0){
        //代表是4个字节
        byteOffset+=4;
        continue;
    }
    throw RangeError('引用错误');
}
```

这里编码转换就完成了。
