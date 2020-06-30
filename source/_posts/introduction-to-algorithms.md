---
title: 「算法导论」读书笔记
catalog: true
date: 2020-06-10 12:15:34
subtitle:
header-img:
tags:
- interview
categories:
- 算法
---

## 堆排序
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/introduction-to-algorithms/build-heap.png?raw=true)

```
class Solution {
public:
    // 堆排序
    void heapSort(vector<int> &nums) {
        // 建堆 时间复杂度O(n)
        buildMaxHeap(nums);
        // 取出堆顶元素，顶重建堆O(nlogn)
        for (int i=nums.size()-1;i>0;i--) {
            swap(nums[0], nums[i]);
            maxHeapify(nums, 0, i); // log(n)
        }
    }

private:
    // 非叶子节点，由下向上逐个下滤
    void buildMaxHeap(vector<int> &nums) {
        for (int i = nums.size() / 2 - 1; i >= 0; i--)
            maxHeapify(nums, i, nums.size());
    }

    // 下滤，把当前节点向下寻找其应该在的位置，要求当前节点的左右子节点都满足堆的性质
    void maxHeapify(vector<int> &nums, int index, int size) {
        if (index >= size) return;
        // 找出节点和左右儿子中的最大者
        int left = 2 * index + 1;
        int right = 2 * index + 2;
        int larger = index;
        if (left < size && nums[left] > nums[index]) larger = left;
        if (right < size && nums[right] > nums[left]) larger = right;
        // 如果节点不是最大的
        if (larger != index) {
            // 交换节点
            swap(nums[index], nums[larger]);
            // 递归下滤
            maxHeapify(nums, larger, size);
        }
    }
};
```

## 快速排序

```
class Solution {
public:
    void quickSort(vector<int>::iterator begin, vector<int>::iterator end) {
        if (begin < end) {
            auto pivot = partition(begin, end);
            quickSort(begin, pivot);
            quickSort(next(pivot), end);
        }
    }

private:
    // (pivot, index)开区间是比pivot小的值，最后将prev(index)和pivot交换
    vector<int>::iterator partition(vector<int>::iterator begin, vector<int>::iterator end) {
        auto pivot = begin;
        auto index = next(begin);
        for (auto iter = next(begin); iter != end; ++iter) {
            if (*iter < *pivot) {
                iter_swap(iter, index);
                index++;
            }
        }
        iter_swap(pivot, prev(index));
        return prev(index);
    }
};
```

## 线性时间排序

### 排序算法下界

对一个正确的比较排序算法来说，n!种可能的排列都应该出现在决策树的叶结点上

一个排序算法中的最坏情况，比较次数等于决策树的高度`h>=lg(n!)=Ω（nlgn）`

### 计数排序

计数排序的最坏时间复杂度、最好时间复杂度、平均时间复杂度、最坏空间复杂度都是O(n+k)。n为元素个数，k为待排序数的最大值

### 基数排序

每位排序

### 桶排序

最坏时间复杂度是O(n^2)， 平均复杂度是O(n+k)，最坏空间复杂度是O(n*k)

查询O((n+k)/n)，排序O(n+k)

## 动态规划

动态规划也是一种分治思想（比如其状态转移方程就是一种分治），但与分治算法不同的是，分治算法是把原问题分解为若干个子问题，自顶向下求解子问题，合并子问题的解，从而得到原问题的解。动态规划也是把原始问题分解为若干个子问题，然后自底向上，先求解最小的子问题，把结果存在表格中，在求解大的子问题时，直接从表格中查询小的子问题的解，避免重复计算，从而提高算法效率。

是指问题的最优解包含其子问题的最优解。最有子结构是使用动态规划的最基本的条件，如果不具有最优子结构性质，就不可以使用动态规划解决。

## 贪心算法

### 贪心动规区别

动规：

1. 全局最优解中一定包含某个局部最优解，但不一定包含前一个局部最优解，因此需要记录之前的所有最优解
2. 动态规划的关键是状态转移方程，即如何由以求出的局部最优解来推导全局最优解

贪心

1. 贪心算法中，作出的每步贪心决策都无法改变，因为贪心策略是由上一步的最优解推导下一步的最优解，而上一部之前的最优解则不作保留。