# Context

In React the state lives inside a component but in some cases you have some values that should be shared across the whole application.
The logged user is one example. Others could be feature flags `history`, `location` and locale information. Everywhere in your app you know that those things exist and you need an easy way to access them.

Until now we were putting this values in our root component `<App>` and passing them down with props. This often meant drilling a prop several layers. What's worse, often times the components in between didn't care about that particular value, they were only "carrying it over".

React 16.3 introduced a new context API that we can leverage exactly for this.

That's how you would handle the currently logged in user:

```jsx
export const CurrentUserContext = React.createContext()

class App extends React.Component {
  state = { currentUser: undefined }

  componentDidMount() {
    const currentUser = await fetchCurrentUser()
    this.setState({ currentUser })
  }
  
  render () {
    <CurrentUserContext.Provider value={this.state.currentUser}>
      {/* rest of App goes here */}
    </CurrentUserContext.Provider>
  }
}
```

We simply crate a new context and we export it. `App` is in charge of changing its value. Now everywhere we need the current user we can leverage render props:

```jsx
import { CurrentUserContext } from '../App'

function AppHeader(props) {
  return (
    <CurrentUserContext.Consumer>
      {currentUser => <div>Welcome back {currentUser.firstName}</div>}
    </CurrentUserContext.Consumer>
  )
}
```

Of course, the value we set for the context can come from a state:

```jsx
export const CurrentUserContext = React.createContext()

class App extends React.Component {  
  render () {
    <CurrentUserState>
      {({ currentUser }) => (
        <CurrentUserContext.Provider value={currentUser}>
          {/* rest of App goes here */}
        </CurrentUserContext.Provider>
      )}
    </CurrentUserState>
  }
}
```

In this way, `App` just sets up the global contexts—plus sets the routes and few other initialisations—but does not modify the state.

---

If you think that context and the new state are similar you are absolutely right. They both hold data and they make it available via render props. There is althought a big difference. Context is global, meaning that the same value will be available to your component and its descendents. State, on the other hand. gets initialized for every component you use.

This is quite important. Importing the same state container in two different components won't allow them to share the state, it will only create two copies of the state that will go out of sync as soon as one gets modified.

If you need two or more components to share the state you can—as usual in React—move it to a parent component.

