# 树相关题目分析

## 最大深度&最小深度

之前在[领扣](https://www.lintcode.com/problem/maximum-depth-of-binary-tree/description)上做过一遍，非常基础的题目。

```javascript
const maxDepth = function (root) {
    // O(n)
    if (!root) return 0;
    let left = root.left==null ? 0 : maxDepth(root.left);
    let right = root.right==null ? 0 : maxDepth(root.right);
    return Math.max(left+1, right+1);
};
```

然后发现自己在[LC](https://leetcode.com/problems/maximum-depth-of-binary-tree/)又做了一遍= =

```javascript
var maxDepth = function(root) {
    if (root) return 1 + Math.max(maxDepth(root.left),maxDepth(root.right));
    return 0;
};
```

然后发现自己在[LC](https://leetcode.com/problems/minimum-depth-of-binary-tree/)上还做了一道最小深度 = =

```javascript
var minDepth = function(root) {
    // note: when there is one missing child, just use another path!
    if (!root) return 0;
    if (!root.left && !root.right) return 1;
    let left = minDepth(root.left);
    let right = minDepth(root.right);
    if(left == 0 || right == 0) {
        return Math.max(left, right) + 1;
    }
        return Math.min(left, right) + 1;
};
```

## 前序遍历、中序遍历和后序遍历

[Basic & Binary Tree](5.%20Tree%20part%201.md)里面咱已经记录过一遍了，只不过都是伪代码，下面是用js的实现。

最简单的，可以全都用递归解（。

而且都是O(n)。

```javascript
// preorder 
var preorderTraversal = function(root) {
    let res = [];
    helper(root,res);
    return res;
};
var helper = function(root, res) {
    if (root!=null) {
        // (change the sequence to get inorder & postorder)
        res.push(root.val);
        if (root.left!=null) helper(root.left,res);
        if (root.right!=null) helper(root.right,res);
    }
};
```

既然可以用递归了，那还可以用栈来解（。

```javascript
// preorder
var preorderTraversal = function(root) {
    let res = [];
    let stack = [];
    if (root!=null) stack.push(root);
    while(stack.length!=0) {
        let cur = stack.pop();
        res.push(cur.val);
        if (cur.right!=null) stack.push(cur.right);
        if (cur.left!=null) stack.push(cur.left);
        
    }
    return res;
};

// inorder
var inorderTraversal = function(root) {
    let stack = [];
    let res = [];
    let cur = root;
    while(stack.length!=0 || cur!=null) {
        while(cur!=null) {
            stack.push(cur);
            cur = cur.left;
        }
        cur = stack.pop();
        res.push(cur.val);
        cur = cur.right;
    }
    return res;
}

// postorder
var postorderTraversal = function(root) {
    let res = [];
    let stack = [];
    let cur = root;
    let pre = null;
    while (cur!=null || stack.length!=0) {
        while(cur!=null) {
            stack.push(cur);
            cur = cur.left;
        }
        cur = stack[stack.length-1]; // peek
        if (pre==cur.right || cur.right==null) {
            res.push(cur.val);
            pre = stack.pop();
            cur = null;
        }
        else cur = cur.right;
    }
    return res;
};
```

## 最近公共祖先

这个[领扣](https://www.lintcode.com/problem/lowest-common-ancestor-of-a-binary-tree/description)上也做过（。 

就是递归地寻找左右节点的最近公共祖先，如果左右公共祖先分别为A、B那么返回该点，如果只有左节点有就返回右节点，反之亦然。O(n)

```python
def lowestCommonAncestor(root, A, B):
        # 1 A left B right => return root
        # 2 A left B left => return lowestCommonAncestor(root.left,A,B)
        # 3 A right B left => return root
        # 4 A right B right => return lowestCommonAncestor(root.right,A,B)
        if root is None:
            return None
        if root==A or root==B:
            return root
        left = lowestCommonAncestor(root.left,A,B)
        right = lowestCommonAncestor(root.right,A,B)
        if left==A and right==B:
            return root
        if left is None and right is None:
            return None
        if left is None:
            return right
        if right is None:
            return left
```

但是还有一个更简单的变种，就是**二分搜索树**的最小公共祖先。二分搜索树的特点在于，左子树的值全比根小，右子树的值全比根大。画几个图之后便可以找出规律——只要从根部开始找出第一个值在这两个节点值之间的即可。

```javascript
var lowestCommonAncestorBST = function(root, A, B) {
  const small = A.val < B.val ? A.val : B.val;
  const big = A.val > B.val ? A.val : B.val;
  while (root) {
    if (small > root.val) {
      // both larger than root, go right
      root = root.right;
    }
    else if (big < root.val) {
      // both smaller than root, go left
      root = root.left
    }
    else if (big > root.val && small < root.val) {
      return root;
    }
    else return null;
  }
  return null;
};
```

## 二叉树转换成堆

给定一个二叉树，要求用数组排序的方法将它变成一个用平衡二叉树实现的堆。题目本身不太难，遍历一遍放到数组里排个序，然后根据逆向层遍历的方式将数组构建回二叉树即可。

但是，比起做题本身，沟通也很重要，你应该提出用**数组排序并不是最好的构建堆的方式**，如果直接Heapify的话，只需要O(n)的时间复杂度，而但就数组排序这一部分就需要至少O(nlogn)的时间复杂度。

不过虽然是这么说，万一人家就是想考你树遍历呢，继续和面试官分析逻辑。要的是最大堆还是最小堆要弄清楚。首先要将二叉树变成一个数组，因为二叉树本身没有什么线索，所以怎么遍历都可以，但面试中写递归的那几个比较快；其次是排序，最快的就是快排和归并这俩模板，时间不允许也可以先用语言自带的方法，具体看面试官的要求。最后是建堆，其实如果是用数组实现的树的话，那么排好续的array已经满足堆的要求了，假设某一个节点的index为`i`，那么他的两个子节点的index分别为`2i+1`和`2i+2`，然后再问问对方需不需要把指针给补上，要是需要的话就按照这个思路排一遍即可。

```javascript
// suppose we need max heap
var heapifyBinaryTree = function(root) {
  // traversal & save to an array
  let arr = [];
  if(root) traverse(root,arr);
  
  // sort the array (choose one)
  // 1. use js built-in sort function
  arr.sort((a,b) => b.val-a.val);
  // 2. use quicksort
  quickSort(arr,0,arr.length-1);
  
  // link the nodes together
  for(let i=0; i<arr.length; i++) {
    if (i*2+1<arr.length) arr[i].left = arr[i*2+1];
    if (i*2+2<arr.length) arr[i].right = arr[i*2+2];
  }
  
  return arr[0]; // not that necessary
};

var traverse = function(root, arr) {
  arr.push(root);
  if (root.left) traverse(root.left, arr);
  if (root.right) traverse(root.right, arr);
};

var quickSort = function(arr,l,r) {
  if(left<right) {
    const index = partition(arr,l,r);
    quickSort(arr,l,index-1);
    quickSort(arr,index+1,r);
  }
};

var partition = function(arr,l,r) {
  let pivot = l;
  let cur = pivot+1;
  for(i=cur;i<r;i++) {
    if(arr[i]>arr[pivot]) {
      // swap cur and i
      const temp = arr[i];
      arr[i] = arr[pivot];
      arr[pivot] = temp;
      cur++;
    }
  }
  // because all the value before cur are greater than pivot
  // we have to swap the pivot to the middle
  const temp = arr[cur-1];
  arr[cur-1] = arr[pivot];
  arr[pivot] = temp;
  return cur-1;
}
```

我们可以再研究一下Heapify的方法：堆有两个常见操作，`siftup`和`siftdown`。一般来说，`siftup`对较靠上的节点比较友好，因为离顶部更近，需要执行的次数更少；`siftdown`同理对靠下的节点更友好。建堆的时候因为要对每一个节点都操作，而肯定是下面的节点更多，所以我们倾向于`siftdown`。

为了方便操作，和上一个算法相同，我们先把整个树遍历一遍放进数组里，这一步是O(n)。

虽然我们可以这么直接对每一个节点进行`siftdown`，但叶节点，也就是最下面那一排不能再搞了，所以可以从倒数第二排开始逐个`siftdown`。因为每层的节点树就是个等比数列求和，自己画个图推导下得出`n/2-1`就是第一个非叶的index，我们可以从这里开始向前搞。

至于对每个节点进行`siftdown`是O(n)而不是O(nlogn)的原因——刚刚有提到说最下面那排节点（占总节点的一半）都是不需要`siftup`的，那么需要执行的操作次数为0，再上面一排的节点数量是它的一半，操作次数为1。得到一个式子0×n/2+1×n/4+2×n/8+...这样就类似于那个泰勒级数了，最后求和的结果是n。

最后一步，和上一个算法一样，如果有要求的话再把指针补上，也是O(n)。

```javascript
// suppose we need max heap
var heapifyBinaryTree = function(root) {
  // traversal & save to an array
  let arr = [];
  if(root) traverse(root,arr); // just use traverse function in method 1
  
  let start = Math.floor(arr.length/2)-1;
  while(start>=0) {
    siftdown(arr,start,end);
    start--;
  }
  
  // link the nodes together (the same as method 1)
  for(let i=0; i<arr.length; i++) {
    if (i*2+1<arr.length) arr[i].left = arr[i*2+1];
    if (i*2+2<arr.length) arr[i].right = arr[i*2+2];
  }
  
  return arr[0]; // not that necessary
};

var siftdown = function(arr,start) {
  let cur = start;
  let left = start*2+1;
  let right = start*2+2;
  leftval = left < arr.length ? arr[left] : NaN;
  rightval = right < arr.length ? arr[right] : NaN;
  if(leftval && leftval>arr[start]) start = left;
  if (rightval && rightval>arr[start]) start = right;
  if(start!=cur) {
    let temp = arr[start];
    arr[start] = arr[cur];
    arr[cur] = temp;
    siftdown(arr,start);
  }
};
```



## 平衡二分搜索树

其实上一题用数组构建堆已经给了我们一些提示了：完全可以把树遍历一遍放进array，然后重新构建——因为BST左树总比右树小的特性，所以inorder自带顺序不用排序了。然后再从中间开始重新遍历树。这样是时间和空间复杂度都是O(n)。

```javascript
var balanceBST = function(root) {
    // get inorder array
    let arr = [];
    inorder(arr,root);
    return constr(arr,0,arr.length);
};

var inorder = function(arr,root) {
    if(root) {
        inorder(arr,root.left);
        arr.push(root);
        inorder(arr,root.right);
    }
};

var constr = function(arr,l,r) {
    if(l<r) {
        let mid = Math.floor(l+(r-l-1)/2);
        let root = arr[mid];
        root.left = constr(arr,l,mid);
        root.right = constr(arr,mid+1,r);
        return root;
    }
    else return null;
}
```

除了这种主流的方法，其实还可以仿照AVL树的方法。但AVL树只需要在插入移除时检查是否需要扭树，但我们要针对每一个节点进行检查是否平衡，如果不是的话就递归地处理两个子树，然后再最终检查一遍。虽然扭树本身是O(1)，但因为检查平衡比需要计算高度，而每次计算高度都需要O(logn)的时间复杂度，所以总合起来就是O(nlogn)，比重新构建要慢，但好处是可以达到O(1)的空间复杂度。

AVL扭树的方法在[Binary Search Tree & AVL Tree & Red Black Tree](6.%20Tree%20part%202.md)有详细的记录。自己在实现过程中遇到的坑是扭两次的判断条件不是判断是否平衡（小于等于1），而是只要一比另一边高就需要单独多扭一次。

```javascript
var balanceBST = function(root) {
    if (!root) return null;
    root.left = balanceBST(root.left);
    root.right = balanceBST(root.right);
    // get height for all the trees
    let ll,lr,rl,rr;
    if (root.left) {
        ll = getHeight(root.left.left);
        lr = getHeight(root.left.right);
    }
    else ll=lr=-1;
    if (root.right) {
        rl = getHeight(root.right.left);
        rr = getHeight(root.right.right);
    }
    else rl=rr=-1;
    // count height of left & right subtree
    let hleft = Math.max(ll,lr)+1;
    let hright = Math.max(rl,rr)+1;

    // check if we need to rotate
    if(hright-hleft>1) {
        // more on the right
        if(rl-rr>0) {
            // twisted
            root.right = rotateRight(root.right);
        }
        root = rotateLeft(root);
    }
    else if(hleft-hright>1) {
        // more on the left
        if(lr-ll>0) {
            // twisted
            root.left = rotateLeft(root.left);
        }
        root = rotateRight(root);
    }
    else return root;
    return balanceBST(root);
};

var rotateLeft = function(root) {
    let temp = root.right;
    root.right = temp.left;
    temp.left = root;
    return temp;
}

var rotateRight = function(root) {
    let temp = root.left;
    root.left = temp.right;
    temp.right = root;
    return temp;
}

var getHeight = function(root) {
    if (root) return 1 + Math.max(getHeight(root.left),getHeight(root.right)); 
    else return 0;
}
```