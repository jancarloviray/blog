---
layout: post
title: React Props in State an Anti-Pattern
permalink: blog/react-props-in-state-anti-pattern/
comments: True
---

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
