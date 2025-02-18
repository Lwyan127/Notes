## [2831. 找出最长等值子数组](https://leetcode.cn/problems/find-the-longest-equal-subarray/)

**滑动窗口**

- 把相同元素分组，相同元素的下标记录到哈希表（或者数组）$posLists$ 中。

  例如 $nums = [1,3,2,3,1,3]$，元素 $3$ 在 $nums$ 中的下标有 1,3,5，那么 $posLists[3]=[1,3,5]$。

- 然后用**滑动窗口**计算。设窗口左右端点为 $left$ 和 $right$。

  - 假设 $nums$ 的等值子数组的元素下标从 $pos[left]$ 到 $pos[right]$，那么在删除前，子数组的长度为

    $pos[right]−pos[left]+1$ 

  - 这个子数组有$right−left+1$个数都是相同的，无需删除，其余元素都需要删除，那么需要删除的元素个数就是

    $pos[right]−pos[left]−(right−left)$

  - 如果上式大于 $k$，说明要删除的数太多了，那么移动左指针 $left$，直到上式小于等于 $k$，此时用 $right−left+1$ 更新答案的最大值。

- 还有，这样的数据结构，即每一个数都有他对应的数组，因为数的大小你不可预测，所以可以使用哈希表中放一个数组的结构，这样key是数字，对应的值为一个数组。如此，就不用开一个很大的二维数组了。

- 计算$vec[i] - vec[j] - (i - j)$ 时可以将数组的值简化为 $vec[i] - i$ ，因为仔细看就可以发现这个值从左到右会越来越大，因此这个滑动窗口就相当于在一个递增数组中找一个，值的差值小于等于k的最大子数组。

- 时间复杂度：O(n)，其中 n 为 nums 的长度。
  空间复杂度：O(n)。


```c++
class Solution {
public:
    int longestEqualSubarray(vector<int>& nums, int k) {
        int n = nums.size();
        unordered_map<int, vector<int>> pos;  // 用这样一个哈希表中放数组
        for (int i = 0; i < n; i++) {
            pos[nums[i]].emplace_back(i);  // 相当于push_back
        }
        int ans = 0;
        for (auto &[_, vec] : pos) {
            /* 缩小窗口，直到不同元素数量小于等于 k */
            for (int i = 0, j = 0; i < vec.size(); i++) {  // 这里j为left，i为right
                while (vec[i] - vec[j] - (i - j) > k) {
                    j++;
                }
                ans = max(ans, i - j + 1);
            }
        }
        return ans;
    }
};
```

## [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

用unordered_map作为哈希表。

```c++
unordered_map<string, vector<string>> mp;
```

用sort可以直接排序string

```c++
sort(key.begin(), key.end());
```

```c++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> mp;
        for (string& str: strs) {
            string key = str;
            sort(key.begin(), key.end());
            mp[key].emplace_back(str);
        }
        vector<vector<string>> ans;
        for (auto it = mp.begin(); it != mp.end(); ++it) {
            ans.emplace_back(it->second);
        }
        return ans;
    }
};
```

## [128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

实现时间复杂度为 `O(n)` 的算法解决此问题，不能用set（本质是红黑树，查询增删都需要logn，每个数放入set中总共就是nlogn），使用unordered_set（哈希表，o(1)）。

这道题的思路是：将所有数放到unordered_set哈希表中，然后遍历。如果num前面的数字不在哈希表里了，说明num可能是数字连续的最长序列的首个数字，那就循环去找num+1,num+2...，如果num前面的数字在哈希表里，就跳过。

记得直接用max取较大值

```c++
max_len = max{len, max_len};
```

```c++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> num_set;
        int max_len = 0;
        for (auto num: nums) {
            num_set.insert(num);
        }
        for (auto num: num_set) {
            if (num_set.find(num - 1) == num_set.end()) {
                int tmp_num = num;
                int len = 1;
                while (num_set.find(tmp_num + 1) != num_set.end()) {
                    tmp_num++;
                    len++;
                }
                max_len = max(max_len, len);
            }

        }
        return max_len;
    }
};
```

## [240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)

看到二维矩阵并且有二分标签，原本想像平面四叉树平面直接分成四块来二分，但是写的递归很麻烦，似乎还会sof

看了题解，两个办法：

1. 只在行/列使用二分
2. 根据行和列都是升序的，从某个角出发，根据升序的规则，每次减少一行或者一列，直到找到或超出边界

这里使用方法2：

```c++
class Solution {
    public:
        bool searchMatrix(vector<vector<int>>& matrix, int target) {
            int m = matrix.size(), n = matrix[0].size();
            int x = 0, y = n - 1;
            while (x < m && y >= 0) {
                if (matrix[x][y] == target) return true;
                else if (matrix[x][y] > target) y--;
                else if (matrix[x][y] < target) x++;
            }
            return false;
        }
};
```

