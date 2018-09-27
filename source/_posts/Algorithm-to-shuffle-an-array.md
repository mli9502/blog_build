---
title: Algorithm to shuffle an array
date: 2018-09-26 21:41:07
tags: [algorithm, leetcode]
---

This algorithm can be used for [leetcode 519](https://leetcode.com/problems/random-flip-matrix/description/). The algorithm is called [Fisher-Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle#Fisher_and_Yates'_original_method).

The question is to output the index of an array in a random order (we don't care the actual value stored at that position), and minimize the number of `rand()` called and also optimize for space and time complexity.

e.g.: Given array of size 6, the output may look like the following:
```
(0, 3), (0, 1), (0, 2), (0, 4), (0, 0), (0, 5)
```

To achieve this, we can use the following code:
```c++
// Get a random number in range [minVal, maxVal].
int getRand(int minVal, int maxVal) {
	return rand() % (maxVal - minVal + 1) + minVal;
}

vector<int> solution(const vector<int>& vec) {
	int maxVal = vec.size() - 1;
	unordered_map<int, int> hm;
	vector<int> rtn;
	// We loop vec.size() times to get output all the index.
	for(int i = 0; i < vec.size(); i ++) {
		// Choose a random index in range [0, maxVal].
		int randIdx = getRand(0, maxVal);
		int currSelectedIdx;
		// If we have not seen randIdx before, 
		// it means that randIdx is the current selected index, and we can just return it.
		if(hm.find(randIdx) == hm.end()) {
			currSelectedIdx = randIdx;
		} else {
			// If we have seen randIdx before, it means that randIdx has been selected before.
			// And, we need to select the index in hm[randIdx], which is what randIdx represents now.
			currSelectedIdx = hm[randIdx];
		}
		// After the index for current iteration is determined, we need to swap it with the index represented by current maxVal.
		// If maxVal still represents maxVal.
		if(hm.find(maxVal) == hm.end()) {
			hm[randIdx] = maxVal;
		} else {
			// IMPORTANT !!!
			// If maxVal has been chosen before, and it now represents something else.
			hm[randIdx] = hm[maxVal];
		}
		maxVal --;
		rtn.push_back(currSelectedIdx);
	}
}
```

This code can be easily extended to 2-D matrix, as we just have to convert `maxVal` into `rowCnt * colCnt - 1`. And `randIdx` has to be converted to `(row, col)` by using:
```c++
int row = randIdx / colCnt;
int col = randIdx % colCnt;
```

Here's an example of running the algorithm:

```
init: 
map = {}
[0, 1, 2, 3, 4, 5]
rtn = []
--- randIdx = 3 (from [0, 5])
map = {3: 5}
rtn = [3]
[0, 1, 2, 5, 4, X]
--- randIdx = 1 (from [0, 4])
map = {3: 5,
       1: 4}
rtn = [3, 1]
[0, 4, 2, 5, X, X]
--- randIdx = 2 (from [0, 3])
map = {3: 5,
       1: 4,
       2: 5}
rtn = [3, 1, 2]
[0, 4, 5, X, X, X]
--- randIdx = 1 (from [0, 2])
map = {3: 5
       1: 4,
       2: 5}
rtn = [3, 1, 2, 4]
[0, 5, X, X, X, X]
--- randIdx = 0 (from [0, 1])
map = {3: 5
       1: 4,
       2: 5,
       0, 5}
rtn = [3, 1, 2, 4, 0]
[5, X, X, X, X, X]
--- randIdx = 0 (from [0, 0])
map = {3: 5
       1: 4,
       2: 5,
       0, 5}
rtn = [3, 1, 2, 4, 0, 5]
[X, X, X, X, X, X]
```