# React Redux Starter Kit Todos Tutorial
This is my example of using [react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit) and [redux-cli](https://github.com/SpencerCDixon/redux-cli) to replicate [the Todos app from the Redux manual](http://redux.js.org/docs/basics/ExampleTodoList.html). I'll go through how I wish the CLI worked because in many places the CLI doesn't match the recommended way to develop a proper app. Or... it doesn't yet support routes or the [fractal design pattern](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure) of routes. This is partially because the [blueprints included with react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit/tree/master/blueprints) don't include routes. Partially because [you can't add a `--path` option](https://github.com/SpencerCDixon/redux-cli/issues/72) (like `redux g module myName --path routes/my-route`) with redux-cli yet. It's probably worse that *components*, *containers* and *modules* are called *dumb*, *smart* and *duck* in the default blueprints. But we can fix that ourselves for our own projects.

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
You need to install [redux-cli](https://github.com/SpencerCDixon/redux-cli) as a global npm package. This is currently a barebones command-line tool but it does a few really nice things for us.  Right now it's mostly useful for initializing a new project but it's clear that this tool will be expanding over time. The major innovation of redux-cli tool over something like [Yeoman](http://yeoman.io/generators/) is that the blueprints you use in your project are checked into your project. There is a strong encouragement for you to [manage the blueprints in your project](https://github.com/SpencerCDixon/redux-cli#creating-blueprints). This is actually a key advancement that is possibly easy to overlook. Being able to easily customize the default generators and add your own is a huge advancement.

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

#navigate into your app
cd todos-app
```

You can poke around if you'd like. You should read about [what's in the box](https://github.com/davezuko/react-redux-starter-kit) and [how to use it](https://suspicious.website/2016/04/29/starting-out-with-react-redux-starter-kit/) if you haven't already. For our purposes we'll be doing everyhting in the `src/` folder.

## Create a new route
It's currently [not possible to add a route with redux-cli](https://github.com/SpencerCDixon/redux-cli/issues/6) but that should be getting fixed eventually. For now we'll have to make it by hand. We're creating a Todo app so we'll create a "Todo" route. We're presuming that you're fairly new to React-Redux so we'll be leaving the boilerplate routes in place for now because they're a good reference. There's a few steps involved in creating a new route and it's ok if youdon't fully understand routes right away. It's important to know that we're using [react-router](https://github.com/reactjs/react-router) with the [react-router-redux](https://github.com/reactjs/react-router-redux) bindings.

If you're new to react-router they recommend that you:
- [Read the react-router tutorial **first**](https://github.com/reactjs/react-router-tutorial)
- [Read the react-router docs](https://github.com/reactjs/react-router/tree/master/docs)

We're actually just going to be copying what the included [`Counter` route](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/Counter/index.js) contains.

### Create the files for your route
We're going to eventually be storing our components, containers and modules using the fractal project structure so we'll be creating `components/`, `containers/` and `modules/` folders inside our `routes/Todos/` folder. *Hint:* [`mkdir -p <path>`](http://unix.stackexchange.com/questions/49263/recursive-mkdir).

```bash
mkdir -p src/routes/Todos/components
mkdir -p src/routes/Todos/containers
mkdir -p src/routes/Todos/modules
touch src/routes/Todos/components/TodosView.js
touch src/routes/Todos/index.js
```

##### `src/routes/Todos/index.js`
We need to add some boilerplate code to your route file. You can see the way the router itself is configured by checking in the [`src/routes/index.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/index.js) file. By default your app is configured to allow for lazy-loading of routes. This is an important consideration for large apps and it's easy enough to support it from the very beginning. You don't have to know exactly how all of this magic works but you need understand that it's important. Under the hood we're [using Webpack's code-splitting feature](https://webpack.github.io/docs/code-splitting.html) to make sure the code bundles created by Webpack are as small as possible.

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

2. We need to specify a [`getComponent(nextState, cb)`](https://github.com/reactjs/react-router/blob/master/docs/API.md#getcomponentnextstate-callback) function in order to lazy-load our route. This is specified as being designed for code-splitting in the react-router manual. We're going to be using [Webpack for code-splitting](https://webpack.github.io/docs/code-splitting.html). If you want to specify a route without using aync loading you can use [`component`](https://github.com/reactjs/react-router/blob/master/docs/API.md#component) instead.

  ```js
  // ...

  export default (store) => ({
    path: 'todos',

    // this is for async loading a route
    // @see https://github.com/reactjs/react-router/blob/master/docs/API.md#getcomponentnextstate-callback
    getComponent (nextState, cb) {

      // ...

      // be sure to pass your component to the callback
      // null means there weren't any errors
      cb(null, TodosView)

      // ...
    }
  })
  ```

```js
// ...

// you should load your view normally
// if you don't need async for some reason
import TodosView from './components/TodosView'
export default (store) => ({
  path: 'todos',

  // it's pretty straightforward
  // if you don't need code-splitting
  component: TodosView
})
```

3. We need to name our [name our Webpack chunk](https://github.com/webpack/webpack/tree/master/examples/named-chunks). Webpack supports [code-splitting](https://webpack.github.io/docs/code-splitting.html) through a specialty `require.ensure()` function. If you've been reading about Node then you've probably heard about using the `require()` function for [loading modules](https://nodejs.org/api/modules.html). If you've been reading about ES6 then you' probably know you should *usually* [use `import` instead of `require`](http://stackoverflow.com/questions/31354559/using-node-js-require-vs-es6-import-export). The `require.ensure()` function is particular to Webpack, not Node, and solves a specific use-case for lazy-loading parts of your app for performance reasons. If it bothers you that we're [using `require` instead of `import`](https://webpack.github.io/docs/code-splitting.html#es6-modules) that's a good instinct but in this case `require` is the only way to do what we need.

  ```js
  // ...

  export default (store) => ({
    path: 'todos',
    getComponent (nextState, cb) {

      // require.ensure is a special Webpack thing
      // @see https://webpack.github.io/docs/code-splitting.html
      require.ensure([], (require) => {

        // require.ensure passes the require function for us to use.
        //...

      }, 'todos') // <-- this names the webpack chunk for this route
    }
  })
  ```

4. We need to import our view and our module using the `require` function passed to us by ` require.ensure`. You'll notice that we're [tacking a `.default` on the end of our `require()` function](https://github.com/esnext/es6-module-transpiler/issues/85). That's required to cover a gap in how `import` works vs `require`. You can read in depth about [ES6 modules](http://www.2ality.com/2014/09/es6-modules-final.html) if you'd like. The key take away is that... if you want to import an ES6 module pragmatically you have to use `require()`. And if you use `require()` to import an ES6 module, you will receive an object where each key matches one of your named exports (ES6 modules support [named exports](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) and `require()` doesn't). In this case we're trying to use the `default` export, so we need to use the `.default` property. If you've ever wondered [how to import every file in a folder in Node](http://stackoverflow.com/questions/5364928/node-js-require-all-files-in-a-folder), that's what `require()` is for.

  ```js
  // ...

  const TodosView = require('./components/TodosView').default // <-- you must specify .default
  const reducer = require('./modules/todos').default

  // ...
  ```

5. We need to inject our reducer into the store using the `injectReducer(store, map)` function. This function comes with the react-redux-starter-kit in the [`src/store/reducers.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/reducers.js) file. This construct allows for lazy-loaded routes to add reducers to the Redux store after the initial page load. There's not much magic going on there but if you poke around and see `asyncReducers` mentioned in the code it's referring to the reducer from a route that was loaded as an asynchonous chunk by Webpack. **Important:** the `key` specifies a key on the store that the reducers in your route will inherit from. It's similar in concept to what [`combineReducers()`](http://redux.js.org/docs/api/combineReducers.html) does.

  ```js
  import { injectReducer } from '../../store/reducers'

  // ...

  // make sure to give your reducer a good name
  injectReducer(store, { key: 'todos', reducer })

  // ...
  ```

##### `src/routes/Todos/components/TodosView.js`
