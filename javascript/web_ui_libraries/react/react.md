---
layout: page
title: "React"
permalink: /web_ui_libraries/react
---

[comment]: <> (TODO: Need to break this up and make the heading levels more consistent then fill in the section with my other notes.)

## JSX

### What is it?

JSX is a syntax extension of JavaScript that allows you to write HTML directly within Javascript.  This lets you use the full programmatic powers of JavaScript within HTML.

### How it works

Since JSX is not a valid JavaScript, JSX code must be compiled into JavaScript  The transpiler Babel is a popular tool for this.

### Syntax

To include JavaScript in HTML using JSX you include the code you want to be treated as JavaScript in curly braces.

`{'this is treaded as JavaScript code}`

To put comments inside JSX, you use the syntax `{/**/}`

JSX elements can be nested, but must return a single top level element.

Below is an example of valid JSX

```JSX
<div>
    <p>Paragraph One</p>
    <p>Paragraph Two</p>
    <p>Paragraph Three</p>
</div>
```

Below is an example of *invalid* JSX

```JSX
    <p>Paragraph One</p>
    <p>Paragraph Two</p>
    <p>Paragraph Three</p>
```

#### JSX differences from HTML

In JSX you can no longer use the world *class* to define HTML classes.  This is because class is a reserved word in JavaScript.  Instead of class JSX uses **className**.

The naming convention for all HTML attribute and event references in JSX become *camelCase*.  Fore example...
* onclick becomes onClick
* onchange becomes onChange

Self closing tag rules are different in JSX.  In HTML you can have certain tags such as <br> that do not require a closing tag since they never have any content.  In JSX any JSX element can be written with a self closing tag, and every element must be closed.

* The line break tag for example must alwasy be written as `<br />` in order to be valid JSX that can be transpiled.
* A `<div>` which may or may not contain content can be written in JSX as either `<div />` or `<div></div>`

## Components

### What are they?

Components are the core of React.  Everything in React is a component.

There are two ways to create a component.

The first is to use a Javascript function.  Defining a component this way creates a **stateless functional component**

```JSX
const DemoComponent = function(){
    return (
        <div className='customClass' />
    );
}
```

The second way is to use the ES6 `class` syntax.

```JSX
class Kitten extends React.Component {
    constructor(props){
        super(props);
    }

    render(){
        return (
            <h1>Hi!</h1>
        );
    }
}
```

### Component props

### Props with a stateless functional component

Its a standard practice to call this value props when dealing with stateless functional components.  You basically consider it as an argument to a function which returns JSX.  You can access the value of the argument in the function body.

```JSX
const Welcome = (props) => <h1>Hello, {props.user}!</h1>

<App>
    <Welcome user='Mark' />
</App>
```

Passing an array as a prop

```JSX
<ParentComponent>
    <ChildComponent colors={['green','blue','red']}>
</ParentComponent>
```

### Props with class components

Access component props using the syntax this.props.myprop

### Default props

You can assign default props to a component as a property on the component itself and React assigns the default prop if necessary.  This allows you to specify what a prop value should be if no value is explicitly provided.

`MyComponent.defaultProps = { location: 'San Francisco' }`

### You an use PropTypes to Define the props you expect.

React provide a type checking feature to verity that components receive props of the correct type.  For example you can use propTypes on your component to require the data to be of type array.  This will throw a useful warning when data is of any other type.  Its considered best practice to set propTypes when you know the type of a prop ahead of time.

`MyCoomponent.propTypes = { handleClick: PropTypes.func.isRequired }`

* func represents function
* Among the seven JavaScript primitive types, function and boolean (bool) are the only two that use unusual spelling.
* As of React v15.5.0 you need to import propTypes independently

## State

### What is state?

State consists of any data your application needs to know about tha can change over time.

You want your apps to respond to state changes and present an updated UI when necessary.

Stat is one of the most powerful features of components in React.  It allows you to track important data in your app and render a UI in response to changes in this data.  **If you data changes, your UI will change.**

### How?

You create state in a React component by declaring a state property on the components class in its constructor.  This initializes the component with state when it is created.  The state property must be set to a JavaScript object.

You have access to the sate object throughout the life of your component.  You an update it, render it in your UI, and pass it as props to a child components.

Note that you must create a class component by extending React.Component in order to create state like this.

React uses what is called a virtual DOM to keep track of changed behind the scenes.  When state data updates, it triggers a re-render of the components using the data (including child components that received data as a prop.)  React updates the actual DOM, but only where necessary.  This means you don't need to worry about updating the DOM.  You simply declare what the UI should look like.

Note that if you make a component stateful, no other components are aware of its state.  Its state is completely encapsulated (local to that component) unless you pass state data to a child component as props.  This notion of encapsulated state is very important because it allows you to write certain logic, then have that logic contained and isolated in one place in your code.

```JSX
class StatefulComponent extends React.Component {
    constructor(props){
        super(props);
        this.state = {
            "name": "Boris"
        }
    }

    render(){
        return(
            <div>
                <h1>{this.state.name}</h1>
            </div>
        );
    }
};
```

### Modifying state

React provides a method for updating component state called `setState`

You call the `setState` method within your component passing in an object with key-value pairs.  The keys are your state properties and the values are the updated state data.

**React expects you to *NEVER* modify state directly** instead you always use `this.setState()`. React may batch multiple state updates to improve performance thus the setState calls may be asynchronous.

```JSX
this.setState({
    username: 'Lewis'
});
```

### Accessing the state of a stateful component

You can access the state of a stateful component using `this.state`.

Before the return call you can access it using straight Javascript without using the curly braces.

Once you define a component's initial state, you can display any port of it in the UI that is rendered.  If a component is stateful, it will always have access tot he data in the state in its `render()` method.

