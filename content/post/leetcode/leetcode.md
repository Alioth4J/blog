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

## Monotonic Stack
```java
class Solution {
    private int[][] nearestGreater(int[] nums) {
        int n = nums.length;

        int[] left = new int[n];
        Deque<Integer> st = new ArrayDeque<>();
        st.push(-1); // sentinel (optional)
        for (int i = 0; i < n; i++) {
            int x = nums[i];
            while (st.size() > 1 && nums[st.peek()] <= x) {
                st.pop();
            }
            left[i] = st.peek();
            st.push(i);
        }

        int[] right = new int[n];
        st.clear();
        st.push(n); // sentinel (optional)
        for (int i = n - 1; i >= 0; i--) {
            int x = nums[i];
            while (st.size() > 1 && nums[st.peek()] <= x) {
                st.pop();
            }
            right[i] = st.peek();
            st.push(i);
        }

        return new int[][]{left, right};
    }
}
```

- What's in the stack? index or number?
- [left to right] or [right to left]
- According to the concrete, choose `<`, `<=`, `>`, `>=`
- Sentinel is optional, which can avoid `stack.isEmpty()`

## BFS
```java
class Solution {

    private int[][] dirs = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}};

    public int numIslands(char[][] grid) {
        if (grid == null || grid.length < 1 || grid[0].length < 1) {
            return 0;
        }

        int m = grid.length;
        int n = grid[0].length;
        int res = 0;

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == '1') {
                    res++;
                    grid[i][j] = '0';
                    Deque<int[]> q = new ArrayDeque<>();
                    q.offer(new int[]{i, j});
                    while (!q.isEmpty()) {
                        int[] cur = q.poll();
                        int r = cur[0];
                        int c = cur[1];
                        for (int[] dir : dirs) {
                            int nr = r + dir[0];
                            int nc = c + dir[1];
                            // index out of bounds
                            if (nr < 0 || nr >= m || nc < 0 || nc >= n) {
                                continue;
                            }
                            // water
                            if (grid[nr][nc] == '0') {
                                continue;
                            }
                            grid[nr][nc] = '0';
                            q.offer(new int[]{nr, nc});
                        }
                    }
                }
            }
        }
        return res;
    }
}
```

## DFS
```java
class Solution {

    private char[][] grid;
    private int m;
    private int n;

    public int numIslands(char[][] grid) {
        if (grid == null || grid.length < 1 || grid[0].length < 1) {
            return 0;
        }

        this.grid = grid;
        m = grid.length;
        n = grid[0].length;

        int res = 0;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == '1') {
                    res++;
                    dfs(i, j);
                }
            }
        }
        return res;
    }

    private void dfs(int i, int j) {
        if (i < 0 || i >= m || j < 0 || j >= n || ) {
            return;
        }
        if (grid[i][j] == '0') {
            return;
        }
        grid[i][j] = '0';
        dfs(i + 1, j);
        dfs(i - 1, j);
        dfs(i, j + 1);
        dfs(i, j - 1);
    }

}
```

