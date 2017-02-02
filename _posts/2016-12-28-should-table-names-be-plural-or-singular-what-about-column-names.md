---
layout: post
title: Should Table Names be Plural or Singular? What about Column Names?
permalink: blog/should-table-names-be-plural-or-singular-what-about-column-names/
comments: True
excerpt_separator: <!--more-->
---

Naming in programming is hard sometimes. I typically think about the future of the app, some "what ifs", conventions and if it truly gives a good context for other developers or users. In the end, *as long as everyone involved in the project is consistent and better yet, have things documented, then that typically outweighs hardlined rules.* But here is a simple guide to help make your decisions.

<!--more-->

With regards to database table and column names, I lean towards a certain convention:

- A database table is a set, and every row is an object. If you were making an array, wouldn't you pluralize your variable name? This is why I believe **table names should be plural**
- A table's column is an element. In a sense, it is a property name of an object that contains a scalar data element (unless you're using arrays in postgres). If you were creating a class with properties that contain basic elements (string, int, bool, etc), wouldn't you make your property name singular? This is why it makes more sense that **column names should be singular** unless you're using arrays.

But of course, many would argue why and why not. There are factors such as convenience, simplicity or just plain aesthetic. Some go into more theoretical or technical. **In the end, as long as you are consistent with the team and have conventions documented, that outweights right vs wrong.**
