---
title: Stack-sortable array
date: 2018-10-13 13:55:33
tags: [algorithm, leetcode]
---

This algorithm can be used for [leetcode 456](https://leetcode.com/problems/132-pattern/description/).

The problem is [Stack-sortable permutation](https://en.wikipedia.org/wiki/Stack-sortable_permutation). The problem is to try to sort a array of numbers using only a single stack.

The input array is stack-sortable only if it does not contain `231` pattern.

As a result, we can check if the input sequence contains `231` pattern by checking if it is stack-sortable.

Also, we can check if the input sequence contains `132` pattern by reverse the input sequence first, then, check if it contains `231` pattern by checking if the reversed sequence is stack-sortable.

The algorithm is as following:

```c++
vector<int> stackSort(const vector<int>& nums) {
    stack<int> st;
    vector<int> rtn;
    // According to Wiki, every two different stack-sortable permutations produce a different Dyck string.
    // The Dyck string corresponds to push and pops.
    // It seems that even the sequence is not stack-sortable, it also corresponds to a Dyck string.
    // For example, both {2, 3, 1} and {1, 3, 2} corresponds to string "()(())".
    string dyck_lang = "";
    for(int i = 0; i < nums.size(); i ++) {
        while(!st.empty() && nums[i] > st.top()) {
            rtn.push_back(st.top());
            st.pop();
            dyck_lang.push_back(')');
        }
        st.push(nums[i]);
        dyck_lang.push_back('(');
    }
    while(!st.empty()) {
        rtn.push_back(st.top());
        st.pop();
        dyck_lang.push_back(')');
    }
    cout << dyck_lang << endl;
    return rtn;
}
```

If the given input has a `231` pattern, say input is `231`, before `3` is pushed onto stack, `2` will be popped and added to result, but, `1` is less than `2` so it should be added before `2`.

Another way to solve the `132 Pattern` problem is as following:

We want to find `s1 < s3 < s2`. We start from the end of the input, and push the elements we get into a stack. 

When the value we encounter is greater than the top of the stack, it means that we have a candidate for `s2`. This is because there are some values after `s2` that is less than `s2`. 

Then, we start popping out the numbers from stack that are less than `s2` we found. And, we keep track of the `max` of the numbers we popped out. This `max` number is the `s3` we want to use for this `s2`. We want to keep the `max` because we want to have more possibility for `s1`.

If the value we encounter is smaller than the `s3` we recorded, we can say that we have found a `s1`. This is because the `s3` we recorded must have a `s2` corresponding to it (`s3 < s2`). If we found a `s1` such that `s1 < s3`, we know that we have found the `s1 < s3 < s2` sequence.

This solution can be found [here](https://leetcode.com/problems/132-pattern/discuss/94071/Single-pass-C++-O(n)-space-and-time-solution-(8-lines)-with-detailed-explanation.).

The implementation is as following:

```c++
class Solution {
public:
    bool find132pattern(vector<int>& nums) {
        int s3 = INT_MIN;
        stack<int> st;
        for(int i = nums.size() - 1; i >= 0; i --) {
            // If we found nums[i] < s3, it means that we found s1 < s3 < s2 pattern. 
            if(nums[i] < s3) {
                return true;
            }
            // We found a s2 candidate.
            if(!st.empty() && nums[i] > st.top()) {
                s3 = INT_MIN;
                while(!st.empty() && nums[i] > st.top()) {
                    s3 = max(s3, st.top());
                    st.pop();
                }
            }
            st.push(nums[i]);
        }
        return false;
    }
};
```