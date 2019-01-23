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

- [ ] Need to add the working sample.

There are some benifits of using this solution:

* We can encode the string literal into anything we want. We don't have to make sure that the encrypted string has the same length as the origional string.

* We can choose the encryption and decryption algorithm we want.

One problem for this approch is that the string we matched using regex can not handle escape characters correctly. All the escape characters are being interpreted literially when doing string encryption.

For example, `\n` will be treat as two characters, `\` and `n` when doing encryption, instead of being treated as ASCII character `LF` (10 in ASCII table). 

As a result, when we decrypt the string, we do not get the `LF` character we want, instead, we get `\` and `n` which is not correct.

To fix this, we have to post-process the string matched by our regex into a vector of characters.

During post-process, we convert all the escape sequences into their corresponding ASCII characters. Then, when decrypt, we will get the correct output.

Another problem is that since we have to write the encrypted characters back into our source files, some of the encrypted characters are not printable nor human readable. So, to write them to the source file, we have to write the hexidecimal representation of the characters.

For example, assume `\n` is encrypted into character sequence `abc`. Then, in the source file, `\n` is then replaced with `\x61\x62\x63`.

## Replace `__FILE__`, `__FUNCTION__` and `__PRETTY_FUNCTION__` 

Since `__FILE__`, `__FUNCTION__` and `__PRETTY_FUNCTION__` will be replace by pre-processor to string literals, it will be added to the `.rodata` section of the binary. 

To prevent this, we can write our own script to pre-process the source files to replace all these macros to empty string. 

For the `__FILE__` macro, we can actually easily replace it with the actual file name string when we do our pre-process. But for `__FUNCTION__` and `__PRETTY_FUNCTION__`, it's harder for a simple pre-process script to actually figure out what they should be replaced with.

This pre-process should be done before we start extracting and encrypting strings. 

## Do not use `virtual` methods for classes that you want to hide name

If a class contains `virtual` methods, its name will be put into `rodata._ZTS<class_name>` section in the binary. Because this information is needed for `dynamic_cast` and exceptions. 

If the code base does not use RTTI features, `-fno-rtti` can be added to the compiler flag to prevent these information from being generated.

Some useful links on `rodata._ZTI` and `rodata._ZTS` sections:

* [https://shaharmike.com/cpp/vtable-part1/](https://shaharmike.com/cpp/vtable-part1/)
* [ftp://gcc.gnu.org/pub/gcc/summit/2003/Getting%20the%20Best%20from%20G++.pdf](ftp://gcc.gnu.org/pub/gcc/summit/2003/Getting%20the%20Best%20from%20G++.pdf)
* [https://stackoverflow.com/questions/49381011/what-does-ztv-zts-zti-mean-in-the-result-of-gdb-x-nfu-vtable-address](https://stackoverflow.com/questions/49381011/what-does-ztv-zts-zti-mean-in-the-result-of-gdb-x-nfu-vtable-address)

## Try to remove `-rdynamic` flag

If our final target is an executable, we can probably remove the `-rdynamic` flag. 

This will prevent a lot of symbols from being exported.

* [https://stackoverflow.com/questions/36692315/what-exactly-does-rdynamic-do-and-when-exactly-is-it-needed](https://stackoverflow.com/questions/36692315/what-exactly-does-rdynamic-do-and-when-exactly-is-it-needed)

## Try adding `-fvisibility=hidden` and `--exclude-libs,ALL`

`-fvisibility=hidden` makes all symbols hidden by default. 

`--exclude-libs,ALL` excludes symbols in all archive libraries from automatic export.

* [https://stackoverflow.com/questions/3570355/c-fvisibility-hidden-fvisibility-inlines-hidden](https://stackoverflow.com/questions/3570355/c-fvisibility-hidden-fvisibility-inlines-hidden)
* [https://sourceware.org/binutils/docs/ld/Options.html](https://sourceware.org/binutils/docs/ld/Options.html)

## Add `-DNDEBUG` as compiler flag

If `-DNDEBUG` is added, all the `assert` s are stripped away from source code.  

## Use `strip` command to get rid of unneeded sections in binary

Use `strip` with `-s` and `--strip-unneeded` will strip away a lot of sections not needed by binary.

Also, `.note.ABI-tag`, `.note.gnu.build-id` and `.comment` sections can also be removed.

## Misc.

* If there are still visible strings left, use `objdump` and `readelf` on the object file that the string might come from, to locate which section does the string come from. 
