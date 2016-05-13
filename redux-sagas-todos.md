# Redux Saga Todos Tutorial
In this tutorial we're going to implement [Redux Saga](http://yelouafi.github.io/redux-saga/) in the same [Todos app from the previous document](./react-redux-starter-kit-todos.md).

The power of sagas doesn't become aparent until you hook it to an API, which we'll be doing in the next step. If we're not going to be using sagas correctly, why should you read this? Because setting up redux-saga with react-redux-starter kit isn't obvious and it's good to cover the fundamentals. This time around we're going to get sagas integrated into our app so that we can focus on the API portion of it later. If you try to use Redux Saga for the first time it can be intimidating because most tutorials jump right into [using redux-saga to interact with an API](https://github.com/yelouafi/redux-saga#usage-example). We're going to work up to it slowly.

If you want to get into Redux Saga they have a [great beginner's tutorial](http://yelouafi.github.io/redux-saga/docs/introduction/BeginnerTutorial.html). You may also want to read some of their [saga background links](http://yelouafi.github.io/redux-saga/docs/introduction/SagaBackground.html) which cover where the core concepts came from.

We'll cover it more later but Redux Saga makes extensive use of [generator functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*). You can read [in depth about generators](http://www.2ality.com/2015/03/es6-generators.html) if you'd like but [the basics](https://davidwalsh.name/es6-generators) become clear quite quickly.

### How do generators work?
Because generators are unfamiliar to most JavaScript developers, much of the documentation above will seem dense and confusing. Don't get discouraged. Generators are really easy.

Test this example out in the online [JavaScript REPL](https://repl.it/CQPk/1)

```js
// A generator function has a star
function * test () {
  // you mark the "steps" with yield
  yield 'hello'
  yield 'goodbye'
  return 'done' // <-- you don't normally return something this way, but you can
}

// calling a generator function returns a generator object
const task = test()

// you "walk" the generator using next
task.next() // --> { value: 'hello', done: false }
task.next() // --> { value: 'goodbye', done: false }
task.next() // --> { value: 'done', done: true }
```

Get it? You "walk" a generator function in "steps". The function "pauses" after each step.

1. Call a generator to get a [Generator object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator).
2. Call [`next()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/next) to step through each [`yield`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield)
3. The [`yield`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield) keyword means "return and pause", it's why you don't normally see a `return` in generator. If you do add in a `return` it will be used as the `value` on the very last `next()` call.
4. You should read about [Iterators and Generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)

### What is redux-saga?
You probably want to read about sagas to get [an authoratative definition](https://msdn.microsoft.com/en-us/library/jj591569.aspx) -- don't bother. Although there is an excellent write up on [why redux-saga is great](http://riadbenguella.com/from-actions-creators-to-sagas-redux-upgraded/). In the simplest terms (forget about generators for a second), redux-sagas is a task runner -- it literally runs whatever function you give it and goes away. We'll see later on that the main "software" that comes in the redux-saga package is the sagaMiddleware. Read some of these [saga resources](http://yelouafi.github.io/redux-saga/docs/ExternalResources.html) for more information.

You use it like this: [`sagaMiddleware.run(saga)`](http://yelouafi.github.io/redux-saga/docs/api/index.html#middlewarerunsaga-args). Remember that.

A `saga` is just a function and the saga middleware runs it once, later. The next time an action is dispatched in your app, your saga will be run. Your saga function recieves `action` as the first argument, pretty normal. Redux-saga works a little differently than other middleware in that the saga middleware is a reusable task runner.

That part is a little different than, say, redux-thunk where the asynchronous functionality gets baked into your action. In redux-saga the *saga* is the *target* of an action, more like a reducer. And like a reducer, it's useful to be able to chain sagas together in novel ways. This means you need to tell the sagaMiddleware about your saga, you do that with `sagaMiddleware.run(saga)`. Once you get going you almost never need to touch the middleware so it's ok if you don't really get it yet. If you find yourself working with the sagaMiddleware all the time you're probably doing something pretty advanced. You rarely need to do this for a standard use-case.

Usually you write your sagas to "stay alive" and keep listening for more actions, but you don't have to. In certain advanced use cases you might decide to run a bunch of one-off sagas to handle the actions you're about to fire off. But that's not the important detail. What's important is that your saga is just a function... a *generator* function.

In a similar way to how a reducer is just a *function* that responds to an action, a saga is just a *generator* that responds to an action. Although the terminology is incorrect, you can imagine that using redux-saga allows you to *subscribe* to actions. Of course the reason you want to subscribe to an action is to fire more actions! A saga is a great way to do a bunch of things in sequence, even if some of the things require waiting.

The classic example is [using redux-saga to make a `fetch()` request](http://yelouafi.github.io/redux-saga/docs/basics/UsingSagaHelpers.html). Because an [AJAX request](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) is asynchronous you can't handle it with the normal `dispatch->reducer` workflow. With asynchronous actions you have to use middleware to manage the flow; redux is strictly synchronous. In [the classic redux async example](http://redux.js.org/docs/advanced/ExampleRedditAPI.html) this means using thunks to dispatch a series of actions. After an initial `fetchSomething()` action is dispatched, your middleware needs to dispatch additional actions to update the application state as the promise completes. This all happens in your action and the results can start to look messy. It's a lot of work just to load data into redux. Redux-saga makes this much easier.

To manage asynchronous actions, redux-saga utilizes generator functions. Generators were *specifically* designed to manage asynchronous actions. If you have been trying to get used to Promises, then you'll get the root concepts right away. If you've used to callbacks you'll quickly see why this is better. A generator looks a lot like a [promise chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) or a [callback hell](http://callbackhell.com/).

```js
// a promise chain
fetchSomething()
  .then(result => {
    return firstThing(result)
  })
  .then(result => {
    return secondThing(result)
  })

// a callback hell
fetchSomething(result => {
  firstThing(result, result => {
    secondThing(result)
  })
})

// a generator function
function * doThingsWithSomething () {
  let result = yield fetchSomething()
  result = yield firstThing(result)
  yield secondThing(result)
}

const task = doThingsWithSomething()

// you can walk a generator
// (redux-saga does this for you)
let finished = false
while (!finished) {
  const { value, done } = task.next()
  finished = done

  // yield returns the value here too
  // (redux-saga wants this to be an effect)
  console.log( value )
}
```

### A saga
What is a saga good for? While redux-saga doesn't do anything you couldn't do with a different solution, it does it in a much more concise and easy-to-follow way. Redux-saga really starts to earn its stripes with the way it runs your generators -- [redux-saga has a glossary](http://yelouafi.github.io/redux-saga/docs/Glossary.html), you'll need it. As a core requirement, redux-saga expects you to yield an effect. Thankfully redux-saga comes with a bunch of helper functions that makes it seamless to [create effects](http://yelouafi.github.io/redux-saga/docs/api/index.html#effect-creators). In practice you'll use `put()` -- just an alias for dispatch -- to dispatch actions. And you'll use `call()` to call a function that either returns a generator or a promise. Here's the example above rewritten as a valid saga:

```js
// a saga
function * doThingsWithSomething () {
  let result = yield call(fetchSomething())
  result = yield put(firstThing(result))
  yield put(secondThing(result))
}
```

Here's some important notes about sagas:

1. Sagas should not update the store directly, that's what a reducer is for.
2. Sagas are for fielding actions *before* they get to reducers, that's why it connects to redux as middleware.
3. Redux-saga uses weird terminology like `put` instead of `dispatch` and `takeEvery` instead of `handle`.
4. Redux-saga manages your generators for you, that's why you normally don't have to call [`next()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/next) on your sagas. Your saga should `yield` an [effect](http://yelouafi.github.io/redux-saga/docs/basics/DeclarativeEffects.html) in order to tell redux-saga how to run your generator.

**A saga is a generator that yields effects.**

## Install Redux Saga
Let's install [redux-saga](https://github.com/yelouafi/redux-saga):

```bash
npm install redux-saga --save

# you may want to install other common packages for working with modules
npm install redux-actions reselect --save
```

## Apply Saga Middleware
Redux Saga is middleware. If you use only Redux Saga for your asynchronous actions you should not need [redux-thunk](https://github.com/gaearon/redux-thunk), the middleware that comes with react-redux-starter kit. However, we'll be keeping redux-thunk around for now. When you're working with Redux you need middleware to handle asynchronous actions, these are called side effects. That's why you see alternates to redux-saga named things like [redux-effects](https://github.com/redux-effects/redux-effects) and [redux-side-effects](https://github.com/salsita/redux-side-effects). Redux-thunk works well for side effects but the other effects libraries make it [much easier to chain your effects](http://stackoverflow.com/questions/32925837/how-to-handle-complex-side-effects-in-redux). The biggest advantage of redux-saga is that it uses native generator functions that are perfectly suited to the types of complex chained actions that you need for making asynchronous requests.

### Add redux-saga to createStore
We want to reuse the clever way that react-redux-starter-kit loads the reducers for a route. You can read about it in detail [in the previous note](react-redux-starter-kit-todos.md#route-srcroutestodosindexjs). At a high level, react-redux-starter-kit provides an example of using Webpack to break routes into dynamic chunks. Part of that functionality involves loading the reducers for that route. This is helpful when you're using the [fractal project structure](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure).

Thankfully redux-saga has [support for dynamically loading sagas](https://github.com/yelouafi/redux-saga/releases/tag/v0.1.0) in a similar fashion to the `injectReducer()` method that comes with react-redux-starter-kit. You may like to read about [how dynamically loading reducers works](https://github.com/reactjs/redux/issues/37). There is an issue that describes [the need for dynamically loading sagas](https://github.com/yelouafi/redux-saga/issues/76). The basics look like this:

We need to import our rootSaga and the sagaMiddleware from our `sagas.js` file (we'll make that file next). We also need to run our middleware with the root saga before returning the store. Redux-saga requires you to "run" your sagas. It's not important how it works but you can't skip this step.

##### `src/store/createStore.js` (compare to [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/createStore.js))

```js
// ...

// we're already importing our reducers
import reducers from './reducers'

// we also need to import our sagas
import rootSaga, { sagaMiddleware } from './sagas'

// this is the createStore function
export default (initialState = {}, history) => {
  // we add our sagaMiddleware last, but order might not matter
  let middleware = applyMiddleware(thunk, routerMiddleware(history), sagaMiddleware)

  // ...

  // we need to run our saga! don't forget!
  sagaMiddleware.run(rootSaga)

  return store
}
```

#### Don't run an empty root saga
If you are not running any sagas app-wide then you can shorten the code above to simply apply the middleware. You might do this if your `src/store/sagas.js` file (see below) returns an empty saga.

##### `src/store/createStore.js` (without app-wide sagas)

```js
// ...

import reducers from './reducers'
import { sagaMiddleware } from './sagas'

export default (initialState = {}, history) => {
  let middleware = applyMiddleware(thunk, routerMiddleware(history), sagaMiddleware)

  // ...

  // purposely didn't run our root saga because we don't use one (common)
  // sagaMiddleware.run(rootSaga)

  return store
}
```


### Add root sagas
We need to create our `src/store/sagas.js` file to provide similar functionality to the `src/store/reducers.js` file. The point of this file is to provide an interface for managing sagas like the `reducers.js` file provides an interface for managing reducers. We need to add an `injectSaga()` function that works similarly to `injectReducer()`.

Within redux-saga, the [`sagaMiddleware.run(saga)`](http://yelouafi.github.io/redux-saga/docs/api/index.html#middlewarerunsaga-args) function returns a [`task`](http://yelouafi.github.io/redux-saga/docs/api/index.html#task-descriptor). This is important because redux-saga simply runs your tasks. If you're not careful you might accidentally start the same saga twice. If you find yourself with double execution bugs, then you're probably running the same saga more than once. Thankfully we replicated similar logic to `injectReducer()` and calling `injectSaga()` twice with the same arguments has no effect. And if you call it twice with the same name and a different saga, then it will cancel the previous saga and run the new one.

We also need a `cancelTask(name)` function for cancelling our named tasks. This makes it possible to manage sagas dynamically from our routes. We can effectively run the saga on enter and cancel the saga on leave.

##### `src/store/sagas.js`

```js
import createSagaMiddleware from 'redux-saga'

// import sync sagas here
// import { helloSaga, watchIncrementAsync } from '../redux/modules/example'

export const sagaMiddleware = createSagaMiddleware()
export const runSaga = (saga) => sagaMiddleware.run(saga)

// use injectSaga() to import from a route
const tasks = {}
export const injectSaga = ({ name, saga }) => {
  let { task, prevSaga } = tasks[name] || {}

  if (task && prevSaga !== saga) {
    cancelTask(name)
    task = undefined
  }

  if (!task || !task.isRunning()) {
    tasks[name] = {
      task: sagaMiddleware.run(saga),
      prevSaga: saga
    }
  }
}

export const cancelTask = (name) => {
  const { task } = tasks[name]
  tasks[name] = undefined

  if (task) {
    task.cancel()
  }
}

export default function * rootSaga () {
  yield [
    // Add sync sagas here
    // helloSaga(),
    // watchIncrementAsync()
  ]
}
```

Let's dig into this step-by-step.

#### You can import and run app sagas
The [`src/store/reducers.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/reducers.js) file is where you'd import a reducer that all of your app would use. Otherwise you'd import reducers within your route. The starter kit calls them "sync reducers" because they are loaded synchronously as the app loads. "Async reducers" are loaded in a route, asynchronously. We're going to copy that format and load our "sync sagas" in the `src/store/sagas.js` file. I want to refer to these types of reducers and sagas are "app sagas" because they're loaded at the app level. As opposed to "route" sagas and reducers which are loaded at the route level.

```js
// sync sagas
import { helloSaga, watchIncrementAsync } from '../redux/modules/example'

// ...

export default function * rootSaga () {
  yield [
    // Add sync sagas here
    helloSaga(),
    watchIncrementAsync()
  ]
}

```

#### Export the sagaMiddleware
This is where we created the middleware we used in `src/store/createStore.js`. We create an `injectSaga()` function so that we don't have to muck with the middleware directly outside of `createStore.js`.

```js
// ...

// this gets used in applyMiddleware()
// we also use it inside of injectSaga() and runSaga()
export const sagaMiddleware = createSagaMiddleware()

// if we just want to run a single saga, we can use this
// (usually when you're doing something clever and one-off)
export const runSaga = (saga) => sagaMiddleware.run(saga)

// ...
```

#### Inject some sagas
Here we're bending some of the core terminology of redux-saga to be more in line with what we see elsewhere in react-redux-starter-kit. At it's core, `injectSaga()` is doing the same thing as `sagaMiddleware.run(saga)`. Remember that? This is the same thing but better. Usually you need to launch a "watcher" saga when you enter a route and kill it when you leave a route (more on this later). We'll see clearly how you use this in the next step so you don't have to understand this fully yet. You only need this when you're setting up a route.

```js
// list of tasks, keyed by name
const tasks = {}

// use injectSaga() to run a saga from a route
export const injectSaga = ({ name, saga }) => {

  // get our task by name
  let { task, prevSaga } = tasks[name] || {}

  // cancel running tasks
  if (task && prevSaga !== saga) {
    cancelTask(name)
    task = undefined
  }

  // running a saga returns a task
  if (!task || !task.isRunning()) {
    tasks[name] = {
      task: sagaMiddleware.run(saga),
      prevSaga: saga
    }
  }
}

// use cancelTask() to cancel by name
export const cancelTask = (name) => {

  // get our task by name
  const { task } = tasks[name]

  // delete it
  tasks[name] = undefined

  // cancel it
  if (task) {
    task.cancel()
  }
}
```

## Dynamically load a saga from a route
We need to dynamically load our saga in our route. This is identical to the [previous example for our todo route](./react-redux-starter-kit-todos.md#route-srcroutestodosindexjs). Or, we're going to be making changes to that file to make it support our sagas. This will be pretty standard on any route you create that needs to implement sagas.

##### `src/routes/Todos/index.js`

```js
import { injectReducer } from '../../store/reducers'
import { injectSaga, cancelTask } from '../../store/sagas'

export default (store) => ({
  path: 'todos',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const TodosView = require('./components/TodosView').default
      const { default: reducer, saga } = require('./modules/todos')

      injectReducer(store, { key: 'todos', reducer })
      injectSaga({ name: 'todos', saga })

      cb(null, TodosView)
    }, 'todos')
  },
  onLeave () {
    cancelTask('todos')
  }
})
```

#### How this works
Because we're using Webpack's `require.ensure`, we have to `require` instead of import. Check out [how to rename a variable with destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Assigning_to_new_variable_names). We'll see later how we construct a typical module. It's best to follow the convention of exporting your reducer by default and exporting your modules sagas as `rootSaga`. We'll see in a second what that looks like, but it's basically what we do in the `src/store/sagas.js`.

```js
// ...

// we use require to, umm, import our module
// we pull out the default reducer
// and we grab the root saga
const { default: reducer, rootSaga: saga } = require('./modules/todos')

// we inject the reducer as normal
injectReducer(store, { key: 'todos', reducer })

// we also run our root saga
injectSaga({ name: 'todos', saga })

// ...
```

## Generic Module
Before we try to implement a saga in our sample app it's important to go over what a generic module looks like. A module is the redux version of a "model" from a traditional MVC app. Of course it's not *exactly* a model but you can see that all of the main pieces are there. Typically a model has getters and setters.

If you're following my anology, you use a selector in place of a getter. We use [reselect](https://github.com/reactjs/reselect) for this. It reads a value from the store. Techically you can store data in redux however you want and there are numerous ways to read data back. In practice you will probably be organizing your module to look similar to something like an [Ember Data Model](http://emberjs.com/api/data/classes/DS.Model.html). Regardless of how you structure your modules, using reselect to read from the store is highly recommended. The [documentation provided](https://github.com/reactjs/reselect) is top-notch.

If a selector is a getter, then what is a setter? The short answer is "reducers" but that's not the whole story. Typically you don't call a reducer directly, instead you call an action which then gets dispatched to a reducer. In redux the simple act of updating an entity is turned into a complex maze of actions, thunks and sagas until it eventually reaches a reducer and updates the application state. Of course that's the power of redux. When you use something like Ember Data it is very easy to get and set a value on a model. But Ember Data itself does a tremendous amount of work to manage all of the underlying side effects without you needing to worry. In redux there's no magic going on and you have to manage those side effects yourself. That makes it slightly harder to get going but actually results in better performing code and completely removes framework-fighting (bending over backwards to get the framework to do what you want).

1. Selectors are for reading data from the store. We use [reselect](https://github.com/reactjs/reselect).
2. Actions are the first step in writing to the store. We use [redux-actions](https://github.com/acdlite/redux-actions).
3. Sagas are for fielding actions that require async operations. We use [redux-saga](https://github.com/yelouafi/redux-saga).
4. Reducers are for writing to the store. We use [redux-actions](https://github.com/acdlite/redux-actions) for this too.
5. Selectors and Actions form the interface to your module. Anyone using your module in their code will import the selectors for use in `mapStateToProps` and the actions for use in `mapDispatchToProps`.

Here's a generic module that makes use of sagas.

```js
import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'
import { createSelector } from 'reselect'
import { takeEvery, delay } from 'redux-saga'
import { put } from 'redux-saga/effects'

// selectors
export const getResult => (state) => state.myReducer.result
export const getPending => (state) => state.myReducer.pending

export const getFinalResult = createSelector(
  [ getResult, getPending ],
  (result, pending) => !!pending ? result : undefined
)

// actions and action creators
export const MY_ACTION = 'MY_ACTION'
export const myAction = createAction(MY_ACTION)

export const MY_ASYNC_ACTION = 'MY_ASYNC_ACTION'
export const myAsyncAction = createAction(MY_ASYNC_ACTION)

export const MY_BETTER_ASYNC_ACTION = 'MY_BETTER_ASYNC_ACTION'
export const myBetterAsyncAction = createAction(MY_BETTER_ASYNC_ACTION)

const STARTED_BETTER_ASYNC_ACTION = 'STARTED_BETTER_ASYNC_ACTION'
const startedBetterAsyncAction = createAction(STARTED_BETTER_ASYNC_ACTION)

const FINISHED_BETTER_ASYNC_ACTION = 'FINISHED_BETTER_ASYNC_ACTION'
const finishedBetterAsyncAction = createAction(FINISHED_BETTER_ASYNC_ACTION)

// sagas
function * mySaga ({ payload }) {
  yield delay(1000) // <-- returns an effect that proceeds after a delay
  yield put(myAction(payload)) // <-- returns an effect that calls an action
}

function * myBetterSaga ({ payload }) {
  const started = yield put(startedBetterAction(payload))
  yield delay(1000)
  yield put(myAction(payload))
  yield put(finishedBetterAction(started.payload))
}

// combine sagas
export function * rootSaga () {
  yield [
    // combine all of your module's sagas
    yield * takeEvery(MY_ASYNC_ACTION, mySaga), // <-- calls the mySaga generator on MY_ASYNC_ACTION
    yield * takeEvery(MY_BETTER_ASYNC_ACTION, myBetterSaga)
  ]
}

// reducers
const myReducer = handleActions({
  [MY_ACTION]: (state, { payload }) => ({
    ...state,
    result: payload
  }),
  [STARTED_BETTER_ASYNC_ACTION]: (state, action) => ({
    ...state,
    pending: true
  }),
  [FINISHED_BETTER_ASYNC_ACTION]: (state, action) => ({
    ...state,
    pending: false
  })
}, {})

// combine reducers
export default combineReducers({
  // combine all of your module's reducers
  myReducer // <-- recieves store.myReducer as store
})
```




## Add sagas to a module

##### `src/routes/Todos/modules/todos.js`

```js

```

