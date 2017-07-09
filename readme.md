<h1 align="center">
  <br>
  <img width="192" src="media/logo.png">
  <br>
  <br>
  <br>
</h1>

> React for CLIs. Build and test your CLI output using components.

[![Build Status](https://travis-ci.org/vadimdemedes/ink.svg?branch=master)](https://travis-ci.org/vadimdemedes/ink)


## Install

```
$ npm install --save ink
```


## Usage

```jsx
const {h, mount, Component, Text} = require('ink');

class Counter extends Component {
	constructor() {
		super();

		this.state = {
			i: 0
		};
	}

	render() {
		return (
			<Text green>
				{this.state.i} tests passed
			</Text>
		);
	}

	componentDidMount() {
		this.timer = setInterval(() => {
			this.setState({
				i: this.state.i + 1
			});
		}, 100);
	}

	componentWillUnmount() {
		clearInterval(this.timer);
	}
}

mount(<Counter/>, process.stdout);
```

<img src="media/demo.gif" width="428" height="232">


## Useful Components

- [ink-redux](https://github.com/vadimdemedes/ink-redux) - Redux bindings.
- [ink-text-input](https://github.com/vadimdemedes/ink-text-input) - Text input component.
- [ink-progress-bar](https://github.com/brigand/ink-progress-bar) - Configurable component for rendering progress bars.


## Guide

- [Getting Started](#getting-started)
- [Core API](#core-api)
- [Components](#components)
- [Props](#props)
- [Lifecycle Methods](#lifecycle-methods)
- [State](#state)
- [Refs](#refs)
- [Context](#context)
- [Stateless Function Components](#stateless-function-components)
- [Built-in Components](#built-in-components)

Ink's goal is to provide the same component-based UI building experience that React provides, but for CLI. That's why it tries to implement the minimum required functionality of React. If you are already familiar with React (or Preact, since Ink borrows a few ideas from it), you already know Ink.

They key difference you have to remember is that the rendering result isn't a DOM, but a string, which Ink writes to output.

### Getting Started

To ensure all examples work and you can begin your adventure with Ink, make sure to set up a JSX transpiler and set JSX pragma to `h`.
Don' forget to import `h` into every file that contains JSX.

```jsx
const {h} = require('ink');

const Demo = () => <div/>;
```

### Core API

#### mount(tree, stream)

Mount a component, listen for updates and update the output.
This method is used for interactive UIs, where you need state, user input or lifecycle methods.

##### tree

Type: `VNode`

##### stream

Type: `Stream`<br>
Default: `process.stdout`

```jsx
const {h, mount, Component} = require('ink');

class Counter extends Component {
	constructor() {
		super();

		this.state = {
			i: 0
		};
	}

	render(props, state) {
		return `Iteration #${state.i}`;
	}

	componentDidMount() {
		this.timer = setInterval(() => {
			this.setState({
				i: this.state.i + 1
			});
		}, 100);
	}

	componentWillUnmount() {
		clearInterval(this.timer);
	}
}

const unmount = mount(<Counter/>);

setTimeout(() => {
	// enough counting
	unmount();
}, 1000);
```

#### renderToString(tree)

Render a previously generated VDOM to a string, which you can flush to output.

#### render(tree, [prevTree])

Build a VDOM representation of components.
Useful if you don't intend to use state or lifecycle methods and just want to render the UI once and exit.

```jsx
const {h, render, renderToString} = require('ink');

const Hello = () => 'Hello World';

const tree = render(<Hello/>);
process.stdout.write(renderToString(tree));
```

### Components

Similarly to React, there are 2 kinds of components: Stateful components (next, "component") and stateless function components.
You can create a component by extending `Component` class.
Unlike stateless function components, they have access to state, context, lifecycle methods and they can be accessed via refs.

```js
class Demo extends Component {
	render(props, state, context) {
		// props === this.props
		// state === this.state
		// context === this.context

		return 'Hello World';
	}
}
```

If you need to extend the constructor to set the initial state or for other purposes, make sure to call `super()` with `props` and `context`:

```js
constructor(props, context) {
	super(props, context);

	this.state = {
		i: 0
	};

	// other initialization
}
```

#### Props

Props are basically arguments for components.
Every parent component can pass props to their children.

```jsx
class Child extends Component {
	render(props) {
		// props === this.props

		return `Hello, ${props.name}`;
	}
}

class Parent extends Component {
	render() {
		return <Child name="Joe"/>;
	}
}
```

#### Lifecycle Methods

Lifecycle methods are component methods that are called whenever a certain event happens related to that specific component. All lifecycle methods are called from top to down, meaning that components on top receive those events earlier than their children.

##### componentWillMount()

Component is initialized and is about to be rendered and written to the output.

##### componentDidMount()

Component is rendered and written to the output.

##### componentWillUnmount()

Component is about to be unmounted and component instance is going to be destroyed.
This is the place to clean up timers, cancel HTTP requests, etc.

##### componentWillReceiveProps(nextProps, nextState)

Component is going to receive new props or state.
At this point `this.props` and `this.state` contain previous props and state.

##### shouldComponentUpdate(nextProps, nextState)

Determines whether to rerender component for the next props and state.
Return `false` to skip rerendering of component's children.
By default, returns `true`, so component is always rerendered on update.

##### componentWillUpdate(nextProps, nextState)

Component is about to rerender.

##### componentDidUpdate()

Component was rerendered and was written to output.

#### State

Each component can have its local state accessible via `this.state`.
Whenever a state updates, component is rerendered.

To set the initial state, extend the constructor and assign an object to `this.state`.
Anywhere in the component state is accessible via `this.state`, and it's also passed to `render()` as a second argument.

```js
class Demo extends Component {
	constructor(props, context) {
		super(props, context);

		this.state = {
			i: 0
		}
	}

	render(props, state) {
		return `Iteration ${state.i}`;
	}
}
```

##### setState(nextState)

###### nextState

Type: `Object`, `Function`
Default: `{}`

Set a new state and update the output.

**Note**: `setState()` works by **extending** the state via `Object.assign()`, not replacing it with a new object. Therefore you can pass only changed values.

```js
class Demo extends Component {
	constructor(props, context) {
		super(props, context);

		this.state = {
			i: 0
		}
	}

	render(props, state) {
		return `Iteration ${state.i}`;
	}

	componentDidMount() {
		this.setState({
			i: this.state.i + 1
		});
	}
}
```

The above example will increment the `i` state property and render `Iteration 1` as a result.

`setState()` also accepts a function, which receives the current state as an argument.
The same effect of incrementing `i` could be achieved in a following way:

```js
this.setState(state => {
	return {
		i: state.i + 1
	}
});
```

This is useful when `setState()` calls are batched to ensure that you update the state in a stable way.

#### Refs

Refs can be used to get a direct reference to a component instance.
This is useful, if you want to access its methods, for example.

Refs work by setting a special `ref` prop on a component.
Prop's value must be a function, which receives a reference to a component as an argument or `null` when the wanted component is unmounted.

**Note**: You can't get refs to stateless function components.

```jsx
class Child extends Component {
	render() {
		return null;
	}

	hello() {
		return 'Hello World';
	}
}

class Parent extends Component {
	constructor(props, context) {
		super(props, context);

		this.state = {
			message: 'Ink is awesome'
		};
	}

	render(props, state) {
		const setChildRef = ref => {
			this.childRef = ref;
		};

		return (
			<div>
				{message}

				<Child ref={setChildRef}/>
			</div>
		)
	}

	componentDidMount() {
		this.setState({
			message: this.childRef.hello()
		});
	}
}
```

#### Context

Context is like a global state for all components.
Every component can access context either via `this.context` or inside `render()`:

```js
render(props, state, context) {
	// context === this.context
}
```

To add new entries to context, add `getChildContext()` method to your component:

```js
class Child extends Component {
	render() {
		return this.context.message;
	}
}

class Parent extends Component {
	getChildContext() {
		return {
			message: 'Hello World'
		};
	}

	render() {
		return <Child/>;
	}
}
```

#### Stateless Function Components

If you don't need state, lifecycle methods, context and refs, it's best to use stateless function components for their small amount of code and readability.

Using stateful components:

```js
class Demo extends Component {
	render(props, state, context) {
		return 'Hello World';
	}
}
```

Using stateless function components:

```js
const Demo = (props, context) => 'Hello World';
```

As you may have noticed, stateless function component still get access to props and context.

#### Built-in Components

Surprise, surprise, our favorite `<div>` and `<span>` can be used in Ink components!
They contain zero functionality, but are useful for grouping elements, since JSX doesn't allow multiple elements without a parent.

This won't work:

```jsx
const Demo = (
	<A/>
	<B/>
	<C/>
);
```

This will:

```jsx
const Demo = (
	<div>
		<A/>
		<B/>
		<C/>
	</div>
);
```

There's also `<br/>`, which serves the same purpose as on the web - a newline.

```jsx
const Demo = (
	<div>
		Line 1
		<br/>
		Line 2
	</div>
);
```

## License

MIT © [Vadim Demedes](https://github.com/vadimdemedes)
