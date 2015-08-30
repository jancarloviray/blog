---
layout: post
title: React.js Best Practices Compilation
permalink: blog/react-best-practices-compilation
comments: True
---

React.js lessons learned and some opinionated best practices. This is an ongoing compilation and in-progress work of everything I have learned from developing in React.js

## Basic Component Organisation

```
React.createClass({
    propTypes: {},
    mixins: [],

    getInitialState(){},
    getDefaultProps(){},

    componentWillMount(){},
    componentWillReceiveProps(){},
    // Store removeChangeListener
    componentWillUnmount(){},
    // Third-party code initialization here...
    // Store addChangeListener here...
    componentDidMount(){},

    _parseData(){},
    _onSelect(){},
    _onChange(){},

    render(){}
});
```

## Flux Pattern Tips

### Stores

- Stores don't have to just represent data. They can also represent *application state*, whether modals are shown/hidden, or if user is online/offline, etc.
- Stores can and probably will need to use data from other stores. Use waitFor method in dispatcher.
- Stores should not contain the logic to fetch themselves on a server; it is better to use DAO (data access objects) that are thinly wrapped API objects to get the data.

### Actions

- Actions should be split into two types: view actions and server actions. This will separate user interaction such as clicking a button from retrieving data.
- Actions should be fire and forget and must not have callbacks. *If you need to respond to the result of an action, you should be listening for a **completion or error event*** This enforces data being kept in the store and not on a component.

## Personal Brainstorm

Focus on the public actions. Simplicity is the key. Do not put implementation as the forefront.

With react, though it promotes a more architecturally sound applications, it does promote unmanageable code due to coupling of html and javascript. HTML files grow big, mainly because of the verbosity of HTML, and the logic and module interface easily will get lost with the process.

Use **spread operators** to have the child inherit the parents' props. Note that the specification order is important here. Later attributes override previous ones.

```
var props = { foo: 'default' };
var component = <Component {...props} foo={'override'} />
```

**use namespaced components to make components simpler and easier to read.**

```
var MyFormComponent = React.createClass({});

MyFormComponent.Row = React.createClass({});
MyFormComponent.Label = React.createClass({});
MyFormComponent.Input = React.createClass({});

//...

var Form = MyFormComponent;
var App = (
    <Form>
        <Form.Row>
            <Form.Label />
            <Form.Input />
        </Form.Row>
    </Form>
);
```

## State

**The issue of duplicating props to state**

This is a somewhat complicated thing. What goes into state gets more complicated due to the eventuality of having to duplicate props... so that it can be editable in the form.

**What about how setState not being able to handle complex objects, making { data:{}, errors:{}} not be a viable architecture. What to do about forms now?**

There has to be a better way... React.addons.update should not be used. It is unwieldy and a hack.

The state should only be a one-level object. It should not have grandchildren, otherwise it will get unmaintainable. *It is better to have more react modules than unwieldy code.*

## Opinionated Conventions

Iterate inline for JSX transformations. Too much moving of cursors and etc reduces the flow. But sections could be divided into variables.

Components should be as dump as possible. All logic should live in stores or services. The only things that components must consist of are Actions, event handlers, and the HTML. Components really are just dumb views.

Now, what about HUGE htmls and forms? How should these things be divided?

## Exploring Different Architectures

**Separate Layout Code as Props**

- I love the className of top element be same as the modulename.. it becomes better to look through when debugging in browser console.
- I like the requiring of style on this module. Style convention is good.. very nice BEM format.

```
require('./DefaultLayout.less');
var DefaultLayout = React.createClass({
    propTypes: {
        header: React.PropTypes.node,
        title: React.PropTypes.node,
        content: React.PropTypes.node
    },
    render: function(){
        return (
            <div className="DefaultLayout">
                <div className="DefaultLayout-header navbar navbar-default">
                    <div className="container">{this.props.header}</div>
                </div>div>
                // etc...
            </div>
        );
    }
});
```

**Separate Render components. Say 'NO' to big render functions**

```
render: function(){
    // ...
    return (
        <div className={classes}>
            <div className="Item-title">
                {this.renderTags()}
                {this.renderDate()}
            </div>
        </div>
    );
},
renderTags: function(){
    return (
        <div></div>
    );
},
renderDate: function(){
    return (
        <div></div>
    );
}
```
