---
layout: post
title: How to Merge Multiple Commits into one Git Commit?
permalink: blog/how-to-merge-multiple-commits-into-one-git-commit/
comments: True
excerpt_separator: <!--more-->
---

To do this, we use the command `git rebase`. Typically, it is used to:

- Edit previous commit messages
- Combine multiple commits into one
- Delete or revert commits that are no longer necessary

Let's work through an example.

<!--more-->

Let's say we already have an existing repository with a lot of commits. First, check your commit log:

```shell
git log --oneline
```

Now, let's say we want to merge last 4 commits. Run `git rebase` with `-i` which means interactive and `HEAD~4` which means to look at last 4 commits

```shell
git rebase -i HEAD~4
```

Something like this should show in your editor:

```
pick 43432432 my commit message to preserve
pick 43132132 my other commit message
pick 12353434 some commit message
pick 64554234 update something
....
```

If you'd like to squash all commits into "my commit message to preserve", then change it into this:

```
pick 43432432 my commit message to preserve
f 43132132 my other commit message
f 12353434 some commit message
f 64554234 update something
....
```

Make sure you read the instructions git added as comments in your editor. Once satisfied, push to remote repository:

```shell
git pull origin master
git push origin master
```

Done? Yes, but a piece of advice here: this is considered a bad practice as it squashes commits and makes it difficult for everyone else that are using the repository. **Do this at your own risk.**
