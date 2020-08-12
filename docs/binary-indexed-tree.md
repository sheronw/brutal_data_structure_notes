# Binary Indexed Tree

## Data Structure

这个数据结构比较刁钻，专门用来处理「数组特定范围和」及其类似问题。也叫 Fenwick Tree。

如果想要计算数组特定范围和，直接遍历一遍累积即可，时间复杂度为 O(n)；或者如果这个数组稳定不变，可以先 O(n) 遍历一边得出从第一个元素到某元素之间的和，然后每次需要的时候相减起始点即可，这样是 O(1)。但如果用 BIT 的话，可以做到更改元素和获得元素和的时间复杂度都为 O(logn)。

这个树的本体是一个线性的数组，在构建和获取值时有着不同的变形，可视化参考[这个网页](https://visualgo.net/en/fenwicktree)。因为构建要求第一个元素从 1 而不是 0 开始，所以构建树用的数组长度比原数组多 1。

先说构建/更新时树的形状。假设子节点的 index 为 i，那么父节点的座标为子节点加子节点二进制的最低位（lowbit）。一旦子节点的和有更新，那么他的所有父节点都要进行相应的更新。最低位用大白话来说就是，二进制从右往左数，找到第一个 1，然后把这个 1 前面的所有 0 算上合在一起是多少。比如 1010100 这个数，从左往右数到的第一个 1 是第三位，所以最低位应该是 100，也就是 4。

而获取值的时候，顺序是从当前节点向下，减子节点二进制的最低位，这个路径的和则为从第一个元素到这个元素的和。

## Mutable Range Sum Query

[LC307](https://leetcode.com/problems/range-sum-query-mutable/)就是这个数据结构的直接应用。

```python
class NumArray(object):

    def __init__(self, nums):
        """
        :type nums: List[int]
        """
        self.bit = [0] * (1 + len(nums))
        self.nums = [0] * len(nums)
        for i,val in enumerate(nums):
            self.update(i,val)


    def update(self, i, val):
        """
        :type i: int
        :type val: int
        :rtype: None
        """
        delta = val-self.nums[i]
        treeIndex = i + 1
        while treeIndex<len(self.bit):
            self.bit[treeIndex] += delta
            treeIndex += self.lowbit(treeIndex)
        self.nums[i] = val


    def getFromTree(self,treeIndex):
        res = 0
        while treeIndex>0:
            res += self.bit[treeIndex]
            treeIndex -= self.lowbit(treeIndex)
        return res


    def sumRange(self, i, j):
        """
        :type i: int
        :type j: int
        :rtype: int
        """
        iTreeIndex = i + 1
        jTreeIndex = j + 1
        return self.getFromTree(jTreeIndex) - self.getFromTree(iTreeIndex-1)

    def lowbit(self,n):
        return n & (-n)
```

## [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/)

暴力解法就是对每个元素进行遍历，这样时间复杂度是 O(n<sup>2</sup>)，LC 可能会超时。

既然这题归在 binary indexed tree 这里，说明就能用它解，那么假设已经有了 BIT 这个数据结构（含有 get 和 update 这两个方法），要如何转换呢？

BIT 储存的是某个数组从**开头**到某个数的总和，而本题求的是本元素**右边**比它小的各数，但只需要把数组倒置就可以解决这个问题（题目没明确说是否有重复数据，如果有的话就不能直接用数组长度相减，所以我们就这么做）。

但即使题目转换成了求本元素左边比它小的各数也没有那么简单——理想状态下里面应该储存从开头到某个数之间的*某种*频率，但我们不知道它左边的哪个数比它小，所以无法通过 BIT 外加 O(1) 的运算得到结果。似乎陷入僵局。

那不如换个角度考虑，当我们初始化 BIT 的时候，肯定是将反转的数组从左到右逐个处理，而在将这个元素 update 进去之前，**彼时 BIT 中储存的信息都是关于该元素左边的**。既然掌握了这个关键线索，用 BIT 储存频率时保持原数组的顺序就变得毫无必要。

如果 BIT 的顺序没必要与原数组顺序相同，又因为求的是比某数小的所有数的和，那么很容易想到根据元素的大小来进行排序，这样「求 BIT 中比这个数字小的数字出现的频率」这个问题就正好变成了「求当时 BIT 中从**开头**到储存这个数字频率的那位的频率和」。

现在我们有了这样一个想法，因为一共有 n 个元素， BIT 的两个操作都是 O(logn)，因此总的时间复杂度为 O（nlogn）：

```
对于逆序数组中的每一个数:
    用 get 方法获取将当前 BIT 中比它小的数字的频率总和，并添加到结果数组中
    用 update 方法把 BIT 中这个数值的频率增加 1
```

在这之前，需要一个 map 来储存每个数值对应的在 BIT 中的 index（也就是说排第几）。用语言内置的实现进行去重、排序、建 map 即可，比如 python 可以将数组元素添加到 set 中然后再进行排序，map 用 dictionary。这么一套下来时间复杂度是 O(nlogn)。

```python
class Solution(object):
    def countSmaller(self, nums):
        # create a map that (key,value) = (value_of_item, seq)
        dic = {}
        noDuplicate = sorted(set(nums))
        for i,v in enumerate(noDuplicate):
            dic[v] = i+1
        # init res and BIT
        res = []
        bit = [0] * (1 + len(noDuplicate))
        # deal with each item in reversed list and add result to result list
        for i in range(len(nums)-1,-1,-1):
            cur = nums[i]
            # get the result from BIT
            count = 0
            treeIndex = dic[cur] -1
            while treeIndex>0:
                count += bit[treeIndex]
                treeIndex -= self.lowbit(treeIndex)
            res.append(count)
            # update data to BIT
            treeIndex = dic[cur]
            while treeIndex<len(bit):
                bit[treeIndex] += 1
                treeIndex += self.lowbit(treeIndex)
        # reverse result list
        res.reverse()
        return res

    def lowbit(self,n):
        return n & (-n)
```
