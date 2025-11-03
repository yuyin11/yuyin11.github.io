+++
date = '2025-11-03T20:37:43+08:00'
draft = false
title = '二分查找算法及其变体总结'
tags = ['算法']
+++

> 这是一篇关于二分查找算法的文章，主要是总结算法本身与它的变体，重在不断的巩固熟悉。

## 最初的写法
```cpp
int binary_search(vector<int> const& nums, int target) {
  int l = 0, r = nums.size() - 1;
  while (l <= r) {
    int mid = (l + r) / 2;
    if (nums[mid] < target)
      l = mid + 1;
    else if (nums[mid] > target)
      r = mid - 1;
    else
      return mid
  }
  return -1;
}
```

## 变体
> 求下界(lower_bound)
```cpp
int binary_search_first(vector<int> const& nums, int target) {
  int l = 0, r = nums.size();
  // 注意是左闭右开区间
  while (l < r) { // '<'
    int mid = (l + r) / 2;
    if (nums[mid] < target)
      l = mid + 1;
    else
      r = mid;
    return l;
  }
}
```

> upper_bound
```cpp
int upper_bound(vector<int> const& nums, int target) {
  int l = 0, r = nums.size();
  while (l < r) {
    int mid = (l + r) / 2;
    if (nums[mid] <= target)
    // 唯一区别 '<='
      l = mid + 1;
    else
      r = mid;
    return l;
  }
}
```

> 求上界(floor)
```cpp
int find_last_less_equal(vector<int> const& nums, int target) {
  int l = 0, r = nums.size();
  int result = -1;
  while (l < r) {
    int  mid = (l + r) / 2;
    if (nums[mid] <= target) {
      result = mid;
      l = mid + 1;
    }
    else
      r = mid - 1;
  }
  return result;
  // 一种与lower_bound保持对称性的写法(也对)
  // int l = 0, r = nums.size() - 1;
  // while (l < r) {
  //   int mid = (l + (r - l) + 1) / 2;
  //   if (nums[mid] > target)
  //     r = mid - 1;
  //   else l = mid;
  // }
  // return (nums[l] <= target) ? l : -1;
  // 但可能并不直观
}
```
