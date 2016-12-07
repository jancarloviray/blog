---
layout: post
title: ReactJS Overview
permalink: blog/reactjs-overview/
comments: True
---

One unique selling points is that ReactJS can also be rendered server side and can work with client/server inter-operably.

## Why React?

render() is just regular javascript. You get full power of JS in your rendering code and not constrained by what your templating language is capable of. You can use map, reduce, filter, groupBy and etc.

It is fast (as fast or even faster than native DOM).

You don't have to give up flexibility for simplicity. You can have

With Flux, it pushes towards unidirectional flow, reducing complexity and increasing predictability.

I like how the jsx and component (not business!) logic is in just one file.

"Suffice to say, that in many cases React is much faster than angular. Develop with angular enough, and you will run into situations where the dirty-checking digest cycle is too slow for the requirements of your application. For these cases, a rather painless solution is to use ngReact. ngReact allows you to include React components inside of an angular application. The first React component I ever built was a re-write of a 20-month calendar widget originally written in angular. The rewrite took about a day which is a testament to the rather easy learning curve of React and the simplicity of ngReact. I didn’t even attempt to do any shouldComponentUpdate() optimizations in the React component and it was several times faster than it’s angular counterpart." - https://medium.com/@gilbox/how-can-react-and-flux-help-us-create-better-stronger-faster-angular-applications-639247898fb

## components

- are the smallest, yet most fundamental part of React. They are similar to widgets and modules.
- when building in React, you need to think of it in terms of the smallest possible components you can define.

## lifecycle of components

### Mounting

- componentWillMount: invoked once, on both client and server, before rendering occurs.
- componentDidMount: invoked once only on the client (not on the server), immediately after the initail rendering occurs. At this point in the lifecycle, the component has a DOM representation which you can access via `this.getDOMNode()`. If you want to integrate other JS frameworks, set timers or intervals or send AJAX requests, perform those operations in this method.

### Updating

- componentWillReceiveProps: invoked when a component is receiving new props. This method is not called for the initial render. Use this as an opportunity to react to a prop transition before render() is called by updateing the state using this.setState().
- shouldComponentUpdate: invoked before rendering when new props or state are being received. This method is not called for the initial render or when `forceUpdate` is used. Use this as an opportunity to `return false` when you’re certain that the transition to the new props and state will not require a component update. If this returns false, then render() will be skipped until the next state change. This is a great place to do performance improvements if needed. ie:
`shouldComponentUpdate: function(nextProps, nextState){ return nextProps.id !== this.props.id; }`
- componentWillUpdate: invoked immediately before rendering when new props or state are being received. This method is not called for the initial render. Use this as an opportunity to perform preparation before an update occurs.

### Unmounting

- componentWillUnmount: immediately invoked before a component is unmounted from the DOM. Perform any necessary cleanup in this method, such as invalidating timers or cleaning up any DOM elements that were created in `componentDidMount`

## props vs state

- both props and state are plain JS objects
- both props and state changes trigger a render update
- both props and state are deterministic. If your component generates different outputs for the same combination of props and state then you’re doing something wrong.
- If a component needs to alter one of its attributes at some point in time, that attribute should be part of its state, otherwise it should be a prop for that component.

| | props | state |
|---|---|---|
| can get initial value from parent component?  | yes | yes |
| can be changed by parent component? | yes | no |
| can set default values inside component? | yes | yes |
| can change inside component? | no | yes |
| can set initial value for child components? | yes | yes |
| can change in child components? | yes | no |

## props

- props are somewhat like a component’s configuration/options.
- a component cannot change its props, BUT it is responsible for putting together the props of its child components.
- Props are handed down from a component’s parent and cannot be changed in any way from the component itself.
- As the application gets bigger, it becomes hard to remember what types and if a certain property is required or not. Make sure you use the `propTypes` feature.
- To spend less time debugging later on, make sure you write strict propTypes from the start.
- The style attribute doesn’t accept strings. You can specify JS objects with keys that correspond to the camelCase CSS property names.
- Events can be assigned using attributes like onClick and onChange

### set property types

```
propTypes: {
    name: React.PropTypes.string,
    visible: React.PropTypes.bool.isRequired,
    before: function(properties, propertyName, componentName){
        if(!(propertyName in properties)){ throw Error(‘Property `before` must be set.’)}
    }
}
```

