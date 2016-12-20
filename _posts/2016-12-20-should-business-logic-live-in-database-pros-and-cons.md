---
layout: post
title: Should Business Logic Be in Database? Pros and Cons
permalink: blog/should-business-logic-live-in-database-pros-and-cons/
comments: True
excerpt_separator: <!--more-->
---

This debate has been labeled as the "Vietnam of Computer Engineering" and deservingly so. Even after a decade, it is still a highly contested issue. In the end, it really depends on the requirements and resources of the business that determines this. There are a lot of factor that involves such as, "is the team proficient in stored procedures?" or "are we permanently staying with this database vendor?" or "how will we test and scale this?" etc.

Here are some pros and cons, in my opinion, of why should business logic be in database?

### Pros

- centralized business logic
- be independent from application language, type, OS, etc.. node vs golang vs php vs ruby will not be an issue
- compared to applications, databases are less likely to need major refactorings
- it is often more performant to have business logic here
- stored procedures can reduce network traffic since you avoid multiple requests

### Cons

- database vendor lock-in, especially that major databases expand SQL standards or do not follow it
- very difficult to code-reuse
- it is much more difficult and more expensive to scale database layer horizontally
- source control is more difficult to do properly with stored procedures

### Conclusion

I believe that core business logic should live in the layer that is most scalable, testable, debuggable and versionable. Putting core business logic in the database makes it very difficult, and very expensive to fulfill those requirements.

<!--more-->

80% of your data access requirements would be simple CRUD and can safely and conveniently live in Application Layer, while 20% of the requirements which may require complex SQL queries or mass data manipulation should probably live in the database, especially if performance is a big concern.

What should live in the database? Logic that ensures data-integrity and relationships, auditing. This is probably limited to foreign key and check constraints, calculated columns, triggers and some views.
