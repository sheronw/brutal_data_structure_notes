# 链表相关题目分析

## 栈的实现

栈是一种LIFO（Last In First Out）的数据结构，每push一个数据都会放在数据的最前端，每pop一个数据也是拿走最上面的。一般来说，栈有两种实现方式——动态数组和链表。动态数组的优势在于能够快速获取每一个元素，但如果用它来实现栈的话，用不到它这个优势，因为至涉及到顶端元素的操作；而且，动态数组需要定期扩容，需要耗费一点时间。链表则是动态分配每一个新节点的内存，因为只需修改顶端元素，所以完全可以接受；但从底层内存分配的角度来说，性能不如动态数组，因为动态数组的元素是相邻的而链表使用的是指针，在数据量非常大的情况下链表可能性能不如动态数组。

但这次我们决定用链表实现。首先需要有一个节点class，里面存有数据和next指针。然后是栈class，储存其本身的长度以及头节点。栈包含push、pop、peek等方法。push需要存储的变量并返回栈长度，pop/peek返回头部的值或者在失败时返回undefined。

```javascript
// define the node calss
class Node() {
  constructor(val) {
    this.val = val;
    this.next = null;
  }
}

// define stack
class Stack() {
  constructor() {
    this.length=0;
    this.head=null;
  }
  
  // O(1)
  push(val) {
    const node = new Node(val);
    // could assign to this.next even this.next is null
    node.next = this.next;
    this.next = node;
    this.length++;
  }
  
  // O(1)
  pop() {
    if(this.head) {
      const cur = this.head;
      this.head = this.head.next;
      this.length--;
      cur.next = null;
      return cur.val;
    }
    else return undefined;
  }
  
  // O(1)
  peek() {
    return this.head ? undefined : this.head.val;
  }
}
```

## 链表中倒数第M个元素

首先，你要问清楚链表接口、面试官第M个是从0开始计还是从1开始、需不需要考虑元素个数少于M的情况，还有如果输入不合法需要返回什么值。我反正自作主张地认为是从1开始计，并且需要考虑元素少于M的情况。

如果有必要的话，应该（不合时宜地）提出在实际情况中，如果有获取倒数第几个元素的需求，那么单向的链表显然不是最好的选择，双向链表和动态数组更方便一些。

最简单粗暴的方法显然是一个一个节点进行测试，以找到正好是倒数第M个的那个节点并返回，时间复杂度是O(n<sup>2</sup>)。

那么把所有的元素都存储到某个外部数据结构呢？比如将每一个元素都放进栈里，然后拿出第M个。这样虽然时间复杂度降到了O(n)，但也需要O(n)的空间复杂度，如果数据足够多，就有些耗内存了。

好吧，既然是单向的数据结构，那么算出这个倒数第M个是正数第多少个就好了，有些链表内置了长度，即使没有的话也只需要加个计数器遍历（时间O(n)）就好了。按照我们的定义，倒数第M个应该是正数第「长度减M加1」个。然后从头找过去时间仍然是O(n)。

如果你去题库看解析，一般最上面的答案肯定让你用双指针——找正数第M个的方法是创造了一把长度为M（或者M-1，随便你吧）的尺子，然后把它的前端放到链表第一个元素，用末尾代表结果；将这把尺子从起点开始向后滑动直到后端卡到终点，尺子的前端就是我们想要的结果了。这两个指针纠就分别是尺子的前端和后端。

我们可以用一个简单的例子来测试。假设一个链表有五个元素，我们需要找倒数第2个元素，那么指针1从头开始遍历到第2个元素之后添加了一个新的从头开始的指针2，他们同步向前3个元素之后，指针1指向最后一个元素，而指针2指向倒数第2个元素。

```javascript
// define the node class
class Node() {
  constructor(val) {
    this.val = val;
    this.next = null;
  }
}

// O(n)
function MthToLast(list,m) {
  if (m<0) return null;
  let p1 = list.head;
  // go to the Mth node
  for(let i=1; i<m; i++) {
    p1=p1.next;
    // if is null then out of boundary
    if (p1===null) return null;
  }
  let p2 = list.head;
  // go until p1 is the last one
  while (p1.next!==null) {
    p1=p1.next;
    p2=p2.next;
  }
  return p2.val;
}
```

## 链表平整化

假设在双向链表的基础上，我们为每一个节点都多添加了一个child指针，它可以指向其他**链表**（的头部）也可以为空，这样就组成了一个非线性的结构。要求将所有相关联的的节点组合成一个线性的双向链表。

```javascript
// define the node class
class Node() {
  constructor(val) {
    this.val = val;
    this.next = null;
    this.prev = null;
    this.child = child;
  }
}
```

