# 二分搜索相关题目解析

之前没做过这个类型的题目，感觉小看二分查找这个知识点了，坑还是蛮多的。

以下题目列表是用的是 [花花酱](https://zxi.mytechroad.com/blog/leetcode-problem-categories/) 的分类。

## 基本的二分查找

可以用递归也可以用循环遍历，一般来讲做题时用的比较多的是下面这个：

```python
def binarySearch(arr,target):
    l, r = 0, len(arr) # 前闭后开原则
    while l < r: # 搜索至搜索区间为空
        mid = l + (r - l) // 2 # 以闭区间起点为中心进行调整防止溢出
        if arr[mid] < target: # 调整区间，确保区间左侧元素都小于区间内元素
            l = mid + 1
        else: # 调整区间，确保区间右侧元素都大于或等于区间内元素
            r = mid
    return l # 当搜索区间为空时，l与r相等
```

当然，当我们在说二分搜索的时候，针对不同的题目要求，可能指的是这几种情况中的一个：

- 寻找第一个大于或等于 target 的元素
- 寻找最后一个不大于或等于 target 的元素
- 寻找第一个大于 target 的元素
- 寻找最后一个不大于 target 的元素

这四种说法等价于：

- `>=` target, upper bound
- `>=` target, lower bound
- `>` target, upper bound
- `>` target, lower bound

上面的代码中，因为是确保区间左侧元素都小于区间内元素，所以表达的是第一种情况。如果想要寻找第一个大于 target 的元素，将`if`的条件换成`<=`，即确保区间左侧元素都大于或等于区间内元素。如果想要求 lower bound，虽然也可以前开后闭再写一次，但最好是用`upper bound-1`，这样背一个模板就可以搞定辣。

## Upper Bound 类题目

### LC704 Binary Search

本题要求返回 target 的座标，不存在则返回-1。

因为模板找的是第一个大于或等于 target 的元素，所以如果 target 不存在，就会返回错误结果。所以可以再加一个判断条件，如果等于目标就返回当前座标，或者在最后查是否边界溢出以及等于 target。最后返回-1 即可。

```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        l, r = 0, len(nums) # 前闭后开原则
        while l < r: # 搜索至搜索区间为空
            mid = l + (r - l) // 2 # 以闭区间起点为中心进行调整防止溢出
            if nums[mid] < target: # 调整区间，确保区间左侧元素都小于区间内元素
                l = mid + 1
            elif nums[mid]==target:
                return mid
            else: # 调整区间，确保区间右侧元素都大于或等于区间内元素
                r = mid
        return -1 # 当搜索区间为空时，l与r相等
```

### LC35 Search Insert Position

本题要求返回 target 座标，不存在就返回 target 应该插入的座标。

显然返回的是第一个大于或等于 target 的元素座标，直接使用模板即可。（不过判断等于目标并返回，也不会错）

### LC34 Find First and Last Position of Element in Sorted Array

本题要求返回 target 的座标范围。比如`[1,1,1]`返回`[0,2]`。如果目标不存在就返回`[-1,-1]`.

如果想要一口气同时找到两个端点其实有点麻烦，所以不如找两次，反正还是`O(logn)`。

第一个座标应该是第一个大于或等于 target 的元素，如果不存在就返回-1，和 LC704 相同。第二个座标应该是最后一个不大于 target 的元素，如果不存在同样返回-1。题目要求的区间不是前闭后开，所以要在`<=`模板的基础上-1。

一个需要注意的点是不能再判断条件中发现等于 target 就返回，因为需要找的是边界值。需要放到循环结束后去判断是否越界或不存在。

也可以找到其中一个端点然后向后或者向前找，但最坏情况下时间复杂度为 O(n)，不推荐。

```python
class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        # 搜索第一个大于等于目标的元素
        l, r = 0, len(nums) # 前闭后开原则
        while l < r: # 搜索至搜索区间为空
            mid = l + (r - l) // 2 # 以闭区间起点为中心进行调整防止溢出
            if nums[mid] < target: # 调整区间，确保区间左侧元素都小于区间内元素
                l = mid + 1
            else: # 调整区间，确保区间右侧元素都大于或等于区间内元素
                r = mid
        p1 = l
        if l<0 or l>=len(nums) or nums[l]!=target:
            p1 = -1

        # 搜索最后一个不大于目标的元素
        l, r = 0, len(nums) # 前闭后开原则
        while l < r: # 搜索至搜索区间为空
            mid = l + (r - l) // 2 # 以闭区间起点为中心进行调整防止溢出
            if nums[mid] <= target: # 调整区间，确保区间左侧元素都小于区间内元素
                l = mid + 1
            else: # 调整区间，确保区间右侧元素都大于或等于区间内元素
                r = mid
        l -= 1
        p2 = l # 当搜索区间为空时，l与r相等
        if l<0 or l>=len(nums) or nums[l]!=target:
            p2 = -1
        return [p1,p2]
```

### LC981 Time Based Key-Value Store

带点 OOP 元素的二分查找。set 与 LC35 相同，get 则是找不大于 target 的最后一个元素。不过要判断字典中是否存在该元素等一系列条件。

### LC69 Sqrt(x)

第一个需要「猜」的题目，虽然之后会出现很多。基本上就是从 0 到 x 中不断地选中点，判断该值的平方与 x 的大小关系。

这题要的是开根号结果的 floor，也就是说求的是最后一个不大于目标的元素。

不过如果按照模板实现的话，因为我们将开区间范围设定为 x，所以需要将 1（它的根号还是它自己，取中点永远得不到）。

```python
class Solution(object):
    def mySqrt(self, x):
        """
        :type x: int
        :rtype: int
        """
        if x<2:
            return x

        l = 1
        r = x
        while l<r:
            mid = l+(r-l)//2
            if mid*mid<x:
                l = mid + 1
            elif mid*mid>x:
                r = mid
            else:
                return mid
        return l-1
```

## 猜答案类题目

二分搜索还可以用在一些需要在一个有序范围内差找答案，但逐一遍历效率太低时猜答案，比如刚刚提到过的开根号。

对于这种题目我们需要确定答案的边界（最大值和最小值），然后写出假设答案已知，计算限制条件的算法。最后用这个算法算出“`arr[mid]`”然后比较逐一缩小范围。

### LC875 Koko Eating Bananas

给定一个数组，每次操作至多将单个元素减少 `k`，如果 `k` 大于元素本身则视为一次操作，清空该元素。要求能够在 `H` 次操作后清空数组的 `k` 的最小值。

`k` 的最小值为 1，最大值为数组元素最大值`max(arr)`。

假设已知 `k`，那么如何计算出所需的 `H` 呢？遍历整个数组求出每一个元素需要几次操作即可（注意边界条件），时间复杂度是 `O(n)`。

老样子，我们使用那个前闭后开模板。

想像针对每一个可能的 `k` 计算出的 `H` 的数组，你会发现这个很有可能是递减的，因此需要在传统的二分搜索基础上做一些修改。我们想要得到的最优 `k` 是这个递减的数组里第一个小于或等于 `H` 的元素，所以要保证区间左侧元素大于`Ｈ`，`if`条件为大于。

```python
class Solution(object):
    def minEatingSpeed(self, piles, H):
        l,r = 1, max(piles)+1
        while l<r:
            mid = l+(r-l)//2
            count = 0
            for p in piles:
                if p%mid!=0:
                    count+=1
                count += p//mid
            if count<=H:
                r = mid
            else:
                l = mid + 1
        return l
```

### LC1011 Capacity To Ship Packages Within D Days

给定一个数组，每次操作清空一个或多个元素，但有一个固定上限`w`，并且要按序操作，不能跳过。要求能够在`D`次操作后清空数组的`w`最小值。

`w`的最小值为 1，最大值为数组元素的和`sum(arr)`。

假设已知`w`，如何计算`D`？显然需要`O(n)`的复杂度遍历每一个元素，记录当前已经添加的货物和所需的操作次数。边界条件仍然注意最后一次操作的所有值可能不到`w`。

针对`w`计算出的`D`仍然是递减的，类似于上一道题，`if`条件为大于。

```python
class Solution(object):
    def shipWithinDays(self, weights, D):
        m = max(weights)
        l,r = 1, sum(weights)
        while l<r:
            mid = l + (r-l)//2
            if mid<m:
                # cannot take this
                l = mid + 1
                continue
            days = 0
            cur = 0
            # calculate if capacity is mid then we need how many days
            for w in weights:
                if w+cur<mid:
                    cur += w
                elif w+cur>mid:
                    cur = w
                    days += 1
                else:
                    days += 1
                    cur = 0
            if cur!=0:
                days += 1

            if days<=D:
                r = mid
            else:
                l = mid + 1
        return l
```

## kth element 类题目

获取第 k 小的元素，类似于猜答案，需要获取所有元素的最大值和最小值作为边界条件，然后取中点，算出有多少个元素比这个值小或者等于。找出第一个大于等于 k 的点。

### LC378 Kth Smallest Element in a Sorted Matrix

Sorted Matrix 指的是一个矩阵，每行从左到右排序，每列上到下排序。给定一个边长为 n 的矩阵求第 k 小的元素。

最小值是左上角的元素，最大值是右下角的元素。

计算多少个元素比这个值小（或者等于）的方法，可以每行遍历求解，但很慢，接近于强行排序`O(nlogn)`。

我们从最后一行开始搜索，如果当前值比目标还大，那么这行后面都不用找了，直接找上一行这个位置；如果比目标值小，那么就说明这一列中它前面的值都比它小，将这些值的个数加到计数器里面，然后搜索下一列这个位置。因为行和列都有边界条件限制，最多只能增减 n 次，而每次循环都会增减一次行或列，所以可以将时间复杂度降到`O(n)`。

总的时间复杂度是`O(nlogn^2)`。

```python
class Solution(object):
    def kthSmallest(self, matrix, k):
        if len(matrix)==0:
            return -1
        n = len(matrix)
        l,r = matrix[0][0], matrix[n-1][n-1]+1
        while l<r:
            mid = l + (r - l) // 2
            res = self.countsmaller(matrix,mid)
            if res < k:
                l = mid + 1
            else:
                r = mid
        return l

    def countsmaller(self,matrix,mid):
        res = 0
        for i in range(len(matrix)):
            j = 0
            while j<len(matrix) and matrix[i][j]<=mid:
                j += 1
                res += 1
        return res
```

### LC668 Kth Smallest Number in Multiplication Table

给定乘法表的行列`m`和`n`，求乘法表中第 k 小的元素。

最小值就是 1 啦，最大值是`m*n`。

如何计算多少个元素比中点值小或等于呢？这次的优势是，在`m`行中，第`i`行的元素其实就是`i`乘以`[1,2,3,...,n]`，那么我们只需要看看这个中点值除以 `m`的值是多少就知道了。这样数完`m`行的时间复杂度就是`O(m)`。

总的时间复杂度是`O(mlogm*n)`。

```python
class Solution(object):
    def findKthNumber(self, m, n, k):
        l, r = 1, m*n+1
        while l<r:
            mid = l + (r-l)//2
            # count how many elements are smaller or equal than mid
            count = 0
            for i in range(1,min(m+1,mid+1)):
                # scan row by row, from 1 to m
                t = mid//i
                count += min(t,n)
                if count >= k:
                    break
            # binary search check
            if count>=k:
                r = mid
            else:
                l = mid + 1
        return l
```

### LC719 Find K-th Smallest Pair Distance

给定一个数组，求数组元素两两组合的差值中第 k 小的元素（仅考虑非负情况）。

最小值就是 0，最大值就是数组中的最大值与最小值的差。

如何计算多少个元素比中点小或者等于呢？首先在这个题里面原数组顺序不重要所以可以先排个序再说。然后是用动态规划。

这里 OPT(i)是指第`i`个元素后面的所有元素减前面（包括`i`）的差值小于等于中点的个数。而 OPT(i)=OPT(i-1)+(j-i-1)，因为 OPT(i-1)已经算好，只需要考虑后面元素减`i`的情况，公式中的`j`是减去`nums[i]`的结果大于中点的 j 的第一个坐标。结果应该是 OPT(n-2)，因为最后一个元素后面没有别的元素可以减了，所以取的是倒数第二个。因为只用到 OPT(i-1)，所以常数空间足够。

时间复杂度是`O(nlog(max(arr)-min(arr)))`和`O(nlogn)`里面比较大的那个。

```python
class Solution:
    def smallestDistancePair(self, nums: List[int], k: int) -> int:
        nums.sort()
        n = len(nums)
        # we know len(nums)>=2
        l, r = 0, nums[-1]-nums[0]
        while l<=r:
            mid = l + (r-l) // 2
            # calculate how many elements is less or equal than mid
            res = 0
            j = 0
            for i in range(0,n):
                # skip until [j]-[i] is greater than mid
                while j<n and nums[j]-nums[i] <= mid:
                    j += 1
                # update answer
                res += j-i-1
            if res >= k:
                r = mid - 1
            else:
                l = mid + 1
        return l
```

当然这道题还可以用桶排序，也是因为最大值最小值已知。建桶 O(n<sup>2</sup>)读取 O(k)。

### LC786 K-th Smallest Prime Fraction

类似上一题，给定一个数组，求数组两两组合的比值中第 k 小的元素（仅考虑小于 1 的数）。

那这最小值就是 0 最大值就是 1。而且因为是小数不怕溢出（Python 倒是没事啦）也不用取整，还可以写`mid=(l+r)/2`的形式。不过因为返回的是两个数组中的数而不是小数，所以需要储存当前的两个数并不断更新。

还是那个样子，排个序再说。排序之后构建一个类似于乘法表的矩阵，每行对分子相同每列分母相同这样。这个数组只有一半，而且从左到右递减，从下到上递减。

求有多少个元素小于或等于，可以参考 sorted matrix 的做法，根据条件判断跳过这一行或这一列，最后也是 `O(n)`。

```python
class Solution:
    def kthSmallestPrimeFraction(self, A: List[int], K: int) -> List[int]:
        n = len(A)
        l, r = 0.0, 1.0
        while l<r:
            mid = (l+r)/2
            # calculate how many fractions are >= mid
            res = 0
            p, q, f = 0, 0, 0.0
            j = 1
            for i in range(n-1):
                # skip numbers that are > mid
                while j<n and A[i] > mid*A[j]:
                    j += 1
                # out of boundary
                if j>=n:
                    break
                res += (n-j)
                # update res
                if A[i]/A[j] > f:
                    p = i
                    q = j
                    f = A[i]/A[j]
            if res == K:
                return [A[p],A[q]]
            elif res > K:
                # number is large
                r = mid
            else:
                l = mid
        return []
```

## 两个数组中的中位数

给定两个排好序的数组，要求返回两个数组合在一起的中位数。

暴力算法就是归并取重点，但时间复杂度为`O(n)`。二分搜索可以降到`O(logn)`。

因为中位数肯定是所有的数里面中间的那个，所以只要从其中一个数组中取出些中位数左边的数，那么另外一个数组中需要取出的量就是确定的，那就挑一个比较短的数组`nums1`做二分搜索呗。

当两个数组总长为奇数时，取中间；为偶数时，要取两个数的平均值。那么我们就需要获取至少两个中位数的坐标。这两个从哪里取呢？假设从`nums1`取出来的最后两个数是 A 和 B，`nums2`取出来的最后两个数是 C 和 D，而他们都是升序的，那么左边的中位数就是 A 和 C 里面比较大的（排第二），右边的中位数就是 B 和 D 里面比较小的。

如何在`O(1)`时间内判定目标在左还是右呢？

举个栗子:

```python
# 原始的两个数组
arr1 = [1,3,5,7]
arr2 = [2,4,6]
# 合并之后的数组，中位数为4
arr = [1,2,3,4,5,6,7]
```

在上面的栗子中，我们可以看到`arr1`中有两个元素`[1,3]`被分到了中位数左侧，两个元素`[5,7]`被分到了中位数右侧；`arr2`中有一个元素`[2]`被分到了中位数左侧，一个元素`[6]`被分到了中位数右侧。

`arr1`的 5 被分到了右侧，因为 5 肯定比 3 大，但`arr2`因为要也要并进来，说明 5 肯定也要比`arr2`中的左侧任何数要大，也就是说这两个数组中，分到右侧的数中最小的，是需要比另外一个数组中分到左侧的数都要大（或者等于）的。

那么我们可以根据这个特点进行二分搜索，直到找到第一个数作为左侧尾数，使得它大于或等于另外一个数组中需要取出的左侧尾数。

思路就是这样，但各种边界条件很恶心，还要分类讨论奇数偶数长度的中位数，需要多加复习。

```python
class Solution(object):
    def findMedianSortedArrays(self, nums1, nums2):
        k = (len(nums1)+len(nums2)+1)//2
        # always use the smaller list to do binary search
        if len(nums1)>len(nums2):
            nums1, nums2 = nums2, nums1
        l, r = 0, len(nums1)
        while l < r:
            mid1 = l + (r-l)//2
            mid2 = k-mid1-1 # there should be k items in total including mid1 & mid2
            if nums1[mid1]<nums2[mid2]:
                # need to seek more element in the first list
                l = mid1 + 1
            else:
                r = mid1
        # l is how many we need to take in mid1
        t1 = l-1
        t2 = k-l-1
        if (len(nums1)+len(nums2))%2==1:
            # if total length is odd, just return the larger one
            # deal with special case
            if t1<0:
                return nums2[t2]
            if t2<0:
                return nums1[t1]
            return max(nums1[t1],nums2[t2])
        else:
            # if total length is even
            # we need to find the next element in the two lists
            # and find the two middle element's average
            # we also need to check boundary
            less = max(nums1[t1] if t1>=0 else float("-inf"),nums2[t2] if t2>=0 else float("-inf"))
            more = min(nums1[t1+1] if t1+1<len(nums1) else float("inf"),nums2[t2+1] if t2+1<len(nums2) else float("inf"))
            return float(less+more)/2
```

## rotated 与 peek 类题目

不知道为什么对这一个系列毫无印象，好像当时是不断调整边界条件写了一堆不优雅的垃圾代码过了的……

rotated 的特点是，数组大致上是排列好的，除了某一个部分出了差错，所以你可以想象成「递增-最大-最小-递增但不会比第零个元素还小」这样的序列。那么二分搜索的话，就必定有一半是排好序的那种递增，另一半是中间有一个叛徒断崖式下跌的那种递增。

再就是，虽然可以用模板，但因为可以直接在中途返回是否等于，而且也不会有等于的情况，所以看起来很多人用的都是`l<=r`以及`mid = r-1 or l+1`这样的模板。

这类题目之所以需要讲，都是因为摁着头让你达到`O(logn)`，因此不再赘述时间复杂度。

### LC33 Search in Rotated Sorted Array

理解起来不难，但需要稍微复杂地分类讨论一下。

- 如果`arr[mid]==target`，或者等于两个端点，可以直接返回
- 如果`arr[mid]<target`，需要判断哪边排好序了，哪边有断崖下跌
  - 如果左边排好序了，都比`target`小，那就要搜右边
  - 如果右边排好序了，都比`target`大，而左边也有一部分可能比它大，那就要判断`arr[mid]`是否比右边最大的元素还大，是的话就搜左边，不是的话搜右边
- 如果`arr[mid]>target`，同样需要判断哪边有断崖下跌
  - 如果右边排好序了，都比`target`大，那就要搜左边
  - 如果左边排好序了，都比`target`小，而右边也有一部分可能比它小，那就要判断`arr[mid]`是否比左边最小的元素还小，是的话就搜右边，不是的话搜左边

仍然需要注意边界条件。

### LC81 Search in Rotated Sorted Array II

上一题的姊妹篇，区别是可能有重复元素，返回结果是是否存在而不是坐标。

重复元素的问题在于，如果它们被拦腰斩断，那么有可能在判断的时候没法通过半边数组的七点和终点比较大小——因为它们是相等的。解决方法是，如果遇到相等的情况，那么就将左右搜索范围各缩小一格。

### LC153&154 Find Minimum in Rotated Sorted Array

找到最小的元素，可以考虑递归，分别处理找到两侧的最小值，对于已经排好序的那个递增，可以`O(1)`选到最前面的最小值，而对于没有排好序的那个来说，继续递归处理得到最小值，最后结果就是两边结果比较小的那个。

### LC852 Peak Index in a Mountain Array

这题有点神奇，我一开始是打死都不相信有`O(logn)`解法的（。

是这样的，把一个数组看作一座山，有着「递增-最大值-递减」的规律，并且我们将数组之外的部分看作无限小。给你这么一个满足要求的数组，求山顶的值。

用二分搜索的话，山顶有且只有一个，要么在左要么在右。所以只会出现两种情况：

- 山顶在左，右边纯递减，左边先递增再递减
- 山顶在右，左边纯递增，右边先递增再递减

判断递增递减，第一反应可能是像做 rotated 类题目一样，比较端点大小，可是左边不管是纯递增还是先递增再递减，看两个端点都一样，啊这……

不过，「先递增再递减」这个倒是可以判断。假设左边先递增再递减，那右边只能递减了，所以`arr[mid]>arr[mid+1]`搜左边；右边先递增再递减，说明左边只能递增，所以`arr[mid]<arr[mid+1]`。然后我们可以用这个不断缩小搜索范围，直到最后返回 l 即可。

```python
class Solution(object):
    def findPeakElement(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        l,r = 0, len(nums)-1
        while l<r:
            mid = l+(r-l)//2
            if nums[mid]>nums[mid+1]:
                # go down
                r = mid
            else:
                l = mid + 1
        return l
```

## 参考资料

> [二分查找有几种写法？它们的区别是什么？ - Jason Li 的回答 - 知乎](https://www.zhihu.com/question/36132386/answer/530313852)
