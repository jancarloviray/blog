---
layout: post
title: React and Flux Tips
permalink: blog/react-flux-tips
comments: True
---

Being that React is all about dynamic data and being that it handles performance very well, at the cost of more boilerplate code to write, I think it is important to focus on simplicity, very small modules, very small code and dynamic code generation. Hard-coding should be limited and be of a thing of a past.

Some tips:
- Use ES6 heavily
- Do not combine layout and domain components
- Remember container components. This is an important design pattern.
- Use {_.map(...)}
- Use spread operators (...this.props)
- The View should contain NO business logic at all.
- React code can be beautiful with a bit more intelligent planning.
- Use private functions and put outside of the react class.
- Each handler really should just contain one line - and that is a call to the Action
- Conventions should be heavily placed. Underscored functions should primarily be just helper functions and if they are not used from within the view.
- Always have just a single "source of truth" which should never be duplicated
- The state of a component should not depend on the props passed on. Why? Because if props change, the view will not change. The getInitialState only runs when the component is mounted.

## Examples of Good React Components

- [https://github.com/callemall/material-ui/blob/master/src/date-picker/calendar.jsx](https://github.com/callemall/material-ui/blob/master/src/date-picker/calendar.jsx)
- [flux todo example](https://facebook.github.io/flux/docs/todo-list.html)

## State is an anti-pattern

As much as possible, do not use state at all. According to [this comment](https://www.reddit.com/r/reactjs/comments/3bjdoe/state_is_an_antipattern/), state should never have been in the library in the first place. It seems that if Flux was introduced in conjunction with React initially, state would not even exist. It seems that state was used to allow react to function by itself without flux. *If a React component must have side-effects, it must use Flux actions* instead of having state. Components ideally should have no state at all.

## State and Props

Props and state are the raw data that the HTML output derives from. You could say that props + state is the input data for the render function of a Component. Note that both are deterministic, meaning *if your component generates different outputs for the same combination of props and state, then you're doing something wrong*

**If a Component needs to alter one of its attributes at some point in time, that attribute should be part of its *state* otherwise it should just be a *prop* for that Component**

**PROPS**

Props are more like configurations or options for the component. It is often something passed down from its parents. It is also common to pass down callback functions through props.

When you want to change a prop value, the parent component should react on this, and provide new props to the child component. Properties are a mechanism for the outside components to configure your component (unlike state, which is just for your internal component).

**STATE**

It suffers from mutations in time (mostly generated from user events). Key here is that, *it is a serializable representation of one point in time - a snapshot*. State only manages its own state. It should not be managing the state of its children.

| 												| props | state |
| Can get initial value from parent Component? 	| yes	| yes 	|
| Can be changed by parent Component? 			| yes 	| no 	|
| Can set default values inside Components?		| yes	| yes 	|
| Can change inside Component?					| no 	| yes	|
| Can set initial value for child Components? 	| yes 	| yes 	|
| Can change in child components? 				| yes 	| no 	|

### Props in initial state: an anti-pattern

**Passing the initial state to a component as a prop is an anti-pattern because the `getInitialState` method is only called the first time the component renders - and never called after that.** This means that if you re-render the parent, while passing a *different* value as a prop, the component will not update the UI because it will keep the state from the first time it was rendered. This will make the application very prone to errors.

**Solution? Make the components stateless. They are easier to test because they render an output based on an input. Have the parent component contain the passed-in data as its own *state*, then if the state changes, it re-renders its children, while passing in everything they need through `props`.**

What does this look like in React?

The parent component (or so called "Controller View") would get all of the data through an ajax call (use flux pattern) and place it in `getInitialState`.

Another solutions is to send in "handlers" to the children, then these parent handlers would do a setState once they are invoked by their children, therefore doing re-rendering of the UI. Here's a good example at StackOverflow: [reactjs-two-components-communicating](http://stackoverflow.com/questions/21285923/reactjs-two-components-communicating?rq=1).

Best solution is to have a "Container" or "ControllerView" passing down the handler to the children components...

```
var ListContainer = React.createClass({
	handleFilterUpdate(filterValue){
		this.setState({
			nameFilter: filterValue
		});
	}	,
	render(){
		return (
			<div>
				<Filters updateFilter={this.handleFilterUpdate}/>
				<List items={displayedItems}/>
			</div>
		);
	}
});

var Filters = React.createClass({
	handleFilterChange(){
		var value = this.refs.filterInput.getDOMNode().value;
		this.props.updateFilter(value);
	},
	render(){
		return (
			<div><input ... ref="filterInput" onChange={this.handleFilterChange} /></div>
		)
	}
});
```

```
// this is not good
getInitialState: function(){
	return {
		text: this.props.text
	}
}
```

Another problem with this is that *the parent that passed the `text` property might be expecting that it contains the latest value, but it does not.*

Using props, passed down from parent to generate state in `getInitialState` often leads to duplication of "source of truth", which is where the real data is.

## React Component Methods

**getInitialState**

Invoked once, before the component is mounted.

**getDefaultProps**

Invoked once and cached when the class is created. Note that values will be set here if not specified by the parent component. *Be aware that any complex objects returned by getDefaultProps will be shared across instances, not copied*

## React Component Lifecycle

**componentWillMount**

Invoked once before the *initial render*. If you call setState within this method, render will see the updated state and will be executed only once despite the state change. If you call setState here, no re-render would be invoked. Remember, this is called just once within the component's lifetime. So, it's best to call in references to your services and etc here.

**componentDidMount**

Invoked once after the *initial render*. Because you now have access to the virtual DOM, you can now call *React.findDOMNode(this)*. If you wanted to make AJAX requests, do it here too. AJAX requests and third-party dom-manipulation should occur here. Basically, this is where you integrate with other frameworks and setTimeouts and setIntervals as well as any AJAX requests

**componentWillReceiveProps**

This is not called on initial render, but instead whenever there is a change to props. Use this method as a way to react to prop changes before another render() is called by updating the state with setState.

Use this is an opportunity to react to a prop transition before render is called by updating the state using *this.setState*

**componentWillUnmount**

This is invoked immediately before a component is unmounted from the DOM. This is where you do cleanup and de-initializing third-party components.

## Flux Overview

**Actions** are helper methods that facilitate passing data to the Dispatcher.

**Dispatcher** receives actions and broadcasts payloads to registered callbacks.

**Stores** are containers for application state and logic that have callbacks registered to the dispatcher.

**Controller Views** are react components that grab the state from stores and pass it down via props to child components.

## Flux Flow

**VIEWS**

Requires *stores* and *actions*.

When a component mounts, it gets its initial state from a store, then sets up a listener so that when the Store emits a change event, it will go ahead and refetch the data from the store so it can update its own internal state.

A component also invokes Actions.

```javascript
var List = React.createClass({
	getInitialState(){
		return {
			list: TodoStore.getList()
		}
	},
	componentDidMount(){
		TodoStore.addChangeListener(this._onChange);
	},
	componentWillUnmount(){
		TodoStore.removeChangeListener(this._onChange);
	},
	handleAddItem(newItem){
		TodoActions.addItem(newItem);
	},
	handleRemoveItem(index){
		TodoActions.removeItem(index);
	},
	_onChange(){
		this.setState({
			list: TodoStore.getList()
		});
	},
	render(){}
});
```

**ACTIONS**

requires *dispatcher* and *constants*

```javascript
var TodoActions = {
	addItem(item){
		AppDispatcher.handleAction({
			actionType: appConstants.ADD_ITEM,
			data: item
		});
	},
	removeItem(index){
		AppDispatcher.handleAction({
			actionType: appConstants.REMOVE_ITEM,
			data: index
		})
	}
};

var appConstants = {
	ADD_ITEM: 'ADD_ITEM',
	REMOVE_ITEM: 'REMOVE_ITEM'}
```

**DISPATCHER**

The Dispatcher is the hub of your app. Every time you want to change the data that lives in a store,  you must go through the Dispatcher. An action invokes a method on the dispatcher *which emits an event along with any payload or data that needs to go to the store*.

**STORES**

The store listens for certain events and when it hears an event (from the Dispatcher) its listening for, it modifies its own internal data then emits a 'change' event which a component listens for and then the component updates itself, bringing it to a full circle.

The store contains four parts:

- First, the store contains the actual "model" or *"collection"* or "data store".

```javascript
var _store = { list: [] };
```

- Second, the store contains *setter methods*, which will be the ONLY interface for manipulating the data of our store (above).

```javascript
var addItem = function(item){ _store.list.push(item); }
var removeItem = function(index){ _store.list.splice(index, 1); }
```

- Third, the store itself, which is really just a collection of getter functions in order to access the data in the first step along with a few emitter/change listener helpers.

```javascript
var todoStore = objectAssign({}, EventEmitter.prototype, {
	addChangeListener: function(cb){
		this.on(CHANGE_EVENT, cb);
	},
	removeChangeListener: function(cb){
		this.removeListener(CHANGE_EVENT, cb);
	},
	getList: function(){
		return _store.list;
	}
});
```

- Fourth, we need to use our Dispatcher to listen for certain events and when we hear those events, invoke the setter functions above.

```javascript
AppDispatcher.register(function(payload){
	var action = payload.action;
	switch(actions.actionType){
		case appConstants.ADD_ITEM:
			addItem(action.data);
			todoStore.emit(CHANGE_EVENT);
			break;
		case appConstants.REMOVE_ITEM:
			removeItem(action.data);
			todoStore.emit(CHANGE_EVENT);
			break;
		default:
			return true;
	}
});
```








