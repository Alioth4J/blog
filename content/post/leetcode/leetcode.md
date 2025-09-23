---
title: Algorithm Template of LeetCode
description: Here are some tricks.
date: 2025-09-14
slug: sliding-window.md
image: 
categories:
    - Algorithm
---

## Sliding Window / Two Pointers
1. Declare variables.
2. Iterate right border. `for (int right = 0; right < n; right++)`
3. Add right border into sliding window.
4. Moving the left border to achieve a border situation.
5. Update `res`.
6. Remove left border from sliding window.

## Binary Search
```java
    /**
     * Find the index of the first num that
     * is greater than or equal to <code>target</code>.
     */
    private int biSearch(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] >= target) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return left;
    }
    
    public void anotherMethod() {
        int idx = biSearch(nums, target);
        if (idx >= nums.length || nums[idx] != target) {
            return ...;
        }
        return ...;
    }
```

