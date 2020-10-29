---
title: 排序算法
comments: true
date: 2020-07-27 09:46:17
tags: algorithm
---

排序算法
<!--more-->

## 快速排序

* 递归&双边循环

```python
def sort_array_quick(nums):
    """
    快排 递归 双边循环
    @param nums: List[int]
    @return: List[int]
    """
    def quick_sort(start, end):
        if start >= end:
            return
        pivot = nums[start]
        left, right = start, end
        while left != right:
            while left < right and nums[right] > pivot:
                right -= 1
            while left < right and nums[left] <= pivot:
                left += 1
            nums[left], nums[right] = nums[right], nums[left]
        nums[start], nums[left] = nums[left], nums[start]
        quick_sort(start, left - 1)
        quick_sort(left + 1, end)

    quick_sort(0, len(nums) - 1)
    return nums
```

* 递归&单边循环

```python
def sort_array_quick_single(nums):
    """
    快排 递归 单边循环
    @param nums: List[int]
    @return: List[int]
    """
    def quick_sort(start, end):
        if start >= end:
            return
        pivot = nums[start]
        index = start
        for k in range(start + 1, end + 1):
            if nums[k] < pivot:
                index += 1
                nums[index], nums[k] = nums[k], nums[index]
        nums[start], nums[index] = nums[index], nums[start]
        quick_sort(start, index - 1)
        quick_sort(index + 1, end)

    quick_sort(0, len(nums) - 1)
    return nums
```

```c
#include <stdio.h>
#include <stdlib.h>

int* sortArray(int*, int, int*);
void quickSort(int *, int, int);

int main(void) {
    int *nums = (int *)malloc(sizeof(int) * 5);
    for (int j = 0; j < 5; j ++) {
        nums[j] = 5 - j;
    }
    int len;
    sortArray(nums, 5, &len);
    for (int i = 0; i < len; i ++) {
        printf("%d\n", nums[i]);
    }
    return 0;
}

int* sortArray(int* nums, int numsSize, int* returnSize){
    *returnSize = numsSize;
    quickSort(nums, 0, numsSize-1);
    return nums;
}

void quickSort(int *nums, int l, int r) {
    if (l >= r) {
        return;
    }
    int pivot = nums[l];
    int index = l;
    int t;
    for (int k = l + 1; k < r + 1; k ++) {
        if (nums[k] < pivot) {
            index ++;
            t = nums[index];
            nums[index] = nums[k];
            nums[k] = t;
        }
    }
    t = nums[l];
    nums[l] = nums[index];
    nums[index] = t;
    quickSort(nums, l, index - 1);
    quickSort(nums, index + 1, r);
}
```


## 归并排序

```python
def sort_array_merge(nums):
    """
    归并排序
    @param nums: List[int]
    @return: List[int]
    """
    if len(nums) <= 1:
        return nums
    mid = len(nums) // 2
    left_nums = merge_sort(nums[:mid])
    right_nums = merge_sort(nums[mid:])
    return merge(left_nums, right_nums)


def merge(left, right):
    res = []
    i, j = 0, 0
    while i < len(left) and j < len(right):
        if left[i] < right[j]:
            res.append(left[i])
            i += 1
        else:
            res.append(right[j])
            j += 1
    res += left[i:]
    res += right[j:]
    return res
```