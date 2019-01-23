---
title: Algorithm to find majority element
date: 2018-10-05 20:46:53
tags: [algorithm, leetcode]
---

This algorithm can be used for [leetcode 229](https://leetcode.com/problems/majority-element-ii/description/) and [leetcode 169](https://leetcode.com/problems/majority-element/description/).

The algorithm is called [Boyer-Moore majority vote algorithm](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_majority_vote_algorithm).

A blog for this algorithm can be found [here](https://gregable.com/2013/10/majority-vote-algorithm-find-majority.html).

This algorithm can solve the following problem: Given an unordered array of size `n` of int, find all the elements that appear more than `floor(n / k)` times, with `O(k - 1)` space complexity and `O(n * (k - 1))` time complexity.

The idea is:
- First, we allocate a vector of `k - 1` elements called `vals`. This is because we can have at most `k - 1` elements that satisfy the condition that it appears more than `floor(n / k)` times. We assign unique numbers to `vals`. We also allocate vector `counts` of size `k - 1` to record the number of times each value appears. `counts` is initialized to all zero.
- Then, for each `num` in `nums`, we check all `k - 1` values.
    - If `num` is the same as one of the `val`, we increase its `count`.
    - If `num` is not the same as any of the `val`, but we find a `val` with `count == 0`, we replace that `val` with `num`, and set its `count` back to `1`.
    - If `num` is not the same as any of the `val`, and for all the `val`s, we have `count > 0`, we then decreace `count` for all the `val`s.
- Finally, we go through all the `k - 1` values we recorded. For each `val`, we find its actual appearance time in `nums`, if this count satisfy `new_count > floor(n / k)`, then, `val` is a solution. If not, we discard this `val`.

The c++ implementation is as following:

```c++
vector<int> majorityElements(const vector<int>& nums, int k) {
    vector<int> vals(k - 1, 0);
    vector<int> counts(k - 1, 0);
    // Initialization.
    for(int i = 0; i < vals.size(); i ++) {
        vals[i] = i;
    }
    for(auto num : nums) {
        // First check if there is a val that equals num.
        bool foundSame = false;
        for(int i = 0; i < vals.size(); i ++) {
            if(vals[i] == num) {
                counts[i] ++;
                foundSame = true;
                break;
            }
        }
        if(foundSame) {
            continue;
        }
        // Then, check if there's a val with count 0.
        bool foundZeroCount = false;
        for(int i = 0; i < vals.size(); i ++) {
            if(counts[i] == 0) {
                vals[i] = num;
                counts[i] = 1;
                foundZeroCount = true;
                break;
            }
        }
        if(foundZeroCount) {
            continue;
        }
        // If we do not find a val == num, nor we find a count == 0, we decrease all counts.
        for(int i = 0; i < vals.size(); i ++) {
            counts[i] --;
        }
    }
    vector<int> rtn;
    // Then, for all the vals we recorded, we check if it is actually a majority element we want.
    for(auto val : vals) {
        int cnt = 0;
        for(auto num : nums) {
            if(num == val) {
                cnt ++;
            }
        }
        if(cnt > nums.size() / k) {
            rtn.push_back(val);
        }
    }
    return rtn;
}
```




