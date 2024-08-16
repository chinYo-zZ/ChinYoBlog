---
title: C++ 算法： 区间符合的二分查找
date: 2024-08-07 15:09:24
tags: C++
categories: C++
sitemap: true
---
属于个人总结

 - [二分查找](#二分查找模版)
 - [区间符合的二分查找](#区间符合的二分查找)


## 二分查找模版
```C++
#include <iostream>
#include <vector>
using namespace std;
int BinarySearch(vector<int> &nums, int Target)
{
    int i = 0, j = nums.size() - 1;
    while (i <= j)
    {
        int mid = i + (j - i) / 2;
        if (nums[mid] < Target)
        {
            i = mid + 1;
        }
        else if (nums[mid] > Target)
        {
            j = mid - 1;
        }
        else
        {
            return mid;
        }
    }
    return -1;
}

int main()
{
    vector<int> vec{5, 10, 15, 20, 25, 30, 35, 40, 45, 50};
    cout << BinarySearch(vec, 30);
    return 0;
}
```
## 区间符合的二分查找
所有极端情况都可以保证low指向最终结果

<div align=center><img  alt="区间符合的二分查找" src="image.png"/></div>



可以看这些题目

[寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/)

[P2678 [NOIP2015 提高组] 跳石头](https://www.luogu.com.cn/problem/P2678)