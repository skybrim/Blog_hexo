---
title: 排序算法
comments: true
date: 2020-07-27 09:46:17
tags: algorithm
---

排序算法
<!--more-->

```cpp
#include <vector>

using namespace std;

class Solution {
public:
    vector<int> sortArray(vector<int>& nums) {
        quickSort(nums, 0, (int)nums.size()-1);
        return nums;
    }
    
    void quickSort(vector<int>& nums, int l, int r) {
        if (l >= r) {
            return;
        }
        int pivot = nums[l];
        int index = l;
        for (int i = l + 1; i <= r; ++i) {
            if (nums[i] < pivot) {
                ++index;
                swap(nums[i], nums[index]);
            }
        }
        swap(nums[l], nums[index]);
        quickSort(nums, l, index-1);
        quickSort(nums, index+1, r);
    }
};
```