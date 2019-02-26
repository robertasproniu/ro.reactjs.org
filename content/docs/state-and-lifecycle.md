---
id: state-and-lifecycle
title: Starea și Ciclul de viață
permalink: docs/state-and-lifecycle.html
redirect_from:
  - "docs/interactivity-and-dynamic-uis.html"
prev: components-and-props.html
next: handling-events.html
---

Această pagină introduce conceptul de stare și ciclu de viață al unei componente React. Puteți găsi aici o [referință detaliată a API-ului componentei](/docs/react-component.html).

Considerați exemplul cronometrului dat în [una din secțiunile anterioare](/docs/rendering-elements.html#updating-the-rendered-element). În [Randarea Elementelor](/docs/rendering-elements.html#rendering-an-element-into-the-dom), am învățat doar o modalitate de a actualiza interfața utilizator. Apelam `ReactDOM.render()` pentru a schimba conținutul elementului afisat:

```js{8-11}
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[**Încercați-l pe CodePen**](https://codepen.io/gaearon/pen/gwoJZk?editors=0010)

În această secțiune vom învăța cum să transformăm componenta `Clock` într-adevăr reutilizabilă și încapsulată. Aceasta va avea propriul cronometru și se va actualiza la fiecare secundă.

Putem începe prin încapsularea la cum va arata ceasul:

```js{3-6,12}
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[**Încercați-l pe CodePen**](https://codepen.io/gaearon/pen/dpdoYR?editors=0010)

Cu toate acestea, este omisă o cerință crucială: faptul că, `Clock` stabilește un cronometru și actualizează interfața de utilizator la fiecare secundă, ar trebui să fie un detaliu de implementare al componentei `Clock`.

În mod ideal, vrem să apelam o singura dată componenta `Clock`, iar aceasta să se actualizeze singură:

```js{2}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

Pentru a implementa acest lucru, trebuie să adăugăm "stare" la componenta `Clock`.

Starea este similară cu proprietățile, numai că este privată și este controlată în totalitate de componentă.

Am [menționat anterior](/docs/components-and-props.html#functional-and-class-components) că acele componente definite ca și clase au câteva caracteristici suplimentare. Starea locală este exact: acea caracteristică disponibilă numai pentru clase.

## Conversia unei funcții într-o clasă {#converting-a-function-to-a-class}

Puteți converti o componentă de tip funcție ca și `Clock` într-o clasă în cinci pași:

1. Creați o [clasă ES6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes), cu același nume, care extinde `React.Component`.

2. Adăugați o singură metodă goală numită `render()`.

3. Mutați corpul funcției în metoda `render()`.

4. Înlocuiți `props` cu `this.props` în corpul metodei `render()`.

5. Ștergeți declarația funcției goale rămase.

```js
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

[**Încercați-l pe CodePen**](https://codepen.io/gaearon/pen/zKRGpo?editors=0010)

`Clock` este acum definită mai degrabă ca o clasă decât ca o funcție.

Metoda `render` va fi apelată de fiecare dată când are loc o actualizare, dar atâta timp cât componenta `<Clock />` este afisată în același nod DOM, va fi utilizată doar o singură instanță a clasei `Clock`. Acest lucru permite folosirea de funcții suplimentare, cum ar fi metodele locale de stare și ciclul de viață.

## Adăugarea de stare locală la o clasă {#adding-local-state-to-a-class}

Vom muta `date` din proprietăți către stare în trei pași:

1) Înlocuiți `this.props.date` cu `this.state.date` în metoda `render()`:

```js{6}
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

2) Adăugați un [constructor de clasă](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes#Constructor) care asignează `this.state` inițial:

```js{4}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Rețineți modul de trimitere `props` la constructorul de bază:

```js{2}
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
```

Componentele clasă ar trebui sa apeleze constructorul de bază cu `props`.

3) Stergeți proprietatea `date` din elementul `<Clock />`:

```js{2}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

Mai târziu vom adăuga codul pentru ceas înapoi în componentă.

Rezultatul arată astfel:

```js{2-5,11,18}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[**Încercați-l pe CodePen**](https://codepen.io/gaearon/pen/KgQpJd?editors=0010)

Apoi, vom face componenta `Clock` să-și creeze propriul cronometru și să se actualizeze la fiecare secundă.

## Adăugarea metodelor ciclului de viață la o clasă {#adding-lifecycle-methods-to-a-class}

In applications with many components, it's very important to free up resources taken by the components when they are destroyed.

We want to [set up a timer](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval) whenever the `Clock` is rendered to the DOM for the first time. This is called "mounting" in React.

We also want to [clear that timer](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/clearInterval) whenever the DOM produced by the `Clock` is removed. This is called "unmounting" in React.

We can declare special methods on the component class to run some code when a component mounts and unmounts:

```js{7-9,11-13}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {

  }

  componentWillUnmount() {

  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

These methods are called "lifecycle methods".

The `componentDidMount()` method runs after the component output has been rendered to the DOM. This is a good place to set up a timer:

```js{2-5}
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
```

Note how we save the timer ID right on `this`.

While `this.props` is set up by React itself and `this.state` has a special meaning, you are free to add additional fields to the class manually if you need to store something that doesn’t participate in the data flow (like a timer ID).

We will tear down the timer in the `componentWillUnmount()` lifecycle method:

```js{2}
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```

Finally, we will implement a method called `tick()` that the `Clock` component will run every second.

It will use `this.setState()` to schedule updates to the component local state:

```js{18-22}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/amqdNA?editors=0010)

Now the clock ticks every second.

Let's quickly recap what's going on and the order in which the methods are called:

1) When `<Clock />` is passed to `ReactDOM.render()`, React calls the constructor of the `Clock` component. Since `Clock` needs to display the current time, it initializes `this.state` with an object including the current time. We will later update this state.