乍一看这个数据结构非常时尚，但如果找一个例子画画图仔细观察的话，会发现它和树很类似，所以你完全可以搞一个深度或者广度遍历。比如搞一个队列和一个新链表，把头部放进队列里面，当队列不为空时，拿出来一个，遍历到尾部，如果中途有节点还有child就放进队列里面……直到队列为空。

但是这个队列真的有必要吗？虽然它看起来是树，但却又不是树。因为树中的同级节点是不相连的，所以如果想要变成线性结构的话，的确就是使用标准的深度/广度遍历。然而我们要处理的这个结构同层级之间是互联的，只要每一层头尾相接，那么就直接变成线性结构了，不需要栈来缓存。既然我们每次都要遍历到尾部再添加新的链表，那么为什么不直接用两个指针，一个用来添加新链表并更新尾部，另一个负责寻找有child的节点？

```javascript
function flattenList(head) {
  let tail = head;
  let cur = head;
  // get the tail
  while(tail.next) {
    tail=tail.next;
  }
  while(cur) {
    if(cur.child) {
      //connect to the main linked list
      cur.prev = tail;
      tail.next = cur;
      // update tail
      while(tail.next) {
        tail=tail.next;
      }
    }
    cur=cur.next;
  }
  return head; // not sure if this line is meaningful, though
}
```

## 链表反向平整化

既然将上题中的多个链表合为一个了，那么是否能够根据child的信息将他们的连接再次切断呢？我们可以根据上题的思路进行反向分析。

比如，我们可以使用某种外部数据结构（列表、队列和栈都可）将所有元素的child存储起来，然后再重新遍历，切断这些child的连接。

如果要求不使用外部结构的话呢？如果像上一道题那样从头开始遍历然后逐一切断的话，因为就会把每一个列表切断成为sublist，就无法遍历到最后了，不过如果把问题变成「处理每一个sublist」的话，可以尝试考虑递归来解决问题—— 一个列表的所有sublist与列表的连接，然后对每一个sublist进行同样地操作。在这个算法中每一个节点都只能被遍历一次，因此平均时间复杂度仍然是O(n)，但优势是写起来更简单。

```javascript
function unflattenList(head) {
  if(head.prev) head.prev.next=null;
  head.prev = null;
  let cur = head;
  while(cur) {
    if (cur.child) {
      unflattenList(cur.child);
    }
    cur=cur.next;
  }
  return head;
}
```

如果提供的数据结构中，链表本身包含了尾部指针，那么逆向思考上一道题的最后一种解，我们还可以：从尾部开始拆。因为从尾部开始，每一个sublist必定是上一层sublist的某个child，那么如果从后往前拆的话，完全可以不遇到断点一路遍历到头部。

```javascript
function unflattenHeadTailList(head,tail) {
  if (!tail) return null;
  let cur = tail;
  while(cur) {
    if(cur.child) {
      cur.child.prev.next = null;
      cur.child.prev = null;
    }
    cur = cur.next;
  }
  return head;
}
```

## 链表是否循环

给定一个链表的头部指针，要求判定链表为普通链表，还是环形列表（列表的末尾可能指向某个链表中的元素）。如为普通则返回false，环形则返回true。 

一种很直觉的方法是，每检查一个节点，就将它的next节点和所有已经遍历的元素一一比对，查看是否相等；如果相等，那么链表是环形的。至于已遍历的元素如何储存，虽然可以放进某个数据结构里，但直接保存头部指针每次从这里遍历更节省内存。不过，这样的话，时间复杂度就是O(n<sup>2</sup>)。

还有一种方案很tricky，如果不是刷过题的话很难想出来，就是和之前所有题一样，使用双指针。回顾之前做过的几道题，双指针可以做尺子来同步滚动，也可以当作两个不同的锚点根据需求滚动。在本题中，同步滚动似乎没什么意义，根据需求的话，我们也没有什么可以用来判断的关键条件（比如上一题中的是否有child）。不过环形链表有个特点——可以回头，也就是说，如果是普通链表，两个指针前进速率不同时，总是快的领先；而环形链表中，快的指针却可能因为跑得太快所以到了慢的指针的后方，这样遍历几次的话，总会出现快的指针正好等于慢的指针（或者是慢的指针的下一个节点）的情况。如果是普通链表的话时间复杂度就是O(n)；如果是环形链表的话，不管循环的节点部分有多少，慢节点转一圈、快节点转个两圈应该就能相遇了，所以最差也是O(3n)，因此总的时间复杂度仍然为O(n)。

```javascript
function isCyclicList(head) {
  let slow = head;
  let fast = head;
  while(true) {
    if(!fast.next || !fast) return false;
    if(fast.next==slow || fast==slow) return true;
    slow = slow.next;
    // if fast.next is null then it will return false before this line
    fast = fast.next.next;
  }
}
```

