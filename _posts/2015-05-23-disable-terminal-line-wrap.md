---
layout: post
title: Disable Terminal Line Wrap
permalink: blog/disable-terminal-line-wrap/
comments: True
---

I’ve gotten a bit annoyed with terminal wrapping long lines. I’ve been using `less` for a long time already, but did not know the `-S` option, or “--chop-long-lines”.

```
this_command_produces_wide_output | less -S
```

Bam! Use horizontal arrow keys to view the rest of the line.
