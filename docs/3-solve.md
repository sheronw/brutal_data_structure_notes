# 数组、字符串 etc. 相关题目解析

## 倒置字符串中单词的顺序

给定一个字符串，将字符串一个一个单词地倒置，见[LC](https://leetcode.com/problems/reverse-words-in-a-string/)。

用C系语言的话，有更加巧妙的解法，但用js的话还是老办法——新建个array然后一个一个往里放，最后合起来。

*Programming Interview Exposed* 这本书里面主要考察C语言，但在LC中还多了个坑：要求去除所有多余的空格。

```
Input: "the sky is blue"
Output: "blue is sky the"

Input: "  hello world!  "
Output: "world! hello"

Input: "a good   example"
Output: "example good a"
```

一个一个遍历检查也可以，不用多说。

```javascript
var reverseWords = function(str) {
    let arr = str.trim().split(" ");
    let temp = "";
    for(let i=arr.length-1;i>=0; i--) {
        if(arr[i]){
            temp += arr[i].trim();
            if (i!=0) temp += " ";
        }
    }
    return temp;
};
```

但如果用js刷题的话，有现成的String / Array method可以用：首先想到的是`split`，之后也有现成的`reverse`。每个单词之间的多余空格有些不太好考虑，可以用`trim`把源字符串首尾的全搞掉，然后split的时候匹配正则。O(n)的时间。

<del>奇怪的是咱看LC最快的答案用的是这个方法，但跑出来比上一种方案还慢hmm</del>

```javascript
var reverseWords = function(str) {
  let arr = str.trim().split(/\s+/);
  return arr.reverse().join(" ");
};
```

## 字符串转换为整数

LC中[atoi的实现](https://leetcode.com/problems/string-to-integer-atoi/)。给定一个字符串，要求将其转换成一个整数，开头会出现空格，并且不论有没有找到数字，都会在第一个非数字的字符停止，没有找到则返回0。

首先我们要把开头的空格去掉，可以一个一个遍历（更省内存），也可以省事直接trim。然后继续遍历。下一个出现的合法非数字字符只能是正负号，然后判断这个，如果没有正负号的话之后都算非法输入。然后再找数字，直到出现不是数字的字符就返回。可以一个一个字符乘以10放到计数器里（js虽然没有todigit的方法，但可以获取char code然后再做运算），也可以搞一串字符串直接取一个整数。

```javascript
// 第一种，不优雅的缝缝补补方案
var myAtoi = function(str) {
    let index = 0;
    let start = -1;
    let end = -1;
    let isNeg = undefined;
    while(str[index]==" " & index<str.length) {
        index++;
    }
    while(index<str.length) {
        if (str[index]=="-" && isNeg===undefined && start==-1) isNeg=true;
        else if (str[index]=="+" && isNeg===undefined && start==-1) isNeg=false;
        else if (str[index].match(/^\d$/)) {
            if (start==-1) start = index;
        }
        else {
            if (start==-1) return 0;
            end=index;
            break;
        };
        index++;
    }
    if (end==-1 && start==-1) return 0;
    if (end==-1) end = str.length;
    let count = parseInt(str.slice(start,end));
    count = isNeg===true ? -1 * count: count;
    
    if (count<-2147483648) return -2147483648;
    if (count>2147483647) return 2147483647;
    else return count;
};
```

```javascript
// 重构后
var myAtoi = function(str) {
    let index = 0;
    let isNeg = false;
    let count = 0;
    
    while(str[index]==" " & index<str.length) {
        index++;
    }
    
    if (str[index]=="+" || str[index]=="-") {
        isNeg = str[index]=="-" ? true : false;
        index++;
    }
    
    while(index<str.length) {
        if (str[index].match(/^\d$/)) {
            let digit = str.charCodeAt(index)-48;
            count = count * 10 + digit;
        }
        else break;
        index++;
    }

    count = isNeg ? -1 * count: count;
    
    return Math.max(-(2**31), Math.min(2**31 - 1, count));
};
```

## UTF-8 验证

这个[LC](https://leetcode.com/problems/utf-8-validation/)上也有。

UTF-8中的字符长度为1-4字节不等，并遵守以下规律：

- 1字节的字符，第一个比特为0，之后为它的unicode
- n字节的字符，前n个比特都为1，第n+1比特为0，紧跟着n-1个「前两位是10的」字节

```
   Char. number range  |        UTF-8 octet sequence
      (hexadecimal)    |              (binary)
   --------------------+---------------------------------------------
   0000 0000-0000 007F | 0xxxxxxx
   0000 0080-0000 07FF | 110xxxxx 10xxxxxx
   0000 0800-0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx
   0001 0000-0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
```

给定一个字符串数组，要求返回其是否是合法的UTF-8编码，其中可能含有好几个字符。

我们需要判断的数字开头分别需要为0、10、110、1110、11110。对于二进制相关的题目，用位运算的方法比较合适，比如遮罩（中文大概是这个？）

先拿以0开头为例，x0000000 AND 任何数字得到的都是y0000000（y是这个数字本来的首位），所以只要判断这个数字 AND 10000000 是否为零即可判断是否以0开头。

同样地，以10为开头的话 AND 11000000 为 10000000、以110为开头的话 AND 11100000 为 11000000、以1110为开头的话 AND 11110000 为 11100000、以11110为开头的话 AND 11111000 为 11110000。

```javascript
var check0 = n => (n & 0b10000000) == 0;
var check10 = n => (n & 0b11000000) == 0b10000000;
var check110 = n => (n & 0b11100000) == 0b11000000;
var check1110 = n => (n & 0b11110000) == 0b11100000;
var check11110 = n => (n & 0b11111000) == 0b11110000;
```

然后再解决题目相关的问题，最传统的是用while循环然后根据情况讨论：

```javascript
var validUtf8 = function(data) {
    let i = 0;
    while(i<data.length) {
        if (check0(data[i])) i++;
        else if (check110(data[i])) {
            i++;
            if(check10(data[i])) i++;
            else return false;
        }
        else if (check1110(data[i])) {
            i++;
            if(check10(data[i]) && check10(data[i+1])) i+=2;
            else return false;
        }
        else if(check11110(data[i])) {
            i++;
            if(check10(data[i]) && check10(data[i+1]) && check10(data[i+2])) i+=3;
            else return false;
        }
        else return false;
    }
    return true;
};
```

不过观察就可以得出，`check10`这个函数出现频率太高了，最好是想办法将需要判断它的条件汇总到一起。这种情况下可以新建一个计数器记录需要判断`check10`的次数。如果是字符的第一个字节的话肯定是不需要的，此时为0；根据第一个字节的情况可以计算出之后需要判断多少次，然后每判断成功一次计数器就减一；如果输入合法，遍历一遍之后计数器应该为0。

```javascript
var validUtf8 = function(data) {
    let count = 0;
    for(let byte of data) {
        if (count==0) {
            // beginning of a character
            if(check0(byte)) continue;
            else if(check110(byte)) count=1;
            else if(check1110(byte)) count=2;
            else if(check11110(byte)) count=3;
            else return false;
        }
        else {
            if (check10(byte)) count--;
            else return false;
        }
    }
    return count==0;
};
```