### set default properties

```
getDefaultProps: function(){
    return { ‘firstName’:’JC’, ‘lastName’:’Viray’ }
},
render: function(){
    return <div> hello {this.props.firstName} </div>
}
```

## state

- `state` is created within the component itself.
- A `state` should be considered private data.
- A state starts with a default value when a component mounts and then suffers from mutations in time (mostly generated from user events).
- A state is optional. Since it increases complexity and reduces predictability, a component without state is preferable.  Try to avoid having states as much as possible
- A state should be serializable at one point in time - a snapshot.
- A compoenent manages its own state internally, but besides setting an initial state, has no business fiddling with the state of its children.
- Unlike props, state can be changed from within a component
- The general consensus in the software engineering world today is to avoid mutable state. This is the first principle of good React component design. Avoid placing state inside components if at all possible.
- `state` should only be seen inside of the component definition. You as the developer of the component should be the only one who know what state your component needs and the correct data types which should be accepted/provided
- note that the use of state should be as limited as it can be. When you use state, you run the risk of introducing a number of errors in the behavior and rendering of your components.

### set initial state

```
getInitialState: function(){
    return { ‘firstName’:’JC’, ‘lastName’:’Viray’ }
}
```

### updating state

```
var Component = React.createClass({
    getInitialState: function(){
        return { name: ‘Chris’};
    },
    handleClick: function(){
        this.setState({ name: ‘Bob’ });
    },
    render: function(){
        return <div onClick={this.handleClick}> hello {this.state.name} </div>
    }
});

React.renderComponent(
    <InterfaceComponent />,
    document.body
)
```

## Thinking in React

### Step 1: Break the UI into a Component Hierarchy

First thing you’ll want to do is to draw boxes around every component and subcomponent in the mock and give them all names. Use Single Responsibility Principle, meaning that a component should ideally only do one thing. If it ends up growing it should be decomposed into smaller subcomponents.

### Step 2: Build a static version in React

Take your data model and build the UI with no interactivity. This usually requires a lot of typing and no thinking.

To build a static version of your app, you’ll want to build components that reuse other components and pass data using props. props are a way of passing data from parent to child. Here, you’re not going to use state at all. State is reserved only for interactivity, that is, data that changes over time.

At this point, you’ll have reusable components, props that make use of your mock data model and will only have the render() methods

### Step 3: Identify the minimal but complete representation of UI state

To make your UI interactive, you need to be able to trigger changes to your underlying data model. Here, you will use state.

To build your app correctly, you first need to think of the minimal set of mutable state that your app needs.

NOTE: Figure out what the absolute minimal representation of the state of your application needs to be and compute everything else you need on-demand. For example: if you are building a “todo list”, just keep an array of the todo items around; do not keep a separate state variable for the count! Instead, when you want to render the todo count, simply take the length of the todo items array. REMEMBER TO USE STATE AS LEAST AS YOU CAN!

In our example application, here’s our data:
- the original list of products (prop... is passed is so not a state)
- the search text the user has entered (state... changes over time and can’t be computed)
- the value of the checkbox (state... changes over time and can’t be computed)
- the filtered list of products (neither since it can be computed)

### Step 4: Identify where your state should live.

Now that you’ve identified what the minimal set of app state is, you need to identify which components mutates or owns the state.

Remember, React is all about ‘one-way’ data flow down the component hierarchy. It may not be immediately clear which component should own the state.

For each piece of state in your application, follow these steps to figure it out:
- identify every component that renders something based on that state
- find a common owner component (a single component above all the components that need the state in the hierarchy)
- either the common owner or another component higher up in the hierarchy should own the state.
- if you can’t find a component where it makes sense to own the state, create a new component simply for holding the state and add it somewhere in the hierarchy above the common owner component

### Step 5: Add inverse data flow

Now it’s time to support data flowing the other way. Best way is for the parent to pass a callback to onChange handler of child to which the child would pass information to.

## Quick intro about Flux

“The name “Flux” is a pretentious barrier to understanding. There is no such thing as Flux. Flux is a concept, not a library. Well, there is a library, sort of.”

“Flux is more of a pattern than a framework”

“The concept “Flux” is simply that your view triggers an event (say, after user types a name in a text field), that event updates a model, then the model triggers an event, and the view responds to that model’s event by re-rendering with the latest data. That’s it.”
