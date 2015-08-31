---
layout: post
title: State is an Anti-Pattern
permalink: blog/state-is-an-anti-pattern
comments: True
---

As much as possible, do not use state at all. According to [this comment](https://www.reddit.com/r/reactjs/comments/3bjdoe/state_is_an_antipattern/), state should never have been in the library in the first place. It seems that if Flux was introduced in conjunction with React initially, state would not even exist. It seems that state was used to allow react to function by itself without flux. *If a React component must have side-effects, it must use Flux actions* instead of having state. Components ideally should have no state at all.
