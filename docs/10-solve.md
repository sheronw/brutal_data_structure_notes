# 图相关题目分析

## Kevin Bacon 数

cs151的一个lab，我当年也做过一遍不过不记得了。

差不多就是给定一些演员，还有电影及其参演演员，如果两个演员共演过一部电影就算有联系，某人的Kevin Bacon数代表了这个人与kevin bacon通过电影共演联系起来的最小数量。Kevin Bacon自己的KB数为0，如果任何和他共演过电影的人的KB数为1。

对于这种现实世界的抽象问题，一般来讲都是用图，而既然题目只要求得KB数，不要求将相关的电影表示出来，那么我们可以做一个简单的图：顶点为不同的演员，只要他们共演过任何一部电影，那么他们之间就添加一道边，因为不需要重复，所以我们可以用Map+Set的组合。

```javascript
// add to graph
let actors = new Map();
for (m in movies) {
  for (let i=0;i<m.length;i++) {
    for (let j=i+1;j<m.length;j++) {
      if (!actors.has(m[i])) {
        actors.set(m[i], {kb:-1, edges:new Set());
      }
      if (!actors.has(m[j])) {
        actors.set(m[j], {kb:-1, edges:new Set());
      }
      actors.get(m[i]).add(m[j]);
      actors.get(m[j]).add(m[i]);
    }
  }
}
actors.get("Kevin Bacon").kb = 0;
```

然后是确定算法。一般来讲对于图不是用DFS就是用BFS，当然你也可以用Dijkstra，但因为我们图中的边没有赋值，其实直接在BFS上改动就可以了。至于为什么不用DFS，是因为DFS找到的可能不是最小距离，而只要保证BFS中的距离为`x`的节点都是最小值，那么向外延伸1一个边、距离为`x+1`的也是最小值。

如果只是搜索某个特定人的KB数，那么最坏情况下可能要遍历两次边和一次节点，所以假设节点数为n边数为m那么时间复杂度是O(m+n)。但同样地，遍历一遍将所有人的KB数记录在数据结构中的复杂度也是O(m+n)。这个时候应该和面试官问清楚需要的是那种，这里我们先实现储存所有人KB数的方案。

```javascript
let queue = [];
let start = actors.get("Kevin Bacon");
if（!start) throw "No actor named Kevin Bacon";
queue.push(start);
while(queue) {
  let cur = queue.shift();
  for (let a of cur.edges.entries()) {
    if (actors.get(a).kb==-1) {
      // unvisited
      actors.get(a).kb=cur.kb + 1;
      queue.push(actors.get(a));
    }
  }
}
```

## [二叉树中的硬币分配](https://leetcode.com/problems/distribute-coins-in-binary-tree/)

> Given the `root` of a binary tree with `N` nodes, each `node` in the tree has `node.val` coins, and there are `N` coins total.
>
> In one move, we may choose two adjacent nodes and move one coin from one node to another. (The move may be from parent to child, or from child to parent.)
>
> Return the number of moves required to make every node have exactly one coin.

这题本来应该算在二叉树里的，但它用树的方式思考不太好破题，需要一些图的概念，所以放到这里来了。

先说用树思考的部分：老样子，全局变量计数器，递归分析每个子树。

如果用动态的角度来考虑，即将子节点流出的硬币个数传递给父节点，然后增加计数，你会发现这题没法解，因为硬币的流动可以双向进行；所以我们考虑对于每一个节点，将「当前状态」与「终止状态」进行对比，从而得出需要移动的次数。

首先，从图的角度来看，很容易想到统计每个节点的inflow和outflow，这里将每个节点的「权重」定义为需要从「以这个节点为根的子树」中流出的硬币的总数量，如果流出硬币该「权重」为负。因为根据题目条件每一个输入的二叉树都肯定能够达到最终态，所以可以计算该权重：左右子节点流入的硬币数量加节点本身持有的硬币，减去自己应该留的那个硬币。

既然计算了权重，那就好办了。因为二叉树递归不能处理该节点的父节点，那么就应该在这个stack上处理掉它在两个子树之间的交换次数即可。答案是左右权重各自的绝对值之和，因为不论流入流出都算一次移动。

```python
class Solution(object):
    count = 0
    def distributeCoins(self, root):
        self.helper(root)
        return self.count
        
    def helper(self,root):
        # return the weight of the node (how many coins in this **tree**)
        if not root:
            return 0
        left = self.helper(root.left)
        right = self.helper(root.right)
        # count how many counts we need to move in/out in subtrees
        self.count += abs(left) + abs(right)
        # count the total weight of this whole tree
        weight = root.val - 1 + left + right
        return weight
```

