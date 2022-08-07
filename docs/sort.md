# 常用排序总结

# 前言
学习计算机就像修炼武功，内力深厚者，飞花摘叶亦可伤人。而个人认为数据结构，就属于计算机的内功心法之一。本人基础薄弱，便借由写文的机会，做一下基础知识点的整理总结工作。

# 总结
![](https://images.xiaozhuanlan.com/photo/2019/2f58133cad1d58c310dc7b85a38ffa66.png)


# 冒泡排序
依次判断相邻的两个元素，并进行交换；如果在一趟循环中，没有发生元素交换，证明已经有序，则退出。
```
def bubble_sort(l):
    for i in range(len(l)):
        flag = False
        for j in range(len(l) - 1):
            if (l[j] > l[j + 1]):
                l[j], l[j + 1] = l[j + 1], l[j]
                flag = True
        if (not flag):
            break
```
# 插入排序
把序列分成两个区间，每次将待排元素插入到有序区间的合适位置，使得有序区间保持有序。

```
def insert_sort(l):
    for i in range(len(l)):
        v = l[i]
        j = i - 1
        while (j >= 0 and v < l[j]):
            l[j + 1] = l[j]
            j -= 1
        l[j + 1] = v

```
# 选择排序
把序列分成两个区间，每次将待排序区间中的最小（或最大值）插入到有序区间末尾，使得有序区间保持有序。

```
def select_sort(l):
    for i in range(len(l)):
        j = i
        min = l[i]
        for k in range(i, len(l)):
            if l[k] < min:
                min = l[k]
                j = k
        l[i], l[j] = l[j], l[i]
```

# 希尔排序
增量缩小插入排序，根据跨度将数据分组，每次组内进行插入排序，最终跨度为1，对整个区间进行插入排序。通过在不同组内排序，可以一次降低若干逆序度，使最后全局的插入排序，只要做微小调整即可。

```
def shell_sort(l):
    gap = len(l) >> 1
    while (gap > 0):
        for i in range(gap, len(l)):
            v = l[i]
            j = i - gap
            while (j >= 0 and v < l[j]):
                l[j + gap] = l[j]
                j = j - gap
            l[j + gap] = v
        gap = gap >> 1
```

# 快速排序
使用递归的思想，选取区间的一个标值，使左区间比标值小，右区件比标值大；再依次对左右区间进行上述操作。
```
def quick_sort(l):
    def _partition(l, p, q):
        pivot = l[p]
        while (p < q):
            while (p < q and l[q] >= pivot):
                q -= 1
            l[p] = l[q]
            while (p < q and l[p] <= pivot):
                p += 1
            l[q] = l[p]
        l[p] = pivot
        return p

    def _quick_sort(l, p, q):
        if (p >= q):
            return
        index = _partition(l, p, q)
        _quick_sort(l, p, index - 1)
        _quick_sort(l, index + 1, q)

    _quick_sort(l, 0, len(l) - 1)
```

# 归并排序
利用递归思想，将区间不断二分，直到区间只有一个元素，再向上合并，合并时同时排序。最终整个区间有序。

```
def merge_sort(l):
    def _merge(l, left, right, p, q):
        i = j = 0
        k = p
        while (i < len(left) and j < len(right)):
            if (left[i] < right[j]):
                l[k] = left[i]
                i = i + 1
            else:
                l[k] = right[j]
                j = j + 1

            k = k + 1

        while (i < len(left)):
            l[k] = left[i]
            i += 1
            k += 1

        while (k < len(right)):
            l[k] = right[j]
            j += 1
            k += 1

        return l[p:p + len(left) + len(right)]

    def _merge_sort(l, p, q):
        if (p >= q):
            return [l[p]]
        mid = (p + q) >> 1
        left = _merge_sort(l, p, mid)
        right = _merge_sort(l, mid + 1, q)
        return _merge(l, left, right, p, q)

    _merge_sort(l, 0, len(l) - 1)
```


# 桶排序
当数据上下限范围不大，且数据分布均匀时，可以将待排数据均匀划分到一定长度的桶中，在桶中依次排序，最后遍历每个桶，即可得到排序结果。

```
def bucket_sort(l, bucketSize):
    minValue = min(l)
    maxValue = max(l)
    bucketNum = (maxValue - minValue + 1) // bucketSize + 1
    bukcets = [[] for x in range(bucketNum)]
    for v in l:
        bukcets[(v - minValue) // bucketSize].append(v)

    l.clear()
    for i in range(bucketNum):
        bukcets[i] = sorted(bukcets[i])
        l.extend(bukcets[i])
```

# 计数排序

计数排序是一种特殊的桶排序，桶的长度为一，只要遍历桶中有值的位置，对应的角标即为原值。


```
def count_sort(l):
    minValue = min(l)
    maxValue = max(l)
    bucketNum = maxValue - minValue + 1
    bucket = [0 for x in range(bucketNum)]
    for v in l:
        bucket[(v - minValue)] += 1

    for i in range(1, bucketNum):
        bucket[i] = bucket[i] + bucket[i - 1]

    tempBucket = [0 for x in range(len(l))]

    for v in reversed(l):
        n = bucket[v - minValue]
        bucket[v - minValue] -= 1
        tempBucket[n - 1] = v

    l.clear()
    l.extend(tempBucket)
```

# 基数排序
原地排序可以不改变同样数据在排列过程中的顺序，基数排序就是利用这一点，对数值的每一位分别排序，最终得到的就是有序数列。

```
def radix_sort(l):
    bitNum = len(str(max(l)))
    for i in range(bitNum):
        buckets = [[] for x in range(10)]
        for v in l:
            if i == 0:
                n = v % 10
            else:
                k = v // (10 ** i)
                n = k % 10

            buckets[n].append(v)
        l.clear()
        for bucket in buckets:
            l.extend(bucket)

```

