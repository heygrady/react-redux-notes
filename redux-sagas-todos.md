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
You probably want to read about sagas to get [an authoratative definition](https://msdn.microsoft.com/en-us/library/jj591569.aspx) -- don't bother. In the simplest terms, redux-sagas is a task runner (it literally runs whatever function you give it and goes away). In a similar way to how a reducer is just a *function* that responds to an action, a saga is a *generator* that responds to an action. Although the terminology is incorrect, you can imagine that using redux-saga allows you to *subscribe* to actions and then *dispatch* more actions later. The classic example is using redux-saga to make a `fetch` request to a server and then dispatch and action to load the data into redux. To do this it utilizes generator functions. If you have been trying to get used to Promises, then you'll get the root concepts very easily. If you've used to callbacks you'll quickly see why this is better.

What is a saga good for? The primary reason for sagas is managing *asynchronous* behaviour. This is called a side-effect. There are other options for managing side effects in Redux. Redux-thunk, redux-effects, redux-side-effects, redux-promises... [check it out](https://www.google.com/search?q=best+redux+side+effects+library). Redux-saga utilizes generator functions which turns out to be the killer feature that makes it better than other solutions. Generator functions can be stopped and started which makes them ideal for asynchronous programming... and that's what we're tying to solve! While redux-saga doesn't do anything you couldn't do with a different solution, it does it in a much more concise and easy-to-follow way. It's set up to handle the most common case: managing actions that need to call a cascade of syncronous and asynchonous actions.

Another way to think about sagas is as action routers. They're not quite the same as asynchronous reducers because they don't update to the store, they only listen to actions.

Here's some important notes about sagas:

1. Sagas should not update the store directly, that's what a reducer is for.
2. Sagas are for fielding actions *before* they get to reducers, that's why it connects to redux as middleware.
3. Redux-saga uses weird terminology like `put` instead of `dispatch` and `takeEvery` instead of `handle`.
4. Iterating generators is not automatic but redux-saga manages that for you, that's why you normally don't have to call [`next()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/next) on your sagas, redux-sagas does that. Your saga should `yield` a [redux-saga effect](http://yelouafi.github.io/redux-saga/docs/basics/DeclarativeEffects.html).

## Install Redux Saga
Let's install [redux-saga](https://github.com/yelouafi/redux-saga):

```bash
npm install redux-saga --save

# you may want to install other common packages for working with modules
npm install redux-actions reselect --save
```

## Apply Saga Middleware
Redux Saga is middleware. If you use only Redux Saga for your asynchronous actions you should not need [Redux Thunk](https://github.com/gaearon/redux-thunk), the middleware that comes with react-redux-starter kit. However, we'll be keeping redux-thunk around for now. When you're working with Redux you need middleware to handle asynchronous actions, these are called side effects. That's why you see alternates to Redux Saga named things like [redux-effects](https://github.com/redux-effects/redux-effects) and [redux-side-effects](https://github.com/gregwebs/redux-side-effect). Redux Thunk works well for side effects but the other effects libraries make it [much easier to chain your effects](http://stackoverflow.com/questions/32925837/how-to-handle-complex-side-effects-in-redux). The biggest advantage of Redux Saga is that it uses native generator functions that are perfectly suited to the types of complex chained actions that you need for making asynchronous requests.

### Add sagas to createStore
We want to reuse the clever way that the react-redux-starter-kit loads the reducers for a route. You can read about it in detail [in the previous note](react-redux-starter-kit-todos.md#route-srcroutestodosindexjs). At a high level, react-redux-starter-kit provides an example of using Webpack to break routes into dynamic loading chunks. Part of that functionality involves loading the reducers for that route. This is helpful when you're using the [fractal project structure](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure).

Thankfully redux-saga has [support for dynamically loading sagas](https://github.com/yelouafi/redux-saga/releases/tag/v0.1.0) in a similar fashion to the `injectReducer()` method that comes with react-redux-starter-kit. You may like to read about [how dynamically loading reducers works](https://github.com/reactjs/redux/issues/37). There is an issue that describes [the need for dynamically loading sagas](https://github.com/yelouafi/redux-saga/issues/76). The basics look like this:

We need to import our rootSaga and the sagaMiddleware from our `sagas.js` file (we'll make that file next). We also need to run our middleware with the root saga before returning the store. Redux-saga requires you to "run" your sagas. It's not important how it works but you can't leave this step off.

**`src/store/createStore.js`** (compare to [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/createStore.js))

```js
// ...

import reducers from './reducers'
import rootSaga, { sagaMiddleware } from './sagas'

export default (initialState = {}, history) => {
  let middleware = applyMiddleware(thunk, routerMiddleware(history), sagaMiddleware)

  // ...

  sagaMiddleware.run(rootSaga)

  return store
}
```

#### Don't run an empty root saga
If you are not running any sagas app-wide then you can shorten the code above to simply apply the middleware. You might do this if your `src/store/sagas.js` file (see below) returns an empty saga. That might likely be the case if you're dynamically loading your sagas in your route.

**`src/store/createStore.js`** (without app-wide sagas)

```js
// ...

import reducers from './reducers'
import { sagaMiddleware } from './sagas'

export default (initialState = {}, history) => {
  let middleware = applyMiddleware(thunk, routerMiddleware(history), sagaMiddleware)

  // ...

  return store
}
```


### Add root sagas
We need to create our `sagas.js` file to provide similar functionality to the `reducers.js` file. We need to add an `injectSaga()` function that works very similarly to `injectReducer()`. Because of how redux-saga manages tasks we need to provide a `name` for our saga. Within redux-saga, the [`sagaMiddleware.run(saga)`](http://yelouafi.github.io/redux-saga/docs/api/index.html#middlewarerunsaga-args) function returns a [`task`](http://yelouafi.github.io/redux-saga/docs/api/index.html#task-descriptor). This is important because redux-saga simply runs your tasks. If you're not careful you might accidentally start the same saga twice. If you find yourself with double execution bugs, then you're probably running the same saga more than once. Thankfully we replicated similar logic to `injectReducer()` and calling `injectSaga()` twice with the same arguments has no effect. And if you call it twice with the same name and a different saga, then it will cancel the previous saga and run the new one.

We also need a `cancelTask(name)` function for cancelling our named tasks. This makes it possible to manage sagas dynamically from our routes. We can effectively run the saga on enter and cancel the saga on leave.

If you are using app-wide sagas, these are saga that aren't related to a specific route and should be running at all times, then you should import them in this file and include them in the array returned by the `rootSaga`.

**`src/store/sagas.js`**

```js
import createSagaMiddleware from 'redux-saga'

// import sync sagas here
// import { helloSaga, watchIncrementAsync } from '../redux/modules/example'

export const sagaMiddleware = createSagaMiddleware()

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

## Dynamically load a saga from a route
We need to dynamically load our saga in our route. 

**`src/routes/Todos/index.js`**

```js
import { injectReducer } from '../../store/reducers'
import { injectSaga, cancelTask } from '../../store/sagas'

export default (store) => ({
  path: 'todos',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const TodosView = require('./components/TodosView').default
      const module = require('./modules/todos')
      const reducer = module.default
      const saga = module.rootSaga

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

## Generic Module
Before we try to implement a saga in our sample app it's important to go over what a generic module looks like. A module is the redux version of a "model" from a traditional MVC app. Of course it's not *exactly* a model but you can see that all of the main pieces are there for working with data.

If you're following my anology, you use a selector in place of a getter. It reads a value. Techically you can store data in redux however you want and there are numerous ways to read data back. In practice you will probably be organizing your module to look similar to what you'd see with something like you'd see in an [Ember Data Model](http://emberjs.com/api/data/classes/DS.Model.html).

If a selector is a getter, then what is a setter? The short answer is "reducers" but that's not the whole story. Typically you don't call a reducer directly, instead you call an action which then gets dispatched to a reducer. In redux the simple act of updating an entity is turned into a complex maze of actions, thunks and sagas until it eventually reaches a reducer and updates the application state. Of course that's the power of redux. When you use something like Ember Data it is very easy to get and set a value on a model. But Ember Data itself does a tremendous amount of work to manage all of the underlying side effects -- sometimes it's difficult to get Ember Data to do what you want and you end up framework-fighting. In redux there's no magic going on and you have to manage those side effects yourself. That makes it slightly harder to get going but actually result in better performing code and completely removes framework-fighting.

1. Selectors are for reading data from the store. We use [reselect](https://github.com/reactjs/reselect).
2. Actions are the first step in writing to the store. We use [redux-actions](https://github.com/acdlite/redux-actions).
3. Sagas are for fielding actions that require async operations. We use [redux-saga](https://github.com/yelouafi/redux-saga).
4. Reducers are for writing to the store. We use [redux-actions](https://github.com/acdlite/redux-actions) for this too.
5. Selectors and Actions form the interface to your module. Anyone using your module in their code will import the selectors and actions you provide.

Here's a generic module that makes use of sagas:

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

**`src/routes/Todos/modules/todos.js`**

```js

```

