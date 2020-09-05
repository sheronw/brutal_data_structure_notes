# Disjoint Set

Disjoint Set，也叫 Union Find Data Structure，中文叫并查集，多用在替代 DFS 上。

## Data Structure

这种数据结构一般用来把树搞到一起，或者找一个节点的根节点。都叫 Union Find 了，所以实现时需要两个方法，Union 和 Find。

先说 Find，也就是说找某个节点的根节点。一般来讲我们就是一个一个往上找，需要 O(n)的时间复杂度。然而，这个 Find 呢在找到根节点之后，会顺便把我们要找的原本的节点变成根节点的 child，hmmm 根节点还是它，没毛病，但这样下次找它的根节点的时候，就变成了 O(1)的复杂度了。这样操作很多次之后，Find 方法就变成了平均 O(1)的时间复杂度。

再说 Union，这里对树有一个定义，叫 Rank，指的是这个树的混乱度，也就是说直觉上看从叶节点到根节点的距离（具体我也不太好说，大概这种感觉吧）。我们 Union 的时候会把两个节点对应的树都搞到一起。具体怎么搞呢，首先找到他们的根节点（用 Find 方法），然后看这两个根节点的 Rank，挑出那个 Rank 比较大的，然后把小树并到大树根节点上去（如果把大树并到小树上，混乱度就变更大了）。如果两棵树一样大那么并到哪里都没问题，只是需要手动将新树的混乱度增加。如果没有 Find 过，那么 Union 需要 O(n)的复杂度，如果有的话，那么可以达到 O(1)。

有一个公式叫 Inverse-Ackermann function 可以用来计算它的时间复杂度。

```python
class DisjointSet:
    def __init__(self, n):
        self.parent = [i for i in range(n)]
        self.rank = [0 for i in range(n)]

    def find(self, x):
        if x != self.parent[x]:
            self.parent[x] = self.find(self.aprent[x])
        return self.parent[x]

    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx==ry:
            return
        if self.rank[rx] > self.rank[ry]:
            self.parent[ry] = rx
        elif self.rank[rx] < self.rank[ry]:
            self.parent[rx] = ry
        else:
            self.parent[ry] = rx
            self.rank[rx] += 1
```

## Number of Islands II

### Description

岛数是道经典的 DFS 题，也能用 Disjoint Set 做。开局读进所有的`1`，把它们作为一个独立的 tree，然后再和 DFS 的操作相同，遍历一遍如果旁边`1`的旁边也是`1`就进行 Union。时间复杂度不太容易计算，因为有一个专门的 disjoint set 的公式，可以近似于类似 DFS 的 O(n)。

而当输入过大，无法将全部数据放入内存进行搜索时，可以边读取输入边分析，也就是说用*岛数 II*这道题来解决。

岛数二因为是锁题，所以就不提供 LC 号码了，这里直接贴题目描述：

> A 2d grid map of **m** rows and **n** columns is initially filled with water. We may perform an `addLand` operation which turns the water at position `(row, col)` into a land. Given a list of positions to operate, **count the number of islands after each addLand operation**. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

**Example:**

```
Input:
m = 3, n = 3, positions = [[0,0], [0,1], [1,2], [2,1]]

Output:
[1,1,2,3]
```

**Explanation:**

Initially, the 2d gridgridis filled with water. (Assume 0 represents water and 1 represents land).

```
0 0 0
0 0 0
0 0 0
```

Operation #1: addLand(0, 0) turns the water at grid[0][0] into a land.

```
1 0 0
0 0 0 Number of islands = 1
0 0 0
```

Operation #2: addLand(0, 1) turns the water at grid[0][1] into a land.

```
1 1 0
0 0 0 Number of islands = 1
0 0 0
```

Operation #3: addLand(1, 2) turns the water at grid[1][2] into a land.

```
1 1 0
0 0 1 Number of islands = 2
0 0 0
```

Operation #4: addLand(2, 1) turns the water at grid[2][1] into a land.

```
1 1 0
0 0 1 Number of islands = 3
0 1 0
```

**Follow up:**

Can you do it in time complexity O(k log mn), where k is the length of thepositions?

### Analysis

这道题中要求的是数出所有的岛数，所以在已有的 Disjoint Set 里面，我们需要添加一个计数器来记录一共有多少个岛。开局陆地为空，所以将 rank 和 parent 都全部标记为不存在。每加块新陆地，就把它自己当成一个独立的新 tree（计数器加一）然后再与旁边的 tree（如果有的话）union 一下就好了，每 union 成功（root 不相同时）计数器就减一。这样处理完每次操作后得到的结果就是当前的岛数。

```python
class Solution:
    class DisjointSet:
        def __init__(self, n):
            # set everything to empty
            self.parent = [-1 for i in range(n)]
            self.rank = [0 for i in range(n)]
            self.count = 0 # 计数器

        def find(self, x):
            if x != self.parent[x]:
                self.parent[x] = self.find(self.aprent[x])
            return self.parent[x]

        def union(self, x, y):
            rx, ry = self.find(x), self.find(y)
            if rx==ry:
                return
            if self.rank[rx] > self.rank[ry]:
                self.parent[ry] = rx
            elif self.rank[rx] < self.rank[ry]:
                self.parent[rx] = ry
            else:
                self.parent[ry] = rx
                self.rank[rx] += 1
            self.count -= 1

        def addParent(self, x):
            self.parent[x] = x
            count += 1

        def isValid(x):
            return self.parent[x]!=-1

        def getCount():
            return self.count

    def numberOfIslands2(m, n, positions):
        res = []
        ds = DisjointSet(m*n)
        for (r,c) in positions:
            # add it to maps
            id = r * n + c
            ds.addParent(id)

            # union in four directions
            for (nr,nc) in [(r,c+1),(r,c-1),(r+1,c),(r-1,c)]:
                if nr>=0 and nr<m and nc>=0 and nc<n:
                    nid = nr * n + c
                    if ds.isValid(nid):
                        ds.union(id,nid)

            # update answer
            res.append(ds.getCount())
        return res
```

## 参考资料

> [花花酱 Disjoint-set/Union-find Forest - 刷题找工作 SP1](https://youtu.be/VJnUwsE4fWA)

> [Number of Islands II - aaronice](https://aaronice.gitbook.io/lintcode/union_find/number_of_islands_ii)
