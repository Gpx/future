# State

This is how we handle the state in `web` right now:

```jsx
class FooContainer extends React.Component {
  render() {
    return <Foo foo={this.state.foo} />
  }
}

function Foo(props) {
  return <div>{this.props.foo}</div>
}
```

There are a few issues with this approach. First, if we want to render `<Foo>` we actually have to render `<FooContainer>`. This might be counterintuitive. There's an easy workaround which is to have an `index.js` file that exports `<FooContainer>` as `<Foo>`. This approach works but adds some magic to how the whole application is structured.

The second problem with having containers is that they are not easily reusable. Since `<FooContainer>` will always render `<Foo>` I can't use its state and logic with a `<Bar>` component. The only way to do it is to pass an additional prop, something like this:

```jsx
class FooContainer extends React.Component {
  render() {
    return this.props.renderFoo
      ? <Foo foo={this.state.foo} />
      : <Bar foo={this.state.foo} />
  }
}
```

This can easily get out of hand since `<FooContainer>` is now dealing with two components instead of having one single responsbility. Imagine how your component will look in a few weeks:

```jsx
class FooContainer extends React.Component {
  handleThingThatHappensForFooOnly() {}
  
  handleThingThatHappensForBarOnly() {}

  render() {
    return this.props.renderFoo
      ? <Foo foo={this.state.foo} onFoo={this.handleThingThatHappensForFooOnly} />
      : (
        <Bar
          foo={this.state.foo}
          bar={this.state.bar}
          onFoo={this.handleThingThatHappensForFooOnly}
          onBar={this.handleThingThatHappensForBarOnly}
        )
      />
  }
}
```

To fix this situation I'm proposing a new patter based on [render props](https://reactjs.org/docs/render-props.html).

## Render Props

First, a brief recap of what render props are.

```jsx
function DoubleComponent(props) {
  const double = this.props.value * 2
  return props.render({ double })
}

// The following will render <div>10</div>
<DoubleComponent
  value={5}
  render={({ double }) => <div>{double}</div>}
/>
```

So, a render property is just a prop that happens to be a function that returns some node. Of course, the prop doesn't have to be called `render`. As a matter of fact, if we name the prop `children` we can do some neat things:

```jsx
function DoubleComponent(props) {
  const double = this.props.value * 2
  return props.children({ double })
}

// The following will render <div>10</div>
<DoubleComponent value={5}>
  {({ double }) => <div>{double}</div>}>}
</DoubleComponent>
```

## Render props for state management

The render props patter can be extrimely useful for state management. Imagine we rewrite our `<FooContainer>` to use render props:

```jsx
class FooContainer extends React.Component {
  render() {
    return this.props.children({ foo: this.state.foo })
  }
}
```

What we have now is a component that handles a state and knows how to modify it but can be used everywhere. We can modify `<Foo>` to use our new state:

```jsx
function Foo(props) {
  return (
    <FooContainer>
      {({ foo }) => <div>{foo}</div>}
    </FooContainer>
  )
}
```

What if `<Bar>` needs `foo` too? As easy as this:

```jsx
function Bar(props) {
  return (
    <BarContainer>
      {({ bar }) => (
        <FooContainer>
          {({ foo }) => <div>{bar} and {foo}</div>}
        </FooContainer>
      )}
    </BarContainer>
  )
}
```

Now components are choosing which state they need and they can easily access it rather than the other way around.

## Naming

