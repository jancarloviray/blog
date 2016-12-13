---
layout: post
title: Better Git Commit Titles
permalink: blog/better-git-commit-titles/
comments: True
---

For almost a year already, I have been establishing commit messages following a semantic way and it has improved code reviews and just skimming through code history. Inspired by [angular](https://github.com/angular/angular) commit messages, I adopted their [Commit Message Guidelines](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-guidelines). An example would be:<br/><br/>`fix(release): need to depend on latest rxjs and zone.js`<br/>`docs(changelog): update change log to beta.5`.<br/><br/>The great thing about this is is you see a **type** and **scope**, which immediately preps you a context instead of skimming through the folder structure and codes first. Try it out in your workflow and let me know how it works for you! Here's an excerpt of types from the documentation:<br/><br/>
**feat:** A new feature<br/>
**fix:** A bug fix<br/>
**docs:** Documentation only changes<br/>
**style:** Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)<br/>
**refactor:** A code change that neither fixes a bug nor adds a feature<br/>
**perf:** A code change that improves performance<br/>
**test:** Adding missing tests or correcting existing tests<br/>
**build:** Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)<br/>
**ci:** Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs)<br/>
**chore:** Other changes that don't modify src or test files<br/>

## Commit Message Format

Each commit message consists of a header, a body and a footer. The header has a special format that includes a type, a scope and a subject:
```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
The header is mandatory and the scope of the header is optional.
```
Any line of the commit message cannot be longer 100 characters! This allows the message to be easier to read on GitHub as well as in various git tools.
