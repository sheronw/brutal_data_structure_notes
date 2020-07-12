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

## 二分搜索

不解释了。

递归：

```javascript
var search = function(nums, target) {
    return helper(0,nums.length,nums,target);
};

var helper = function(start,end,nums,target) {
    if(end>start) {
        let mid = Math.floor(start+(end-start)/2);
        if(nums[mid]==target) return mid;
        else if(nums[mid]>target) return helper(start,mid,nums,target);
        else return helper(mid+1,end,nums,target);
    }
    else return -1;
};
```

遍历：

```javascript
var search = function(nums, target) {
    let start = 0;
    let end = nums.length;
    while(true) {
        if(end<=start) return -1;
        let mid = Math.floor(start+(end-start)/2);
        if(nums[mid]==target) return mid;
        else if (nums[mid]>target) end = mid;
        else start = mid + 1;
    }
    return -1;
};
```

## 数组排列

给定一个元素**各不相同**的整数数组，返回所有可能的排列。

一般都是用DFS递归，速度上的差异和实现时用的方法有关。时间复杂度O(n!)。如果不想每次筛选，可以放一个数组来储存是否已经访问过。

```javascript
var permute = function(nums) {
    if(nums.length<=1) return [nums];
    let res = [];
    for(let n of nums) {
        let temp = permute(nums.filter(x=>x!=n));
        temp.forEach(a => a.push(n));
        res = res.concat([...temp]);
    }
    return res;
};
```

还有一个进阶版，是给定一个元素**可能重复**的整数数组，返回所有可能的排列。

如果要从上面的解中改动得到一个暴力算法的话，大概就是用filter筛选index而不是数字，然后再用set存起来查重。但毕竟转换来转换去的很麻烦，最好是寻求其他的方案。

这个时候可以找个实例画画图找规律，你会发现在同一层级中（比如，第x个数组的元素的选择），选择相同的数字所得到的结果是一样的，比如第0个元素选了两个1中的任意一个，后面的其他元素的排列都是相同的，那么我们可以在遍历递归的时候只将1递归一次。如何保证在每个层级中不遍历重复的数字呢？方法其实有很多，比如新建一个数组或者Set标注是否重复过。有一个不需要外部存储的方法是将源数组排序，这样从第一个元素到最后一个元素一个一个遍历的时候，只要当前元素与上一个元素相等，那么就可以跳过该元素，在每次递归后写一个while loop将重复数字跳过即可。

```javascript
var permuteUnique = function(nums) {
    if(nums.length<=1) return [nums];
    nums.sort();
    let res = [];
    for(let i=0;i<nums.length;i++) {
        let temp = permuteUnique(nums.filter((x,index)=>index!=i));
        temp.forEach(a => a.push(nums[i]));
        res = res.concat([...temp]);
        while(nums[i+1]==nums[i]) i++;
    }
    return res;
};
```

## 数组组合

给定一个**各不相同**的整数数组，返回所有可能的排列。

还是老样子，找个例子画图。一个可能的尝试是分别列出长度为1、2、3……n的组合，但并不是很好找规律。还有一种方式是列出以不同的数字为起始的组合，然后就会发现以每一个数字为开始的所有组合为这个数字加上其他所有现有数字的组合。以[1,2,3,4]为例：

| 4    | 3    | 2    | 1    |
| ---- | ---- | ---- | ---- |
| 34   | 23   | 12   |      |
| 234  | 123  |      |      |
| 1234 | 13   |      |      |
| 134  |      |      |      |
| 24   |      |      |      |
| 124  |      |      |      |
| 14   |      |      |      |

按照上表的规律写出代码：

```javascript
var combination = function(nums) {
  let res = [];
  for(let n of nums) {
    res.push([n]);
    let curres = [];
    for(let x of res) {
      let temp = x.slice();
      temp.push(n);
      curres.push(temp);
    }
    res = res.concat(curres);
  }
  return res;
};
```

如果假设元素**可能重复**，那么要在每一个循环中确保不会出现重复。和排列一样，先排序，然后判断是否与上一个元素相同。如果相同的话虽然不跳过，但是可以和上一个元素（也就是说和他相同的元素）进行组合，原因是上一个元素已经和其他元素组合过了，所以不需要再来一次，并且这次不要添加这个元素本身。

## 电话号码的字母组合

这道题[领扣](https://www.lintcode.com/problem/letter-combinations-of-a-phone-number/description)上有。

给一个不包含0和1的数字字符串，每个数字代表一个字母，返回其所有可能的九宫格按键字母组合。

首先，为了方便起见，我们可以写一个 helper function 来获取九宫格键盘对应的字母：

<del>PIE很贴心，把所有按键都搞成仨字母的了，然后发现领扣的题里面7和9是四个字母（。</del>

```javascript
	/**
   * Recursively get the minimum value in the tree.
   * @param {number} the number on the key
   * @param {number} 1st, 2nd or 3rd letter of the key
   * @return {String} corresponding letter
   */
var getCharKey = function(keyStr, place) {
  let key = parseInt(keyStr);
  let charCode = 0;
  if (key<=6) charCode = 96+(key-2)*3+place;
  if (key==7) charCode = 111+place;
  if (key==8) charCode = 115+place;
  if (key==9) charCode = 118+place;
  return String.fromCharCode(charCode);
};
```

老样子，以23为例，就会发现和刚刚的题目是类似的：

| 2    | 3    |
| ---- | ---- |
| A    | AA   |
| B    | AB   |
| C    | AC   |
|      | BA   |
|      | BB   |
|      | BC   |
|      | CA   |
|      | CB   |
|      | CC   |

对每一按键，在上一个按键的基础上每个数组分别加这个按键的仨字母就可以了。按键顺序是固定的，没有查重的必要。

```javascript
var letterCombinations = function (digits) {
    let res = [];
    if (!digits) return res;
    res.push("");
    for(let i=0; i<digits.length; i++) {
        let temp = [];
        for(let j=1; j<4; j++) {
            for(let e of res) temp.push(e.concat(getCharKey(digits[i],j)));
        }
        if(digits[i]=="7" || digits[i]=="9"){
            for(let e of res) temp.push(e.concat(getCharKey(digits[i],4)));
        }
        res = temp.slice();
    }  
    return res;
};
```

