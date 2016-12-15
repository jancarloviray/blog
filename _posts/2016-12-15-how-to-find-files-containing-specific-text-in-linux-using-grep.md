---
layout: post
title: How to Find Files Containing Specific Text in Linux Using Grep
permalink: blog/how-to-find-files-containing-specific-text-in-linux-using-grep/
comments: True
excerpt_separator: <!--more-->
---

How to find files containing a specific text recursively in Unix / Linux system? I typically use `grep` like this: 

```shell
grep -rnw "pattern" /path/to/file
```

Helpful options:

- `-r` means recursively search
- `-n` adds line number
- `-w` matches the whole word, so remove it if you want partial match
- `-i` for case insensitive search, but it's a bit slow. Note that grep is case insensitive by default.
