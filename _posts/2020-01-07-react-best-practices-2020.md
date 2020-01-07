---
layout: post
title: React Best Practices 2020
permalink: blog/react-best-practices-2020/
comments: True
excerpt_separator: <!--more-->
---

Hopefully you have enjoyed some of the articles I posted here throughout the years. I geared it more as a personal reference for myself and yet, it has gained a good amount of views. At this point, I will be refocusing and rebranding this blog into topics like entrepreneurship, product creation and growth, as this is now my main focus. Until then, this blog will be under construction. 

### Domain Driven Design Structure instead of Type Driven Design
- Organizing projects by function or type works okay in smaller application but as it grows, hundreds of files get created. It then becomes difficult to find dependencies if project is not modularized. Example: /redux/{actions, middleware, reducers, utils}. Instead, it is better to organize by domain.

### Use Function-Based components instead of State-less Class-Based ones

- There are some performance optimizations React does when it comes to function-based components. Avoid using class-based components unless you need lifecycle methods.

### Keep components small and tied to a specific function - make it predicable and testable

-	It’s easier to understand, update, implement performance optimizations, test and helps promote reusability
-	Big components tend to become difficult to maintain, debug, test and refactor
-	Components with over 200+ lines of code is a big sign that it should be broken up

### DRY your code (Don’t Repeat Yourself)

- If you find any patterns or similarities it is possible you are repeating code.

### Separate stateful data-related logic from presentation logic.

- Mixing the two can lead to complexity. Keep responsibilities small and isolated.

### All files related to any one component should be in a single folder

- Keep all files relating to any one component in a single folder, including styling files. It makes hierarchy easy to understand and makes refactoring and navigating codebases easier.

### Strict Linting

- Linting will help developers be aware of best and common practices and develop good habits. This also helps unify the style of the codebase.

### Do not use Index or Value as a Key

- Key is what React uses to identify elements. If you push an item to the list and the key is same, React assumes that the element represents the same component as before even if it is not. This may break your application or display wrong data
- Each key should be permanent and unique. Generating random numbers is worse because it will re-render the components every time.
- Solution according to official docs is to pass in key in the mapped ListItem component but do not return it

### bind() and arrow functions

- .bind() creates a new function each time it is run. This means a new function is being created every time the render function executes. This has performance implications as the app grows.
- Arrow function has the same concern since it’s an anonymous function and can’t be compared.
- Solution? Bind function in constructor lifecycle.

### Do not use anonymous functions as handlers

- Anonymous functions cannot be compared. This means that React will always re-render. Pass references only

### Only use setState to mutate state. Do not mutate state directly

- If you mutate the state directly, the component will not be re-rendered and the changes will not be reflected. This is because the state is compared shallowly. You should always use setState for changing the value of the state.

### Do not use Props as State

- The constructor is only called once. The state won’t be updated if changed. You can this pattern only if you want to seed the state. Otherwise, this leads duplication of source of truth. Use the props directly instead.

### Use React.Fragments to Avoid Unnecessary Wrappers Populating the DOM

- Instead of using unnecessary wrappers which populates the DOM such as div, span, etc use React.Fragment or use the more concise syntax of <></>

### Avoid spreading unnecessary props on DOM elements

- Instead of doing {...props} be more explicit otherwise it will populate attributes on DOM elements

### Avoid using inline styles

- Separate style from component. This will also cause bugs in the future due to CSS specificity and prevent theming, branding, etc.
- Additionally, this causes React to always re-render components regardless if data has changed

### Use Reselect to help prevent re-renders on unchanged inputs through memoization

- Alternatively can also use React.memo. Might be better to use that.

### Use Web Workers for CPU extensive tasks

- Web Workers make it possible to run operations in the background thread, separate from the main thread. It helps prevent UI blocking or slowing

### Use Optional Chaining

- Prevents crashes when accessing a child property without guard checks

### Do not use export default - be explicit instead

- This makes refactoring harder on larger files and also slows down intellisense on larger codebases. Seems like a minor code styling preference, but it starts becoming a problem on larger codebases

### Use shouldComponentUpdate to compensate for shallow comparison on arrays, functions, objects, etc

- Side Note: can be used to fix some of the major performance issues in a hacky but temporary way

### Virtualize long lists

- If you have long lists, rendering all elements including hidden ones can cause performance problems and stuttering. Use utilities like react-window to virtualize long lists.

### Big Refactors / Improvements

- Use React.Memo to cache components. Use it for functional components.
- Memoized functions are faster because if called with the same values as the previous call, it will fetch return values from cache.
- If you have several components that rarely change state, you should consider caching them. It’s as easy as export default React.memo(ProfileView)
- Use Reselect to cache Redux selectors
- CSS Architecture
- Naming Consistency
- Structure Consistency
- Lazy Loading of Components (later)
