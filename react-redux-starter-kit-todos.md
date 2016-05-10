# React Redux Starter Kit Todos Tutorial
This is my example of using [react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit) and [redux-cli](https://github.com/SpencerCDixon/redux-cli) to replicate [the Todos app from the Redux manual](http://redux.js.org/docs/basics/ExampleTodoList.html). I'll go through how I wish the CLI worked because in many places the CLI doesn't doesn't yet support the [fractal design pattern](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure) of routes. This is partially because the [default blueprints included with react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit/tree/master/blueprints) don't include routes. Partially because [you can't use a `--path` option](https://github.com/SpencerCDixon/redux-cli/issues/72) (like `redux g module myName --path routes/my-route`) with redux-cli yet. It's probably worse that *components*, *containers* and *modules* are called *dumb*, *smart* and *duck* in the default blueprints. But we can fix that ourselves for our own projects.

## This is a hands-on tutorial
I'm going to write this as if you have your command line open (you should [probably](http://apple.stackexchange.com/questions/25143/what-is-the-difference-between-iterm2-and-terminal) be using [iTerm2](https://www.iterm2.com/) and you're following along. This document is designed for getting you up to speed on the react-redux-starter-kit ecosystem. There *will be* other articles if you're interested in [writing an app with sagas](./redux-sagas-todos.md) or [connecting sagas to an api](./redux-sagas-data.md).

#### Install Node
It's a no-brainer but you should have the [current version of Node](https://nodejs.org/en/) installed. If you're using an other version of Node, there is no reason not to upgrade your local to version 6 (or whatever is currently listed), take a moment and upgrade. It's as easy as downloading the latest installer and double-clicking it. Some people [recommend homebrew to install Node](http://blog.teamtreehouse.com/install-node-js-npm-mac) (from 2014) but I've never run into any issues with the official installer and I try to avoid homebrew (no good reason). These days I use docker anytime I'm working with something that's usually designed for Linux. Working with "linuxy" things is [the main reason you would use homebrew](http://computers.tutsplus.com/tutorials/homebrew-demystified-os-xs-ultimate-package-manager--mac-44884). Node isn't [a foreign thing to OSX](https://gist.github.com/DanHerbert/9520689). I prefer the installer.

*Note:* If you are getting permissions issues with NPM after using the installer you may need to [fix your permissions](https://docs.npmjs.com/getting-started/fixing-npm-permissions).

```bash
# official way to fix your permissions
# @see https://docs.npmjs.com/getting-started/fixing-npm-permissions
sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}
```

#### Install redux-cli
You need to install [redux-cli](https://github.com/SpencerCDixon/redux-cli) as a global npm package. This is currently a barebones command-line tool.  Right now it's mostly useful for initializing a new project but it's clear that this tool will be expanding over time. The major innovation of redux-cli tool over something like [Yeoman](http://yeoman.io/generators/) is that there is a strong encouragement for you to [manage the blueprints from within your project](https://github.com/SpencerCDixon/redux-cli#creating-blueprints). This is actually a key advancement that is possibly easy to overlook. Being able to easily customize the default generators and create new blueprints is a huge advancement.

The [react-reduct-starter-kit includes default blueprints](https://github.com/davezuko/react-redux-starter-kit/tree/master/blueprints).

Read this history of [how the redux-cli tool got started](https://github.com/davezuko/react-redux-starter-kit/issues/595).

```bash
npm install redux-cli -g
```


## Start a new project
Navigate to your projects folder. If you don't have one, consider creating one `mkdir -p ~/Projects/tests`. I'm going to be pasting commands below with the assumption that you're working in a folder as valuable to you as `~/Projects/tests`.

```bash
redux new todos-app

# creates a todos-app directory
# puts the latest react-redux-starter-kit in there
# @see https://github.com/SpencerCDixon/redux-cli#getting-started

# navigate into your app
cd todos-app

# install npm modules
npm install
```

You can poke around if you'd like. You should read about [what's in the box](https://github.com/davezuko/react-redux-starter-kit) and [how to use it](https://suspicious.website/2016/04/29/starting-out-with-react-redux-starter-kit/) if you haven't already. For our purposes we'll be doing everyhting in the `src/` folder.

#### Rename `dumb`, `smart` and `duck` blueprints
For fun, and to give a small taste of what blueprints are capable of, let's rename some of the default blueprints. Renaming the folders under `blueprints/` changes how you call the generators. Now, instead of calling something like `redux g dumb MyName` you would use `redux g component MyName`. It makes sense to rename these blueprints because if you look you'll see that the react-redux-starter-kit names those folders `components/`,  `containers/` and  `redux/modules/` respectively.

```bash
mv blueprints/dumb blueprints/component
mv blueprints/smart blueprints/container
mv blueprints/duck blueprints/module

# see if it works
redux g component fun
```

## Create a new route
It's currently [not possible to add a route using redux-cli](https://github.com/SpencerCDixon/redux-cli/issues/6) but that should be getting fixed eventually. For now we'll have to make it by hand. We're creating a Todo app so we'll create a "Todo" route. We're presuming that you're fairly new to React-Redux so we'll be leaving the boilerplate routes in place for now because they're a good reference. There's a few steps involved in creating a new route and it's ok if you don't fully understand routes right away. It's important to know that we're using [react-router](https://github.com/reactjs/react-router) with the [react-router-redux](https://github.com/reactjs/react-router-redux) bindings.

If you're new to react-router they recommend that you:
- [Read the react-router tutorial *first*](https://github.com/reactjs/react-router-tutorial)
- [Read the react-router docs](https://github.com/reactjs/react-router/tree/master/docs)

We're actually just going to be copying the [`Counter` route](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/Counter/index.js) from the boilerplate starter-kit app.

### Create the files for your route
We're going to eventually be storing our components, containers and modules using the [fractal project structure](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure) so we'll be creating `components/`, `containers/` and `modules/` folders inside our `routes/Todos/` folder. *Hint:* [`mkdir -p <path>`](http://unix.stackexchange.com/questions/49263/recursive-mkdir).

```bash
mkdir -p src/routes/Todos/components
touch src/routes/Todos/components/TodosView.js
touch src/routes/Todos/index.js

# don't forget to create tests
mkdir -p tests/routes/Todos/components
touch tests/routes/Todos/components/TodosView.spec.js
touch tests/routes/Todos/index.spec.js

# this doesn't work yet 
redux g route Todos
```

### `src/routes/Todos/index.js`
We need to add some boilerplate code to our route file. You can see the way the router itself is configured by checking in [`src/routes/index.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/index.js). By default your app is configured to allow for lazy-loading of routes. This is an important consideration for large apps and it's easy enough to support it from the very beginning. You don't have to know exactly how all of this magic works but you need understand that it's important. Under the hood we're [using Webpack's code-splitting feature](https://webpack.github.io/docs/code-splitting.html) for performance.

```js
import { injectReducer } from '../../store/reducers'

export default (store) => ({
  path: 'todos',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const TodosView = require('./components/TodosView').default
      const reducer = require('./modules/todos').default

      injectReducer(store, { key: 'todos', reducer })

      cb(null, TodosView)
    }, 'todos')
  }
})
```

It's important to pay attention to what's going on here:

1. We need to specify a [`path`](https://github.com/reactjs/react-router/blob/master/docs/API.md#path) property [using the config notation](https://github.com/reactjs/react-router/blob/master/docs/guides/RouteConfiguration.md#configuration-with-plain-routes). Most examples demonstrate building the router as a series of React components like `<Route path="about" component={About} />` but we're [using the config notation instead](https://github.com/reactjs/react-router/issues/865).

  ```js
  //..

  export default (store) => ({
    path: 'todos', // <-- this shows up in the URL
    //...
  })
  ```

2. We need to specify a [`getComponent(nextState, cb)`](https://github.com/reactjs/react-router/blob/master/docs/API.md#getcomponentnextstate-callback) function in order to lazy-load our route. This is designed for code-splitting as specified in the react-router manual. We're going to be using [Webpack for code-splitting](https://webpack.github.io/docs/code-splitting.html). If you want to specify a route without using aync loading you can use [`component`](https://github.com/reactjs/react-router/blob/master/docs/API.md#component) instead (but you should use async unless you have a good reason not to).

  ```js
  // ...

  export default (store) => ({
    path: 'todos',

    // this is for async loading a route
    // @see https://github.com/reactjs/react-router/blob/master/docs/API.md#getcomponentnextstate-callback
    getComponent (nextState, cb) {

      // ... TodosView

      // be sure to pass your view component to the callback
      // null means there weren't any errors
      cb(null, TodosView)

      // ...
    }
  })
  ```

  ```js
  // ...

  // you could import your view normally
  // if you don't need async for some reason
  import TodosView from './components/TodosView'

  export default (store) => ({
    path: 'todos',

    // it's pretty straightforward
    // if you don't need code-splitting
    component: TodosView
  })
  ```

3. We need to [name our Webpack chunk](https://github.com/webpack/webpack/tree/master/examples/named-chunks). Webpack supports [code-splitting](https://webpack.github.io/docs/code-splitting.html) through a specialty `require.ensure()` function. If you've been reading about Node then you've probably heard about using the `require()` function for [loading modules](https://nodejs.org/api/modules.html). If you've been reading about ES6 then you probably know you should *usually* [use `import` instead of `require`](http://stackoverflow.com/questions/31354559/using-node-js-require-vs-es6-import-export). The `require.ensure()` function is particular to Webpack, not Node, and solves a specific use-case for lazy-loading parts of your app for performance reasons. If it bothers you that we're [using `require` instead of `import`](https://webpack.github.io/docs/code-splitting.html#es6-modules) that's a good instinct but in this case `require.ensure()` is the only way to do what we need.

  ```js
  // ...

  export default (store) => ({
    path: 'todos',
    getComponent (nextState, cb) {

      // require.ensure is a special Webpack thing
      // the first argument should be an empty array
      // @see https://webpack.github.io/docs/code-splitting.html
      require.ensure([], (require) => {

        // require.ensure passes the require function for us to use.
        // ...

      }, 'todos') // <-- this names the webpack chunk for this route
    }
  })
  ```

4. We need to import our view and our module using the `require` function passed to us by `require.ensure`. You'll notice that we're [tacking a `.default` on the end of our `require()` function](https://github.com/esnext/es6-module-transpiler/issues/85). That's required to cover a gap in how `import` works vs `require`. You can read in depth about [ES6 modules](http://www.2ality.com/2014/09/es6-modules-final.html) if you'd like. The key take away is that **if you want to import an ES6 module programatically you have to use `require()`**. And if you use `require()` to import an ES6 module, you will receive an object where each key matches one of your named exports (ES6 modules support [named exports](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) and `require()` doesn't). In this case we need to use the `default` export, so we need to use the `.default` property. *Note:* If you've ever wondered [how to import every file in a folder](http://stackoverflow.com/questions/5364928/node-js-require-all-files-in-a-folder), that's what `require()` is for.

  ```js
  // ...

  const TodosView = require('./components/TodosView').default // <-- you must specify .default
  const reducer = require('./modules/todos').default

  // ...
  ```

5. We need to inject our reducer into the store using the `injectReducer(store, map)` function. This function comes with the react-redux-starter-kit in [`src/store/reducers.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/reducers.js). This allows for lazy-loaded routes to add reducers to the Redux store after the initial page load. There's not much magic going on there but if you poke around and see `asyncReducers` mentioned in the code it's referring to list of reducers from routes that were loaded as asynchronous chunks by Webpack. **Important:** the `key` specifies a key on the store that the reducers in your route will inherit from. It's similar in concept to what [`combineReducers()`](http://redux.js.org/docs/api/combineReducers.html) does.

  ```js
  import { injectReducer } from '../../store/reducers'

  // ...

  // make sure to give your reducer a good name
  injectReducer(store, { key: 'todos', reducer })

  // ...
  ```

*Note:* If you notice, our "route" is a function that accepts `store` and returns a plain object. If you wanted to be pedantic you might call this a routeCreator instead of a route. It hardly matters what you call it but passing in the store this way is an important part of making Redux available to the containers in your route.

```js
export default (store) => ({ // <-- am I a route creator?
  // ... 
})
```

#### Add our route to the router
In a typical React-Redux app nothing is loaded by magic. Simply creating a route won't make it show up in the site. You can check out the parent router in [`src/routes/index.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/index.js). You should be able to get a handle on what's happening in this file. One important consideration is how the Redux `store` is passed into each route.

```js
// src/routes/index.js
// ...
import Home from './Home'
import CounterRoute from './Counter'
import TodosRoute from './Todos' // <-- import your new route

export const createRoutes = (store) => ({
  path: '/',
  component: CoreLayout,
  indexRoute: Home,
  childRoutes: [
    CounterRoute(store),
    TodosRoute(store) // <-- initialize your new route
  ]
})

// ...
```

1. You can see that `Home` is specified as the `indexRoute`. You can also see there is a `CoreLayout` included as the `component`. This is how your app can provide a "layout" for things like managing the global header and footer.
2. You can see that you have to call your `TodosRoute` creator function and pass in `store` manually. If you had a route that didn't need the store you could return a plain object instead of a function in your route.
3. If you notice, this file is also just a route. You can use layouts and `childRoutes` in any route that you create. The routes you define are imported by [`src/main.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/main.js). That's the file that does all the hard work of bootstrapping your app.

#### Add our route to the navigation
You also need to manually manage your site's navigation. You can see how the default app manages the nav in [`src/components/Header/Header.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/components/Header/Header.js). Because our app uses react-router you don't have to import your route to create a link to it. You use react-router's [`Link` helper component](https://github.com/reactjs/react-router/blob/master/docs/API.md#link) instead. You may remember that we're using [react-router-redux](https://github.com/reactjs/react-router-redux) but that's an unimportant detail. In most cases you'll be working directly with react-router and only using react-router-redux for fancy things like time travel.

Here we need to add a `Link` to our `/todos` route. What you put in `<Link to='/todos'` should match the `path: 'todos'` from your route. Here we're prefixing it with a `\` because it is "beneath" the home route. How you nest your routes determines how the URL path to the route is constructed.

```jsx
// src/components/Header/Header.js
// ...
export const Header = () => (
  <div>
    <h1>React Redux Starter Kit</h1>
    <IndexLink to='/' activeClassName={classes.activeRoute}>
      Home
    </IndexLink>
    {' · '}
    <Link to='/counter' activeClassName={classes.activeRoute}>
      Counter
    </Link>
    {' · '}
    <Link to='/todos' activeClassName={classes.activeRoute}>
      Todos
    </Link>
  </div>
)
// ...
```

*Note:* You might notice the `{' · '}` in the code above. That's the [standard React way to add whitespace](https://github.com/facebook/react/issues/1643#issuecomment-45325969). Normally React will strip whitespace between elements. You might like reading about [how whitespace works in React](http://andrewhfarmer.com/how-whitespace-works-in-jsx/).

### `src/routes/Todos/components/TodosView.js`
In the `src/routes/Todos/index.js` file we referenced the `TodosView`. A view is just a component. A view is specific to a route, meaning that a route's primary component is usually considered a view. The react-redux-starter-kit has a [blueprint for views](https://github.com/davezuko/react-redux-starter-kit/tree/master/blueprints/view) that you can use with `redux g view ViewName` but we're not going to use that because it can't add files inside of a route and we want to use the fractal project layout. By default the generator creates a `${ViewName}View.js` file in the `views/` folder. We've placed our view in the `components/` folder under our route.

Why? Because a view is just a component (unless it's a container). It's not necessary to break views out into another folder because it's customary to name a view component with like `${RouteName}View`. This makes it unmistakable that the component in question is the view for your route. You are likely to see different developers have different preferences for where the view is located. One reason to keep it in the components folder is that you could reasonably have a view that is a container. In that case the `views/` folder would conceal the fact that the view was a container. Meanwhile, if you keep the view in either the `components/` or `containers/` folder a developer would have a hint about what the file contains before they even open it.

You can easily create a view from the command line with a command like this:

```bash
mkdir -p src/routes/Todos/components
touch src/routes/Todos/components/TodosView.js

# don't forget to make tests
mkdir -p tests/routes/Todos/components
touch tests/routes/Todos/components/TodosView.spec.js

# this doesn't work yet
redux g component TodosView --path routes/Todos
```

This is more straighforward file than a route because it's just a simple presentational component. You can see that we import React, import the `Footer`, `AddTodo` and `VisibleTodoList` and then export a template that simply outputs them.

```jsx
import React from 'react'
import Footer from './Footer'
import AddTodo from '../containers/AddTodo'
import VisibleTodoList from '../containers/VisibleTodoList'

const TodosView = () => (
  <div>
    <AddTodo />
    <VisibleTodoList />
    <Footer />
  </div>
)

export default TodosView
```

## Testing it all out
At this point we've got a new route with a view. We're referencing a bunch of other components but we can comment that part out and see our app in action right now!

```jsx
// src/routes/Todos/components/TodosView.js

import React from 'react'

// comment out the view for now

// import Footer from './Footer'
// import AddTodo from '../containers/AddTodo'
// import VisibleTodoList from '../containers/VisibleTodoList'

// const TodosView = () => (
//   <div>
//     <AddTodo />
//     <VisibleTodoList />
//     <Footer />
//   </div>
// )

// return a temporary view
const TodosView = () => (
  <div>
    Todo: Make a todos app
  </div>
)
export default TodosView
```

```bash
npm run dev
```

# Create the Todo files
Now that we have our route set up we just need to add in all of the files needed to complete the classic [Todo example from the Redux manual](http://redux.js.org/docs/basics/index.html). We're going to assume that you've reviewed that application. Below we're changing the files to fit better with the react-redux-starter-kit.


## Filter Links
These are the files needed to complete the filter links.

### `src/routes/Todos/components/Footer.js`

Compare to [`components/Footer.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-components-footer-js).

```bash
mkdir -p src/routes/Todos/components
touch src/routes/Todos/components/Footer.js

# don't forget to make tests
mkdir -p tests/routes/Todos/components/
touch tests/routes/Todos/components/Footer.spec.js

# this doesn't work yet
redux g component Footer --path routes/Todos
```

```jsx
import React from 'react'
import FilterLink from '../../containers/FilterLink'

const Footer = () => (
  <p>
    Show:
    {" "}
    <FilterLink filter="SHOW_ALL">
      All
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_ACTIVE">
      Active
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_COMPLETED">
      Completed
    </FilterLink>
  </p>
)

export default Footer
```

### `src/routes/Todos/containers/FilterLink.js`
Compare to [`containers/FilterLink.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-containers-filterlink-js).

```bash
mkdir -p src/routes/Todos/containers
touch src/routes/Todos/containers/FilterLink.js

# don't forget to make tests
mkdir -p tests/routes/Todos/containers
touch tests/routes/Todos/containers/FilterLink.spec.js

# this doesn't work yet
redux g container FilterLink --path routes/Todos
```

```js
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../modules/Todos'
import Link from '../components/Link'

const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)

export default FilterLink
```

### `src/routes/Todos/components/Link.js`
Compare to [`components/Link.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-components-link-js).

```bash
mkdir -p src/routes/Todos/components
touch src/routes/Todos/components/Link.js

# don't forget to make tests
mkdir -p tests/routes/Todos/components
touch tests/routes/Todos/components/Link.spec.js

# this doesn't work yet
redux g component Link --path routes/Todos
```

```jsx
import React, { PropTypes } from 'react'

const Link = ({ active, children, onClick }) => {
  if (active) {
    return <span>{children}</span>
  }

  const _onClick = e => {
    e.preventDefault()
    onClick()
  }

  return (
    <a href='#'
      onClick={_onClick}
    >
      {children}
    </a>
  )
}

Link.propTypes = {
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func.isRequired
}

export default Link
```

*Note:* `_onClick` is pulled out to comply with the [eslint-plugin-react jsx-no-bind rule](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md). Because of a technicality of how functions are bound to `this` it is not wise to use a fat-arrow function inside of a JSX template.

## Todos Module
These are the files needed to manage the todos state.

### `src/routes/Todos/modules/todos.js`

Compare to [`actions/index.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-actions-index-js) and [`reducers/todos.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-reducers-todos-js).

```bash
mkdir -p src/routes/Todos/modules
touch src/routes/Todos/modules/todos.js

# don't forget to make tests
mkdir -p tests/routes/Todos/modules
touch tests/routes/Todos/modules/todos.spec.js

# this doesn't work yet
redux g module todos --path routes/Todos
```

```js
import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'
import { v4 as uuid } from 'node-uuid'

// Constants
export const ADD_TODO = 'ADD_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'
export const TOGGLE_TODO = 'TOGGLE_TODO'

// Action Creators
export const addTodo = createAction(ADD_TODO, text => ({ id: uuid(), text }))
export const setVisibilityFilter = createAction(SET_VISIBILITY_FILTER)
export const toggleTodo = createAction(TOGGLE_TODO)

// Reducers

// single todo
const todo = handleActions({
  [ADD_TODO]: (state, { payload }) => ({
    id: payload.id,
    text: payload.text,
    completed: false
  }),
  [TOGGLE_TODO]: (state, { payload }) => (
    state.id !== payload ? state : {
      ...state,
      completed: !state.completed
    }
  )
})

// todos list
export const todos = handleActions({
  [ADD_TODO]: (state, action) => ([
    ...state,
    todo(undefined, action)
  ]),
  [TOGGLE_TODO]: (state, action) => state.map(t => todo(t, action))
}, [])

// visibility filter
export const visibilityFilter = handleActions({
  [SET_VISIBILITY_FILTER]: (state, { payload }) => payload
}, 'SHOW_ALL')

export default combineReducers({
  todos,
  visibilityFilter
})
```

## Todos Form

### `src/routes/Todos/containers/AddTodo.js`

Compare to [`containers/AddTodo.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-containers-addtodo-js).

```bash
mkdir -p src/routes/Todos/containers
touch src/routes/Todos/containers/AddTodo.js

# don't forget to make tests
mkdir -p tests/routes/Todos/containers
touch tests/routes/Todos/containers/AddTodo.spec.js

# this doesn't work yet
redux g container AddTodo --path routes/Todos
```

```js
import React, { PropTypes } from 'react'
import { connect } from 'react-redux'
import { addTodo } from '../modules/Todos'

let AddTodo = ({ dispatch }) => {
  let input

  const onSubmit = e => {
    e.preventDefault()
    if (!input.value.trim()) {
      return
    }
    dispatch(addTodo(input.value))
    input.value = ''
  }

  const ref = node => {
    input = node
  }

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input ref={ref} />
        <button type='submit'>
          Add Todo
        </button>
      </form>
    </div>
  )
}
AddTodo.propTypes = {
  dispatch: PropTypes.func.isRequired
}
AddTodo = connect()(AddTodo)

export default AddTodo

```

## Todos List

### `src/routes/Todos/containers/VisibleTodoList.js`
Compare to [`containers/VisibleTodoList.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-containers-visibletodolist-js).

```bash
mkdir -p src/routes/Todos/containers
touch src/routes/Todos/containers/VisibleTodoList.js

# don't forget to make tests
mkdir -p tests/routes/Todos/containers
touch tests/routes/Todos/containers/VisibleTodoList.spec.js

# this doesn't work yet
redux g container VisibleTodoList --path routes/Todos
```

```js
import { connect } from 'react-redux'
import { fetchTodos, toggleTodo, saveTodo } from '../modules/Todos'
import TodoList from '../components/TodoList'

const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todosApp.todos, state.todosApp.visibilityFilter),
    loadingTodos: !!state.todosApp.isLoading
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    saveTodo: (id) => {
      dispatch(saveTodo(id))
    },
    fetchTodos: () => {
      dispatch(fetchTodos())
    },
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

### `src/routes/Todos/components/TodoList.js`
Compare to [`components/TodoList.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-components-todolist-js).

```bash
mkdir -p src/routes/Todos/components
touch src/routes/Todos/components/TodoList.js

# don't forget to make tests
mkdir -p tests/routes/Todos/components
touch tests/routes/Todos/components/TodoList.spec.js

# this doesn't work yet
redux g component TodoList --path routes/Todos
```

```jsx
import React, { PropTypes } from 'react'
import Todo from './Todo'

const TodoList = ({ todos, onTodoClick, loadingTodos }) => (
  <ul>
    {todos.map(todo =>
      <Todo
        key={todo.id}
        {...todo}
        onClick={function () { onTodoClick(todo.id) }}
      />
    )}
  </ul>
)

TodoList.propTypes = {
  todos: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired,
    text: PropTypes.string.isRequired
  }).isRequired).isRequired,
  loadingTodos: PropTypes.bool.isRequired,
  fetchTodos: PropTypes.func.isRequired,
  saveTodo: PropTypes.func.isRequired,
  onTodoClick: PropTypes.func.isRequired
}

export default TodoList
```

### `src/routes/Todos/components/Todo.js`
Compare to [`components/Todo.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-components-todo-js).

```bash
mkdir -p src/routes/Todos/components
touch src/routes/Todos/components/Todo.js

# don't forget to make tests
mkdir -p tests/routes/Todos/components
touch tests/routes/Todos/components/Todo.spec.js

# this doesn't work yet
redux g component Todo --path routes/Todos
```

```jsx
import React, { PropTypes } from 'react'

const Todo = ({ onClick, completed, text }) => (
  <li
    onClick={onClick}
    style={{
      textDecoration: completed ? 'line-through' : 'none'
    }}
  >
    {text}
  </li>
)

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  completed: PropTypes.bool.isRequired,
  text: PropTypes.string.isRequired
}

export default Todo
```







