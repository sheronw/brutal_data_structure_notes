# 哈希表相关题目分析

## 字符串中第一个未重复过的字符

第一种，暴力算法：从头开始让每个字符都与它之后的字符串比对，如果之后没有出现过和它相同的字符串则返回该字符，时间复杂度 O(n)。

为了不从头遍历找字符，我们需要用哈希表。不过如果用 js 实现的话可以用 Map。用哈希表来存储出现的字符串的频率，扫完一遍是 O(n)，然后从头遍历一遍看哪个出现频率为 1 就返回哪个，还是 O(n)。因为只问是否重复，所以你甚至不需要存出现频率，只需要存重复（true）和未重复（false）即可。

```javascript
var firstNonRepeated = function(str) {
  let hash = new Map();
  for(let i=0; i<str.length; i++) {
    hash.set(str[i], has.has(str[i]));
  }

  for (let i=0; i<str.length; i++) {
    if !hash.get(str[i]) return str[i];
  }
  return null;
};
```

## 移除字符串中的指定字符

给定两个字符串，要求移除 a 中所有 b 含有的字符。

暴力算法仍然是遍历 ab 中的字符，假设他们长度分别为 m 和 n 那么时间复杂度就是 O(mn)。首先将 b 中的字符串放入哈希表中可以减少每次遍历 b 时所耗费的复杂度，如果创建一个新字符串而不是对源字符串进行切割操作的话，可以减少切割操作耗费的时间和空间。（因为我用的是 js 实现，js 中字符串为 immutable，所以使用 array 可能效率更高。）

```javascript
var removeChars = function (str, rem) {
  let hash = new Map();
  for (let i = 0; i < rem.length; i++) {
    hash.set(rem[i], true);
  }

  let arr = [];
  for (let i = 0; i < str.length; i++) {
    if (!hash.has(str[i])) arr.push(str[i]);
  }
  return arr.join("");
};
```
