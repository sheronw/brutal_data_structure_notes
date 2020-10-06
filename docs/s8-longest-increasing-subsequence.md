# 最长递增子序列相关题目解析

## LC674. Longest Continuous Increasing Subsequence

最简单的：一维 DP，只看它前面。O(n)时间，O(n)空间。

```python
def findLengthOfLCIS(nums: List[int]) -> int:
    n = len(nums)
    if n==0:
        return 0
    memo = [0 for i in range(n)]
    memo[0] = 1
    for i in range(1,n):
        if nums[i]>nums[i-1]:
            memo[i] = memo[i-1] + 1
        else:
            memo[i] = 1
    return max(memo)
```

DP 数组可以简化为一个整数来保存。O(n)时间，O(1)空间。

```python
def findLengthOfLCIS(nums: List[int]) -> int:
    n = len(nums)
    if n==0:
        return 0
    memo = 1
    res = memo
    for i in range(1,n):
        if nums[i]>nums[i-1]:
            memo = memo + 1
        else:
            memo = 1
        res = max(res,memo)
    return res
```

滑动窗口（LC 官方解）：
所有的 increasing subsequence 不可能重叠，所以从头遍历一遍，保存每个 seq 的起点，如果有降序就重新清空，同时记录最大长度。

```python
def findLengthOfLCIS(nums: List[int]) -> int:
    n = len(nums)
    if n==0:
        return 0
    start = 0
    res = 1
    for i in range(1,n):
        if nums[i]<=nums[i-1]:
            start = i
        res = max(res,i-start+1)
    return res
```

## LC300. Longest Increasing Subsequence

最简单的，一维 DP 求最大值。时间 O(n<sup>2</sup>)，空间 O(n)。

```python
def lengthOfLIS(nums: List[int]) -> int:
    n = len(nums)
    if n==0:
        return 0
    memo = [1 for i in range(n)]
    for i in range(1,n):
        for j in range(i):
            if nums[j]<nums[i]:
                memo[i] = max(memo[i],memo[j]+1)
    return max(memo)
```

找最大值可以通过二分查找来优化。这个感觉不怎么像 DP，先记录下来。O(nlogn)时间，O(n)空间。

```python
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        n = len(nums)
        if n==0:
            return 0
        memo = [0 for i in range(n)]
        length = 0
        for i in range(n):
            # search insertion position
            j = self.bs(0,length,memo, nums[i])
            memo[j] = nums[i]
            if j==length:
                length += 1
        return length

    def bs(self,l,r,nums, target):
        # return inserted position
        # find the first element that is greater or equal than this one
        # upper bound, >= target so use <
        while l<r:
            mid = l + (r-l) // 2
            if nums[mid] < target:
                l = mid + 1
            else:
                r = mid
        return l
```

## LC673. Number of Longest Increasing Subsequence

在上一题的基础上对 count 进行计数。

```python
def findNumberOfLIS(self, nums: List[int]) -> int:
    n = len(nums)
    if n<=1:
        return n
    length = [1 for i in range(n)]
    count = [1 for i in range(n)]
    for i in range(n):
        for j in range(i):
            if nums[j]<nums[i]:
                if length[j] == length[i] - 1:
                    # if we already have that, update count
                    count[i] += count[j]
                if length[j] + 1 > length[i]:
                    # if we find a new max length
                    count[i] = count[j]
                    length[i] = max(length[i],length[j]+1)

    longest = max(length)
    res = 0
    for i in range(n):
        if length[i] == longest:
            res += count[i]
    return res
```

好像还能用线段树。等学了数据结构之后再补。

## LC354. Russian Doll Envelopes

听说用单纯 DP python 会超时，所以实现的是二分搜索的那个方法。

也是 Longest Increasing Subsequence 的变种，但首先需要排序。

直觉上看，排序应该是 w 从大到小，然后 h 也从大到小。但这样的话会遇到以下情况：

```python
arr = [[5,4],[6,4],[6,7],[2,3]] # 原数组
sorted = [[2,3],[5,4],[6,4],[6,7]] # 正常排序
memo = [None,None,None,None] # 初始memo
memo = [[2,3]] # 插入[2,3]
memo = [[2,3],[5,4]] # 插入[5,4]
memo = [[2,3],[6,4]] # 插入[6,4]
memo = [[2,3],[6,7]] # 插入[6,7]
```

如果按照这样计算的话，得出 2，但答案应该是 3（`[2,3]->[5,4]->[6,7]`）。对于 w 相同的情况下，如果 h 也从大到小，可能会把之前的 w 更小的情况给覆盖掉，从而无法再往里添加新的 w 相同却更大的信封。

所以如果 w 相同的话，应该让 h 从小到大排。这样更小的 w 上一个信封不会因为当前 w 的这个 h 比较小的信封塞不进去而惨遭覆盖。如果相同的 w 里还有更小的信封，只要覆盖就行了，反正 w 相同的情况下只能放一个信封，而这个信封应该是 w 相同的里面 h 最小的那个。

至于二分搜索中的比较，使用的 comparator 就是「能不能放进去」这个准则了。和排序所用的 comparator 不一样。

```python
class Solution:
    def maxEnvelopes(self, envelopes: List[List[int]]) -> int:
        envelopes.sort(key=lambda x:(x[0],-x[1]))
        n = len(envelopes)
        if n==0:
            return 0
        if n==1:
            return 1
        length = 0
        memo = [None for i in range(n)]
        for i in range(n):
            j = self.bs(0,length,memo,envelopes[i])
            if length==j:
                length += 1
            memo[j] = envelopes[i]
        return length

    def bs(self,l,r,nums,target):
        while l<r:
            mid = l + (r-l)//2
            if nums[mid][0] < target[0] and nums[mid][1] < target[1]:
                l = mid + 1
            else:
                r = mid
        return l
```
