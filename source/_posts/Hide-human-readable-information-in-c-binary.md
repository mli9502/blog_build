---
title: Hide human-readable information in c++ binary
date: 2018-09-26 12:59:32
tags: [c++]
---
I was recently working on hiding human readable information from a binary compiled from c++ source code. 

Here are some of the steps I took to hide as much human readable information from binary as possible.

## Encrypt string literals in source code

Since string literals will be put into `.rodata` section in binary, it is easy for user to read it simply by using the `strings` command in linux. 

There are some ways mentioned online that use template meta programming to encrypt the string literals at compile time (and decrypt at run time). But, it require `c++11` standard to work. 

Then, we found [this post](http://blog.sevagas.com/?String-encryption-using-macro-and) which does not need `c++11` to work. 

The basic idea is to surround a macro around the string litrals you want to encrypt as following:
```c++
// In a header file:
#define SEC_STR(A) (decryptStr(A ## "\0\."))

// In the place where you want your string to be encrypted:
printf("A string you want to hide in binary: %s", SEC_STR("some secret..."));
```
The macro `SEC_STR` does two things:

* It adds a suffix `\0\.` after string `A`.

	According to the post, encryption is done by directly replace the string literals in binary to the encrypted version. As a result, it needs a way to make sure that the string it locates in binary is the string that user want's to encrypt. So, after it finds a string in binary, it checks the suffix to see if it is a target string. 

* It calls the function `decryptStr` before return the string.

	By doing this, the actual string (which is the reutrn value of `decryptStr`) is used at **run-time**.

However, the implementation in the blog have several problems:

* The `decryptStr` function uses a buffer to keep the decrypted string. 

	This is a problem when multiple strings needs to be decrypted before they are used, e.g.:
	```c++
	// In helper.h:
	// we would like to hide str1 and str2 in binary.
	void some_func_call(const char* str1, const char* str2);
	// In helper.cpp:
	some_func_call(SEC_STR("hello"), SEC_STR("world"));
	```
	Before `some_func_call` is called at run-time, the `"hello"` and `"world"` need to be decrypted first.

	However, when `"hello"` is decrypted, the string `hello` is stored in the buffer. Then, when `"world"` is decrypted, it will overwrite the `hello` in the buffer. As a result, in the `some_func_call`, we get two `world` instead of `hello world`.

	This problem will also occure when multi-thread is used.

	But, this is relatively easy to fix. All we have to do is to use a global `std::map<const char*, std::string>` to store the strings we decrypted. 

* When we search for strings with our suffix in binary, there is a chance that there are some random string also have the same suffix. We could potentially break the binary when we encrypt the string.

* If we use this method, the encrypted string must have the same length as the origional string, otherwise, we could also break the binary.

To fix these problems, instead fo modifying the binary, we decided to encrypt all raw strings directly in the source file before compile.

This requires a regular expression to match all string literals in source code. A base one is provided [here](https://stackoverflow.com/questions/41909225/regex-for-matching-c-string-constant).

to-be-continued...