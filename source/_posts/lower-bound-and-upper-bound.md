---
title: lower_bound and upper_bound
date: 2018-07-29 13:21:19
tags: [c++]
---
Tried to use stl `lower_bound` and `upper_bound` today and noticed that I didn't quite understand them correctly before.

[The corresponding leetcode problem](https://leetcode.com/problems/random-pick-with-weight/description/)

Some notes on these:

For `std::lower_bound(T k)`, it returns the first element that is **greater or equal** to k.

For `std::upper_bound(T k)`, it returns the first element that is **greater** than k.

For example:

```c++
#include <algorithm>
#include <cassert>
#include <map>
#include <iostream>

using namespace std;

int main() {
    map<int, int> hm {{1, 1}, {3, 3}, {5, 5}, {7, 7}};
    map<int, int>::iterator it;
    // it == hm.end() because there does not exist a value in hm that is >= 8.
    it = hm.lower_bound(8);
    if(it == hm.end()) {
        cout << "Can't find a value >= 8..." << endl;
    } else {
        assert(0);
    }
    // it points to {7, 7}.
    it = hm.lower_bound(7);
    cout << "lower_bound for 7: " << it->first << endl;
    // it == hm.end() because there does not exist a value in hm that is > 7.
    it = hm.upper_bound(7);
    if(it == hm.end()) {
        cout << "Cannot find a value > 7..." << endl;
    } else {
        assert(0);
    }
    // it points to {5, 5}.
    it = hm.lower_bound(5);
    cout << "lower_bound for 5: " << it->first << endl;
    // it points to {7, 7}.
    it = hm.upper_bound(5);
    cout << "upper_bound for 5: " << it->first << endl;	
    return 0;
}
```

If we want to find a value in `std::map` that is strictly less than a given key `k`, we can do the following:

```c++
auto it = hm.lower_bound(k);
if(it == hm.begin()) {
    cout << "Can't find a value that is strictly less than k!" << endl;
} else {
    --it;
    cout << "The value that is strictly less than k is: " << it->first << endl;
}
```