```JSX
class MyComponent extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            name: 'Boris'
        }
    }

    render(){
        const name = this.state.name;
        return (
            <div>
                <h1>{name}<h1>
            </div>
        );
    }
}
```

sometimes you may need to know the previous state before you update state (toggles, and counter are an example of this scenario).  State updates may be asynchronous.  This means that you can't rely on the previous value of `this.state` or `this.props`.  To solve this issue you need to pass setState a function that will allow you to access state and props.

note that you need to wrap the object literal in parenthesis or Javascript thinks its a block of code.

```JSX
// with props passed
this.setState((state, props) => {
    counter: state.counter + props.increment
})

// without props passed
this.setState(state => ({
    counter: state.counter + 1
}));

// example used in a component
class MyComponent extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            visibility: false
        };
        this.toggleVisibility = this.toggleVisibility.bind(this);
    }

    toggleVisibility(){
        this.setState(state => ({
            visibility: !state.visibility
        }));
    }

    render(){
        if (this.state.visibility){
            return(
                <div>
                    <button onClick={this.toggleVisibility}>Click Me</button>
                    <h1>Now you see me!</h1>
                </div>
            );
        }
        else {
            return (
                <div>
                    <button onClick={this.toggleVisibility}>Click Me</button>
                </div>
            )
        }
    }
}
```

### Binding this to a class method

A class method typically needs to use the `this` keyword so it can access properties on the class such as state and props inside the scope of the method.  There are a few ways to allow your class method to access `this`.  One common way is to explicitly bind `this` in the constructor so that `this` becomes bound to the class methods when the component is initialized.

```JSX
class MyComponent extends React.Component {
    constructor(props){
        super(props){
            this.state = {
                text: "Hello"
            };
        }
        // doing the binding to this in the constructor
        this.handleClick = this.handleClick.bind(this)
    }

    handleClick(){
        // because we did the bind we are able to use this.handleClick.
        return(
            <div>
                <button onClick={this.handleClick}>Click Me</button>
                <h1>{this.state.text}</h1>
            </div>
        );
    }
};
```

### Rendering

The `render()` method of a component is what you use to return the JSX that the component will display to the user in the UI.

#### Use && for a More Concise Conditional Rendering

If based on state/logic of your component some portions should or should not be rendered, this is a clean way to accomplish this.

```JSX
{ this.state.display && <h1>Displayed!</h1> }
```

#### Ternary Expressions for Conditional Rendering

You can use the ternary operator as a shortcut for if/else statements in JavaScript.

The below example shows a chain of ternary operations and the logic is we load one button upon fresh load where the value is not set, and one of two buttons based on the input once user click the first button.

```JSX
{
    this.state.userAge === ''
    ? buttonOne 
    : this.state.userAge >= 18
        ? buttonTwo
        : buttonThree
}
```

### Composition

Composition is just the ability to have a component be used in another component.  In other words you can compose a component form other components.  This allows for re use of components as well as makes it easier to break up your application for maintenance.

## Lifecycle Methods

### What is it?

React components have special methods that provide opportunities to perfrom actions at specific points in the lifecycle of a component.  These are called **lifecycle methods** or **lifecycle hooks**.

Some of the main lifecycle methods are

* `componentDidMount()`
  * This method is called after a component is mounted to the DOM. It is best practice to place API calls into this lifecycle method. Any calls to `setState()` in this method will trigger a re-rendering of your component.  Thus if you call an API in this method, and set the state of your component based on the API call results it will trigger an update once you receive the data.  
  * The `componentDidMount()` method is also the best place to attach any event listeners you need to add for specific functionality.
    * `onClick` is an example of one of these synthetic event handlers.
    * React provides a synthetic event system which wraps the native event system present in browsers.  This means that the synthetic event system behaves the same regardless of the users's browser.
* `shouldComponentUpdate()`
  * takes `nextProps` and `nextState` as parameters.  You can compare the next props and next state to current using `this.props`.  This method must return a boolean that tells React if the component should update or not.
  * This method is useful for optimizing performance.
* `componentDidUpdate()`
* `componentWillUnmount()`
  * It is a good practice to use this method to do any cleanup such as removing event listeners.

## Controlled Inputs and Controlled Form

A controlled input is a React component that takes input and uses the component state to track the value of the input.

A controlled form is the same idea, but you have a form instead of a single field and have the state managed in the form.

Note that you must call event.preventDefault() on a form submit so that the default HTML behavior is bypassed.

## Inline Styles

To style a JSX element you can either assign it a class if you are importing styles from a stylesheet or you can use Inline Styles.

JSX elements use the style attribute, but because of the way JSX is transpiled, you can't set the value to a string.  Instead you set it equal to a JavaScript object.

```JSX
<div style={{color: "yellow", fontSize: 16}}>Mellow Yellow</div> 
```

Notice that we used fontSize instead of the css font-size.  In React you go camel case as the hyphenated version is invalid syntax for JavaScript.

All property value length units (like height, width, and fontSize) are assumed to be in px

If you want to use `em` use `""` for example `{fontSize: "4em"}`

## Render React on the Server with renderToString

Since React is a JavaScrip view library you can run JavaScript on a server with Node.  

You can use the  ReactDOMServer.renerToString() method to render on the server.

There are two key reasons why rendering on the server may be used in a real world app.

Without doing this your React app would be a small HTML file and a large bundle of JavaScript when it's initially loaded to the browser.  This may not be ideal for search engines that are trying to index the content on your page.  If you render the initial HTML markup on the server and send it to the client, the initial page load contains all of the page's markup which can be crawled.

This also make the initial page load faster.  Rendered HTML is smaller than the JavaScript code fo the entire app.  React will be able to recognize your app and mange it after the initial load.