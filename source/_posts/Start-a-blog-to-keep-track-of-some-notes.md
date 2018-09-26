---
title: Start a blog to keep track of some notes.
date: 2018-07-24 23:32:57
tags: [Misc, hexo]
---
This is my github page, a place where I take notes on the stuff I learned from the courses I took and through-out my work.

Hexo is pretty easy to set up.

One problem I encounter is that when I run:
``` bash
$ hexo server
```

I got the following error:
``` bash
ERROR Local hexo not found
```

It turns out it's because my node version is too low...

Everything works after I updated my node version.

When trying to deploy after adding a new post, the github page does not update.

It seems that a `hexo clean` is needed before running `hexo depoly`.

[A useful link](https://gist.github.com/btfak/18938572f5df000ebe06fbd1872e4e39)

Process of creating a blog:
- `cd <path to blog/blog_init>`
- `hexo new post <title>`
- `hexo generate`
- Use `hexo server` to check the generated page
- `hexo deploy` to deploy the change to github