2) React then calls the `Clock` component's `render()` method. This is how React learns what should be displayed on the screen. React then updates the DOM to match the `Clock`'s render output.

3) When the `Clock` output is inserted in the DOM, React calls the `componentDidMount()` lifecycle method. Inside it, the `Clock` component asks the browser to set up a timer to call the component's `tick()` method once a second.

4) Every second the browser calls the `tick()` method. Inside it, the `Clock` component schedules a UI update by calling `setState()` with an object containing the current time. Thanks to the `setState()` call, React knows the state has changed, and calls the `render()` method again to learn what should be on the screen. This time, `this.state.date` in the `render()` method will be different, and so the render output will include the updated time. React updates the DOM accordingly.

5) If the `Clock` component is ever removed from the DOM, React calls the `componentWillUnmount()` lifecycle method so the timer is stopped.

## Using State Correctly {#using-state-correctly}

There are three things you should know about `setState()`.

### Do Not Modify State Directly {#do-not-modify-state-directly}

For example, this will not re-render a component:

```js
// Wrong
this.state.comment = 'Hello';
```

Instead, use `setState()`:

```js
// Correct
this.setState({comment: 'Hello'});
```

The only place where you can assign `this.state` is the constructor.

### State Updates May Be Asynchronous {#state-updates-may-be-asynchronous}

React may batch multiple `setState()` calls into a single update for performance.

Because `this.props` and `this.state` may be updated asynchronously, you should not rely on their values for calculating the next state.

For example, this code may fail to update the counter:

```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

To fix it, use a second form of `setState()` that accepts a function rather than an object. That function will receive the previous state as the first argument, and the props at the time the update is applied as the second argument:

```js
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

We used an [arrow function](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions) above, but it also works with regular functions:

```js
// Correct
this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});
```

### State Updates are Merged {#state-updates-are-merged}

When you call `setState()`, React merges the object you provide into the current state.

For example, your state may contain several independent variables:

```js{4,5}
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```

Then you can update them independently with separate `setState()` calls:

```js{4,10}
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```

The merging is shallow, so `this.setState({comments})` leaves `this.state.posts` intact, but completely replaces `this.state.comments`.

## The Data Flows Down {#the-data-flows-down}

Neither parent nor child components can know if a certain component is stateful or stateless, and they shouldn't care whether it is defined as a function or a class.

This is why state is often called local or encapsulated. It is not accessible to any component other than the one that owns and sets it.

A component may choose to pass its state down as props to its child components:

```js
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```

This also works for user-defined components:

```js
<FormattedDate date={this.state.date} />
```

The `FormattedDate` component would receive the `date` in its props and wouldn't know whether it came from the `Clock`'s state, from the `Clock`'s props, or was typed by hand:

```js
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/zKRqNB?editors=0010)

This is commonly called a "top-down" or "unidirectional" data flow. Any state is always owned by some specific component, and any data or UI derived from that state can only affect components "below" them in the tree.

If you imagine a component tree as a waterfall of props, each component's state is like an additional water source that joins it at an arbitrary point but also flows down.

To show that all components are truly isolated, we can create an `App` component that renders three `<Clock>`s:

```js{4-6}
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/vXdGmd?editors=0010)

Each `Clock` sets up its own timer and updates independently.

In React apps, whether a component is stateful or stateless is considered an implementation detail of the component that may change over time. You can use stateless components inside stateful components, and vice versa.
