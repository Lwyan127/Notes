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

## [307. 区域和检索 - 数组可修改](https://leetcode.cn/problems/range-sum-query-mutable/)

相见algorithm.md中的树状数组。

```c++
class NumArray {
private: 
    vector<int> tree;
    vector<int> original_nums;
    
    inline int lowbit(int x) {
        return x & (-x);
    }

    int getPrefixSum(int x) {
        int sum = 0;
        while (x > 0) {
            sum += tree[x];
            x -= lowbit(x);
        }
        return sum;
    }

public:
    NumArray(vector<int>& nums): original_nums(nums.size(), 0), tree(nums.size() + 1, 0) {
        for (int i = 0; i < nums.size(); i++) {
            update(i, nums[i]);
        }
    }
    
    void update(int index, int val) {
        int delta = val - original_nums[index];
        original_nums[index] = val;
        index++;
        while (index <= original_nums.size()) {
            tree[index] += delta;
            index += lowbit(index);
        }
        return;
    }
    
    int sumRange(int left, int right) {
        return getPrefixSum(right + 1) - getPrefixSum(left);
    }
};
```

## [2179. 统计数组中好三元组数目](https://leetcode.cn/problems/count-good-triplets-in-an-array/)

难题，思考量很大。

题目本质上是求：nums1 和 nums2 的长度恰好为 3 的**公共子序列**的个数。

如果使用 [1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)则需要n*n的dp数组，时间为O(n^2^)，太大了。

因为0-n每个数只出现过一次，这道题使用**置换的办法将公共子序列变成递增子序列**的问题。

把 nums1=[4,0,1,3,2] 中的每个元素置换，变为nums1=[0,1,2,3,4]。根据这个置换，修改nums2: nums2=[0,2,1,4,3]。由此这是一个递增子序列的问题：即求在nums2中有多少个长度为3的递增子序列。

**长度为3的递增子序列**：

对中间的数nums2[i]枚举：在nums2中它左边且小于它的个数记为less，则右边比他大的个数为n - 1- nums2[i] - (i - less)个，根据乘法原理，两个相乘就是中间的数为num2[i]的递增子序列的个数。

现在只需要求less了：使用**值域树状数组**，值域树状数组的意思是，把元素值视作下标。添加一个值为 3 的数，就是调用树状数组的 update(3,1)，3为下标idx，1为值val。查询小于 3 的元素个数，即小于等于 2 的元素个数，就是调用树状数组的 prefixSum(2)。

```c++
class Solution {
private:
    vector<int> original_nums;
    vector<int> bit;

    inline int lowbit(int x) {
        return x & (-x);
    }

    int prefixSum(int idx) {
        idx++;
        int sum = 0;
        while (idx > 0) {
            sum += bit[idx];
            idx -= lowbit(idx);
        }
        return sum;
    }

    void update(int idx, int val) {
        int delta = val - original_nums[idx];
        original_nums[idx] = val;
        idx++;
        while (idx < bit.size()) {
            bit[idx] += delta;
            idx += lowbit(idx);
        }
        return;
    }

    long long IncreasedSequencesOfThree(vector<int>& nums) {
        long long ans = 0;
        bit.resize(nums.size() + 1, 0);
        original_nums.resize(nums.size(), 0);
        int less;
        for (int i = 0; i < nums.size(); i++) {
            less = prefixSum(nums[i] - 1);
            ans += less * (nums.size() - 1 - nums[i] - (i - less));
            update(nums[i], 1);
        }
        return ans;
    }

public:
    long long goodTriplets(vector<int>& nums1, vector<int>& nums2) {
        int n = nums1.size();
        vector<int> pos(n);
        for (int i = 0; i < n; i++) {
            pos[nums1[i]] = i;
        }
        for (int i = 0; i < n; i++) {
            nums2[i] = pos[nums2[i]];
        }
        return IncreasedSequencesOfThree(nums2);
    }
};
```

## [2537. 统计好子数组的数目](https://leetcode.cn/problems/count-the-number-of-good-subarrays/)

**滑动数组**：也有点难度，要好好想想。

