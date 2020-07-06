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
        actors[m[i]] = {kb:-1, edges:new Set()};
      }
      if (!actors.has(m[j])) {
        actors[m[j]] = {kb:-1, edges:new Set()};
      }
      actors[m[i]].add(m[j]);
      actors[m[j]].add(m[i]);
    }
  }
}
actors["Kevin Bacon"].kb = 0;
```

然后是确定算法。一般来讲对于图不是用DFS就是用BFS，当然你也可以用Dijkstra，但因为我们图中的边没有赋值，其实直接在BFS上改动就可以了。至于为什么不用DFS，是因为DFS找到的可能不是最小距离，而只要保证BFS中的距离为`x`的节点都是最小值，那么向外延伸1一个边、距离为`x+1`的也是最小值。

如果只是搜索某个特定人的KB数，那么最坏情况下可能要遍历两次边和一次节点，所以假设节点数为n边数为m那么时间复杂度是O(m+n)。但同样地，遍历一遍将所有人的KB数记录在数据结构中的复杂度也是O(m+n)。这个时候应该和面试官问清楚需要的是那种，这里我们先实现储存所有人KB数的方案。

```javascript
let queue = [];
let start = actors["Kevin Bacon"];
if（!start) throw "No actor named Kevin Bacon";
queue.push(start);
while(queue) {
  let cur = queue.shift();
  for (let a of cur.edges.entries()) {
    if (actors[a].kb==-1) {
      // unvisited
      actors[a].kb=cur.kb + 1;
      queue.push(actors[a]);
    }
  }
}
```