**核心思路**：

- 如果窗口中有 c 个元素 x，再进来一个 x，会新增 c 个相等数对。

- 如果窗口中有 c 个元素 x，再去掉一个 x，会减少 c−1 个相等数对。

外层循环：从小到大枚举子数组右端点 right。现在准备把 x=nums[right] 移入窗口，那么窗口中有 cnt[x] 个数和 x 相同，所以 pairs 会增加 cnt[x]。然后把 cnt[x] 加一。

内层循环：如果发现 pairs≥k，说明子数组符合要求，右移左端点 left，先把 cnt[nums[left]] 减少一，然后把 pairs 减少 cnt[nums[left]]。

内层循环结束后，[left,right] 这个子数组是不满足题目要求的，但在退出循环之前的最后一轮循环，[left−1,right] 是满足题目要求的。由于子数组越长，越能满足题目要求，所以除了 [left−1,right]，还有 [left−2,right],[left−3,right],…,[0,right] 都是满足要求的。也就是说，当右端点固定在 right 时，左端点在 0,1,2,…,left−1 的所有子数组都是满足要求的，这一共有 left 个。

```c++
class Solution {
public:
    long long countGood(vector<int>& nums, int k) {
        long long ans = 0;
        int left = 0;
        unordered_map<int, int> hash;
        int pairs = 0;
        for (int right = 0; right < nums.size(); right++) {
            pairs += hash[nums[right]];
            hash[nums[right]]++;
            while (pairs >= k) {
                hash[nums[left]]--;
                pairs -= hash[nums[left]];
                left++;
            }
            ans += left;
        }
        return ans;
    }
};
```

## [1298. 你能从盒子里获得的最大糖果数](https://leetcode.cn/problems/maximum-candies-you-can-get-from-boxes/)

其实可以算作一个模拟的题目，并不困难，思路类似层序遍历：每次检查size个盒子（和层序遍历一样size要固定住），能开就打开，不能就push到队列后面去。终止条件是用一个flag来判断这次检查size个盒子有没有打开过，一个盒子都没打开说明结束了，另外也有可能所有盒子开完了队列变空了。

## [3403. 从盒子中找出字典序最大的字符串 I](https://leetcode.cn/problems/find-the-lexicographically-largest-string-from-the-box-i/)

暴力枚举即可。注意numFriends为1时的情况。

## [1061. 按字典序排列最小的等效字符串](https://leetcode.cn/problems/lexicographically-smallest-equivalent-string/)

复习了并查集，这道题只需要在union的时候保证并查集的根为字典序最小即可。

## [3170. 删除星号以后字典序最小的字符串](https://leetcode.cn/problems/lexicographically-minimum-string-after-removing-stars/)

使用一个哈希表来记录即可。直接使用map可以用m.begin()直接得到哈希表中最小key，map自动按照key升序排列。

```
map<char, stack<int>> m;
```

## [386. 字典序排数](https://leetcode.cn/problems/lexicographical-numbers/)

这实际上是一个深度优先搜索。因此，使用递归：

```c++
class Solution {
public:
    void getAns(int x, int n) {
        x *= 10;
        if (x > n) return;
        for (int i = 0; i <= 9; i++) {
            if (x + i > n) return;
            ans.emplace_back(x + i);
            getAns(x + i, n);
        }
        return;
    }

    vector<int> lexicalOrder(int n) {
        for (int i = 1; i <= 9; i++) {
            if (i > n) return ans;
            ans.emplace_back(i);
            getAns(i, n);
        }
        return ans;
    }

private:
    vector<int> ans;
};
```

题目要求使用O(1)，因此可以直接按照排序的逻辑来完成这个

```c++
class Solution {
public:
    vector<int> lexicalOrder(int n) {
        int x = 1;
        for (int i = 0; i < n; i++) {
            ans.emplace_back(x);
            if (10 * x <= n) {
                x *= 10;
            } else {
                while (x % 10 == 9 || x + 1 > n) {
                    x /= 10;
                }
                x++;
            }
        }
        return ans;
    }

private:
    vector<int> ans;
};
```

