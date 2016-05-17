# redux-saga Todos Tutorial
In this tutorial we're going to implement [redux-saga](http://yelouafi.github.io/redux-saga/) in the same [Todos app from the previous document](./react-redux-starter-kit-todos.md).

The power of sagas doesn't become aparent until you hook it to an API, which we'll be doing in the next step. If we're not going to be using sagas correctly, why should you read this? Because setting up redux-saga with react-redux-starter kit isn't obvious and it's good to cover the fundamentals. This time around we're going to get sagas integrated into our app so that we can focus on the API portion of it later. If you try to use redux-saga for the first time it can be intimidating because most tutorials jump right into [using redux-saga to interact with an API](https://github.com/yelouafi/redux-saga#usage-example). We're going to work up to it slowly.

If you want to get into redux-saga they have a [great beginner's tutorial](http://yelouafi.github.io/redux-saga/docs/introduction/BeginnerTutorial.html). You may also want to read some of their [saga background links](http://yelouafi.github.io/redux-saga/docs/introduction/SagaBackground.html) which cover where the core concepts came from.

We'll cover it more later but redux-saga makes extensive use of [generator functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*). You can read [in depth about generators](http://www.2ality.com/2015/03/es6-generators.html) if you'd like but [the basics](https://davidwalsh.name/es6-generators) become clear quite quickly.

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

A `saga` is just a function and the saga middleware runs it once, later. The next time an action is dispatched in your app, your saga will be run. Your saga function recieves `action` as the first argument, pretty normal. Redux-saga works a little differently than other middleware because sagas run outside of redux.

In a normal middleware like redux-thunk the asynchronous functionality gets baked into your action. In redux-saga it's different. **The *saga* is the *target* of an action**, more like a reducer. The `sagaMiddleware` is the bridge between dispatched actions and your saga. Once you get going you almost never need to touch the middleware outside of a route so it's ok if you don't really get it yet. If you find yourself working with the sagaMiddleware all the time you're probably doing something pretty advanced.

Because sagas run independently from redux, you need to manage them in peculiar ways. Again, once you get going it's pretty seamless but it's important to know that **your saga won't do anything until you run it.** Usually you configure your sagas to "stay alive" and keep listening for more actions (more on that later), but you don't have to do it that way. In certain advanced use cases you might decide to run a bunch of one-off sagas for a single request. But that's not the important detail. What's important is that **your saga is just a function... a *generator* function.**

In a similar way to how a reducer is just a *function* that responds to an action, **a saga is just a *generator* that responds to an action.** Although the terminology is incorrect, you can imagine that using redux-saga allows you to *subscribe* to actions. Of course the reason you want to subscribe to an action is to fire more actions! A saga is a great way to do a bunch of things in sequence, even if some of the things require waiting.

The classic example is [using redux-saga to make a `fetch()` request](http://yelouafi.github.io/redux-saga/docs/basics/UsingSagaHelpers.html). Because an [AJAX request](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) is asynchronous you can't handle it with the normal `dispatch->reducer` workflow. With asynchronous actions you have to use middleware to manage the flow; redux is strictly synchronous. An async flow looks more like like `dispatch->middleware( wait )->reducer`. After an initial `fetchSomething()` action is dispatched, your middleware needs to dispatch additional actions to update the application state as the promise completes. In [the classic redux async example](http://redux.js.org/docs/advanced/ExampleRedditAPI.html) this means using thunks to dispatch a series of actions. When you use thunks everything happens in your action and the resulting code can start to look messy. Redux-saga makes this much easier.

**A saga is the target of an action, not the action itself.** Sagas usually dispatch additional actions after waiting for an asynchronous action.

### Different ways to manage async actions in JavaScript
To manage asynchronous actions, redux-saga utilizes generator functions. Generators were *specifically* designed to manage asynchronous actions. If you have been trying to get used to Promises, then you'll get the root concepts right away. If you're used to callbacks you'll quickly see why this is better. A generator looks a lot like a [promise chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) or a [callback hell](http://callbackhell.com/).

##### A promise chain
In a promise chain you can keep adding `then()` functions to a promise. It ensures that your additional actions execute only after the initial promise has resolved. Check out a more [complete example](https://repl.it/CR47/9)

```js
// a promise chain
fetchSomething()
  .then(result => {
    return firstThing(result)
  })
  .then(result => {
    return secondThing(result)
  })
```

##### A callback hell
You should read about [callback hell](http://callbackhell.com/) and also how [reactive programming is better](http://stackoverflow.com/questions/25098066/what-is-callback-hell-and-how-and-why-rx-solves-it). Check out a more [complete example](https://repl.it/CR47/8).

```js
// a callback hell
fetchSomething(result => {
  firstThing(result, result => {
    secondThing(result)
  })
})
```

##### A generator function
Generators are the core of reactive programming. It's basically the same thing as a callback hell or a promise chain but it gives the user much more control. You have to walk your generators (see below). Check out a more [complete example](https://repl.it/CR47/11).

```js
// a generator function
function * doThingsWithSomething () {
  let result
  result = yield fetchSomething()
  result = yield firstThing(result)
  yield secondThing(result)
}

const task = doThingsWithSomething()
```

##### Walking a generator function
Generators give you a lot of control from the outside. It's subtle in the above example but the value captured in `result = yield fetchSomething()` is actually passed in from the outside using the [`next()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/next) function. Internally redux-saga takes full advantage of this, except it passes in the result from the previous effect. If this doesn't make sense don't worry -- redux-saga does this for you!

```js
// you can walk a generator
// (redux-saga does this for you)
function walkTask(task, result = {}) {
  let value

  while (!result.done) {
    result = task.next(result.value) // <-- call next() with the previously yielded value
    
    // redux-saga waits for promises to finish
    if (typeof result.value.then === 'function') {
      return result.value.then( (response) => {
        result.value = response // <-- override the value with the promise response
        return walkTask(task, result) // <-- resume walking
      })
    }
  }

  return result.value
}

walkTask(task)
```

*Note:* The `walkTask()` function above is just a toy example. Redux-saga actually analyzes the effect you yield and does some magic. However, once redux-saga is done with its internal magic it simply calls `next()` with the result of the previous effect.

### Managing async actions with a saga
What is a saga good for? Chaining asynchronous actions.

While redux-saga doesn't do anything you couldn't do with a different solution, it allows you to chain actions in an easy-to-follow way. [Redux-saga has a glossary](http://yelouafi.github.io/redux-saga/docs/Glossary.html), you'll need it. As a core requirement **redux-saga expects you to yield an effect**. Thankfully redux-saga comes with a bunch of helper functions that makes it seamless to [create effects](http://yelouafi.github.io/redux-saga/docs/api/index.html#effect-creators). In practice you'll use [`put()`](http://yelouafi.github.io/redux-saga/docs/api/index.html#putaction) -- just an alias for dispatch -- to dispatch actions. And you'll use [`call()`](http://yelouafi.github.io/redux-saga/docs/api/index.html#callfn-args) to call a function that does something asynchronously, either returning a generator or a promise. 

##### A saga
Here's the async example from above rewritten as a saga:

```js
// a saga
function * doThingsWithSomething () {
  let result
  result = yield call(fetchSomething()) // <-- presume fetchSomething returns a value, promise or generator
  result = yield put(firstThing(result)) // <-- presume firstThing is an action
  yield put(secondThing(result))
}
```

*Note:* The saga above is not quite right. A Saga should only dispatch actions. Don't worry, you likely won't need to chain results like this. It's just an example that looks similar to the generic examples above.

Here's some important notes about sagas:

1. Sagas should not update the store directly, that's what a *reducer* is for.
2. Sagas are for fielding actions *before* they get to reducers, that's why redux-saga provides middleware.
3. Redux-saga uses weird terminology like `put` instead of `dispatch` and `takeEvery` instead of `handle`.
4. Redux-saga manages your generators for you, that's why you normally don't have to call [`next()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/next) on your sagas. Your saga should `yield` an [effect](http://yelouafi.github.io/redux-saga/docs/basics/DeclarativeEffects.html) in order to tell redux-saga how to run your generator.

**A saga is a generator that yields effects.**

# Install redux-saga
Let's install [redux-saga](https://github.com/yelouafi/redux-saga):

```bash
npm install redux-saga --save

# you may want to install other common packages for working with modules
npm install redux-actions reselect --save
```

## Apply Saga Middleware
Redux-saga is middleware. When you're working with Redux you need middleware to handle asynchronous actions, these are called side effects. That's why you see alternates to redux-saga named things like [redux-effects](https://github.com/redux-effects/redux-effects) and [redux-side-effects](https://github.com/salsita/redux-side-effects). Redux-thunk works well for side effects but the other effects libraries make it [much easier to chain your effects](http://stackoverflow.com/questions/32925837/how-to-handle-complex-side-effects-in-redux). The biggest advantage of redux-saga is that it uses native generator functions that are perfectly suited to the types of complex chained actions that you need for making asynchronous requests.

*Note:* If you use redux-saga for your asynchronous actions you shouldn't need [redux-thunk](https://github.com/gaearon/redux-thunk), the middleware that comes with react-redux-starter kit. However we'll be keeping redux-thunk around for now.

### Add redux-saga to createStore
We want to reuse the clever way that react-redux-starter-kit loads the reducers for a route. You can read about it in detail [in the previous note](react-redux-starter-kit-todos.md#route-srcroutestodosindexjs). At a high level, react-redux-starter-kit provides an example of using Webpack to break routes into dynamic chunks. Part of that functionality involves loading the reducers for that route. This is helpful when you're using the [fractal project structure](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure).

Thankfully redux-saga has [support for dynamically loading sagas](https://github.com/yelouafi/redux-saga/releases/tag/v0.1.0) in a similar fashion to the `injectReducer()` method that comes with react-redux-starter-kit. You may like to read about [how dynamically loading reducers works](https://github.com/reactjs/redux/issues/37). There is an issue that describes [the need for dynamically loading sagas](https://github.com/yelouafi/redux-saga/issues/76). The basics look like this:

We need to import our rootSaga and the sagaMiddleware from our `sagas.js` file (we'll make that file next). We also need to run our middleware with the root saga before returning the store. Redux-saga requires you to "run" your sagas. It's not important how it works but you can't skip this step.

**You need to add some code to your `createStore.js` to integrate the sagaMiddleware.**

##### `src/store/createStore.js`
(compare to [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/createStore.js))

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
If you are not running any sagas app-wide then you can shorten the code above to simply apply the middleware. You might do this if your `src/store/sagas.js` file (see below) returns an empty root saga.

##### `src/store/createStore.js` (without app-wide sagas)
This shows the same alterations to `createStore.js` above with the `rootSaga` commented out. In many cases you don't need a rootSaga at the application level. It all depends on your app.

```js
// ...

import reducers from './reducers'
import { sagaMiddleware } from './sagas'

export default (initialState = {}, history) => {
  let middleware = applyMiddleware(thunk, routerMiddleware(history), sagaMiddleware)

  // ...

  // purposely didn't run our root saga because we only run sagas from our routes (common)
  // sagaMiddleware.run(rootSaga)

  return store
}
```


### Add root sagas
We need to create our `src/store/sagas.js` file to provide similar functionality to the `src/store/reducers.js` file. The point of this file is to provide an interface for managing sagas like the `reducers.js` file provides an interface for managing reducers. We need to add an `injectSaga()` function that works similarly to `injectReducer()`.

Within redux-saga, the [`sagaMiddleware.run(saga)`](http://yelouafi.github.io/redux-saga/docs/api/index.html#middlewarerunsaga-args) function returns a [`task`](http://yelouafi.github.io/redux-saga/docs/api/index.html#task-descriptor). This is important because redux-saga simply runs your tasks. If you're not careful you might accidentally start the same saga twice. If you find yourself with double execution bugs, then you're probably running the same saga more than once. Thankfully we replicated similar logic to `injectReducer()` and calling `injectSaga()` twice with the same arguments prevents double execution. You can also swap out a saga by injecting a news saga with an existing name.

We also need a `cancelTask(name)` function for canceling our named tasks. This makes it possible to manage sagas dynamically from our routes. We run our sagas when we enter a route; cancel them when we leave a route.

**You need to create a `sagas.js` file.**

```bash
touch src/store/sagas.js
```

##### `src/store/sagas.js`
We'll dig into this step-by-step below. We create a `sagas.js` file to operate similarly to [`reducers.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/reducers.js). Sagas and reducers do different things but they are both potential targets for actions so it's sensible to configure them similarly.

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
We're treating our sagas similarly to reducers. The [`src/store/reducers.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/reducers.js) file is where you'd import a reducer that all of your app would use. Otherwise you'd import reducers within your route. The starter kit calls them "sync reducers" because they are loaded synchronously as the app loads. "Async reducers" are loaded in a route, asynchronously. We're going to copy that format and load our "sync sagas" in the `src/store/sagas.js` file. You might refer to these types of reducers and sagas as "app sagas" because they're loaded at the app level. As opposed to "route" sagas and reducers which are loaded at the route level.

```js
// ... snippet from src/store/sagas.js

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
This is where we created the middleware we used in `src/store/createStore.js`. We create an `injectSaga()` function so that we don't have to muck with the middleware directly outside of `createStore.js`. In order for a saga to respond to redux actions it needs to be "run" by the saga middleware.

```js
// ... snippet from src/store/sagas.js

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
// ... snippet from src/store/sagas.js

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
We need to dynamically load our saga in our route. This is nearly identical to the [previous example for our todo route](./react-redux-starter-kit-todos.md#route-srcroutestodosindexjs). We're going to be making changes to that file to make it support our sagas. This will be pretty standard on any route you create that needs to implement sagas.

**You need to alter your route to run your sagas.**

##### `src/routes/Todos/index.js`

```js
import { injectReducer } from '../../store/reducers'
import { injectSaga, cancelTask } from '../../store/sagas'

export default (store) => ({
  path: 'todos',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const TodosView = require('./components/TodosView').default
      const { default: reducer, rootSaga: saga } = require('./modules/todos')

      injectReducer(store, { key: 'todosApp', reducer })
      injectSaga({ name: 'todosApp', saga })

      cb(null, TodosView)
    }, 'todos')
  },
  onLeave () {
    cancelTask('todosApp')
  }
})
```

#### How this works
Because we're using Webpack's `require.ensure`, we have to `require` instead of import. Check out [how to rename a variable with destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Assigning_to_new_variable_names). We'll see later how we construct a typical module. It's best to follow the convention of exporting your reducer by default and exporting your modules sagas as `rootSaga`. We'll see in a second what that looks like, but it's basically what we do in the `src/store/sagas.js`.

```js
// ... snippet from src/routes/Todos/index.js

// we use require to, umm, import our module
// we pull out the default reducer
// and we grab the root saga
const { default: reducer, rootSaga: saga } = require('./modules/todos')

// we inject the reducer as normal
injectReducer(store, { key: 'todosApp', reducer })

// we also run our root saga
injectSaga({ name: 'todosApp', saga })

// ...
```

## About Modules
Before we try to implement a saga in our sample app it's important to go over what a generic module looks like. A module is the redux version of a "model" from a traditional MVC app. Of course it's not *exactly* a model but you can see that all of the main pieces are there. Typically a model has getters and setters. In practice you will probably be organizing your module to look similar to something like an [Ember Data Model](http://emberjs.com/api/data/classes/DS.Model.html). 

If you're following my anology, you use a selector in place of a getter. You should use selectors in the `mapStateToProps` function of a container. We use [reselect](https://github.com/reactjs/reselect) for this. It reads a value from the store. Technically you can store data in redux however you want and there are numerous ways to read data back. Regardless of how you structure your modules, using reselect to read from the store is highly recommended. The [documentation provided](https://github.com/reactjs/reselect) is top-notch. More on this below.

If a selector is a getter, then what is a setter? The short answer is "reducers" but that's not the whole story. Typically you don't call a reducer directly, instead you call an action which then gets dispatched to a reducer. You should dispatch actions from the `mapDispatchToProps` function in a container. In redux the simple act of updating an entity is turned into a complex maze of actions and sagas until it eventually reaches a reducer and updates the application state.

When you use something like Ember Data it is very easy to get and set a value on a model. But Ember Data itself does a tremendous amount of work to manage all of the underlying side effects without you needing to worry. In redux there's no magic going on and you have to manage those side effects yourself. That makes it slightly harder to get going but actually results in better performing code and completely removes framework-fighting (bending over backwards to get the framework to do what you want).

1. Selectors are for reading data from the store. We use [reselect](https://github.com/reactjs/reselect).
2. Actions are the first step in writing to the store. We use [redux-actions](https://github.com/acdlite/redux-actions).
3. Sagas are for fielding actions that require async operations. We use [redux-saga](https://github.com/yelouafi/redux-saga).
4. Reducers are for writing to the store. We use [redux-actions](https://github.com/acdlite/redux-actions) for this too.
5. Selectors and Actions form the interface to your module. Anyone using your module in their code will import the selectors for use in `mapStateToProps` and the actions for use in `mapDispatchToProps`.

##### Generic example module
Here's a generic module that makes use of sagas. It's ok to skim this. We'll be rewriting this below for our todo example we've been working on.

```js
// generic module

import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'
import { createSelector } from 'reselect'
import { takeEvery, delay } from 'redux-saga'
import { put } from 'redux-saga/effects'

// selectors
export const getResult = (state) => state.myReducer.result
export const getPending = (state) => state.myReducer.pending

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
  myReducer // <-- receives state.myReducer as state
})
```

## Add sagas to the todos module
If that generic module above is a little confusing that's ok. It's just boilerplate to capture some of the things you typically do in a module. With regards to our todo app we'll just use the todos module recreated in the previous tutorial. We'll be making some changes. We'll start with the finished modules and then explain it piece by piece below.

You should note that right now the async portion of this is totally superficial -- we're simply adding a delay instead of actually syncing to a server. It's important not to get too hung up on the server part of the transaction yet. We'll get deeper into using this with an API in the next tutorial.

##### `src/routes/Todos/modules/todos.js`
Here is a full version of the todos module that utilizes [reselect](https://github.com/reactjs/reselect), [redux-actions](https://github.com/acdlite/redux-actions) and [redux-saga](https://github.com/yelouafi/redux-saga) to recreate the [todos app](http://redux.js.org/docs/basics/index.html). There's a lot going on here and you can start to see why some developers prefer to break their modules into smaller files. We'll go through this in detail below. 

```js
// src/routes/Todos/modules/todos.js

import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'
import { createSelector } from 'reselect'
import { v4 as uuid } from 'node-uuid'
import { takeEvery, delay } from 'redux-saga'
import { put } from 'redux-saga/effects'

// Selectors
export const getAppState = (state) => state.todosApp
export const getVisibilityFilter = (state) => getAppState(state).visibilityFilter
export const getTodos = (state) => getAppState(state).todos
export const getPendingTodos = (state) => getAppState(state).pendingTodos

export const getVisibleTodos = createSelector(
  [ getVisibilityFilter, getTodos ],
  (visibilityFilter, todos) => {
    switch (visibilityFilter) {
      case 'SHOW_ALL':
        return todos
      case 'SHOW_COMPLETED':
        return todos.filter(t => t.completed)
      case 'SHOW_ACTIVE':
        return todos.filter(t => !t.completed)
    }
  }
)

// Constants
export const ADD_TODO_ASYNC = 'ADD_TODO_ASYNC'
const ADD_PENDING_TODO = 'ADD_PENDING_TODO'
const ADD_TODO = 'ADD_TODO'
const REMOVE_PENDING_TODO = 'REMOVE_PENDING_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'
export const TOGGLE_TODO = 'TOGGLE_TODO'

// Action Creators
export const addTodoAsync = createAction(ADD_TODO_ASYNC)
const addPendingTodo = createAction(ADD_PENDING_TODO, text => ({ id: uuid(), text }))
const addTodo = createAction(ADD_TODO, text => ({ id: uuid(), text }))
const removePendingTodo = createAction(REMOVE_PENDING_TODO)
export const setVisibilityFilter = createAction(SET_VISIBILITY_FILTER)
export const toggleTodo = createAction(TOGGLE_TODO)

// Sagas
export function * addTodoAsyncSaga ({ payload }) {
  const pending = yield put(addPendingTodo(payload))
  yield delay(1000)
  yield put(addTodo(payload))
  yield put(removePendingTodo(pending.payload))
}

// Root Saga
export function * rootSaga () {
  yield [
    yield * takeEvery('ADD_TODO_ASYNC', addTodoAsyncSaga)
  ]
}

// Reducers
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

export const todos = handleActions({
  [ADD_TODO]: (state, action) => ([
    ...state,
    todo(undefined, action)
  ]),
  [TOGGLE_TODO]: (state, action) => state.map(t => todo(t, action))
}, [])

export const visibilityFilter = handleActions({
  [SET_VISIBILITY_FILTER]: (state, { payload }) => payload
}, 'SHOW_ALL')

const pendingTodo = handleActions({
  [ADD_PENDING_TODO]: (state, { payload }) => ({
    id: payload.id,
    text: payload.text,
    completed: false,
    pending: true
  })
})

export const pendingTodos = handleActions({
  [ADD_PENDING_TODO]: (state, action) => ([
    ...state,
    pendingTodo(undefined, action)
  ]),
  [REMOVE_PENDING_TODO]: (state, { payload }) => state.filter(t => t.id !== payload.id)
}, [])

// Combined Reducer
export default combineReducers({
  todos,
  visibilityFilter,
  pendingTodos
})
```

## Using Selectors
You can see above that we're using selectors for the first time in this tutorial. At their core, selectors are functions that return a value from a specific part of the state. This helps formalize how your app interacts with the state. For instance, if you're storing the `visibilityFilter` under `state.todosApp.visibilityFilter` you might find it unsettling to paste that into every part of your app that needs to read the current visibility filter. It's easier to provide a simple accessor function.

You can see below that a selector is just a function that returns part of the state. You can easily chain your selectors. It's good practice to provide a generic `getAppState(state)` selector so that you could easily "move" your app in the redux store without having to refactor your entire app.

#### A simple selector in a module
A simple selector is just a function. It shouldn't perform any action. It should simply return a value from the state.

```js
// a simple selector in a module

// we provide an appState selector
// if state.todosApp needed to change we'd only have to update it here
export const getAppState = (state) => state.todosApp

// we use the appState selector to make our app's selectors easier to refactor
// this selector just returns the visibilityFilter from the todo app's state
export const getVisibilityFilter = (state) => getAppState().visibilityFilter

// you are intended to use a selector inside of mapStateToProps in a container component
// mapStateToProps take two arguments: state, containerProps
// standard selector functions take two arguments: state, props
// (it's not common to use the props argument unless you're doing something clever)
export const standardSelector = (state, props) => state.some.key.path
```

#### Using a selector in a container
From a container you use it like this:

```js
// ... snippet from src/routes/Todos/containers/FilterLink.js

// import a selector into a container
import { getVisibilityFilter } from '../modules/todos'

// you use selectors inside here
const mapStateToProps = (state, ownProps) => {
  return {

    // we pass the state to our selector, it returns the value we're looking for
    active: ownProps.filter === getVisibilityFilter(state)
  }
}
```

#### Creating memoized selectors with reselect
In practice most of the selectors you create will just be plain functions. When you need to combine selectors in complex ways, like if you're filtering a list, it's best practice to [memoize your selector](http://stackoverflow.com/questions/32543277/implementing-and-understanding-memoize-function-in-underscore-lodash). "[Memoizing](https://addyosmani.com/blog/faster-javascript-memoization/)" a function means that you cache the results of the function so that you only perform an expensive operation when you need to. This is a standard pattern and selectors are the ideal usecase to apply it. Helpfully, the [reselect](https://github.com/reactjs/reselect) makes it easy to compose selectors and memoize them in a standard way. Similar to using `createAction` from redux-actions, reselect provides a `createSelector` function. You should read the reselect Github page for more information.

Read about [memoization in the good parts](https://www.safaribooksonline.com/library/view/javascript-the-good/9780596517748/ch04s15.html).

```js
// ... selectors snippet from src/routes/Todos/modules/todos.js

import { createSelector } from 'reselect'

// Selectors
export const getAppState = (state) => state.todos
export const getVisibilityFilter = (state) => getAppState(state).visibilityFilter
export const getTodos = (state) => getAppState(state).todos

// this is a memoized selector function
// it calculates an expensive result by combining multiple selectors
export const getVisibleTodos = createSelector(

  // pass in an array of selector functions this memoized selector depends on
  // our memoized selector will only refresh if the results of any selector function changes
  // each selector is passed (state, containerProps)
  [ getVisibilityFilter, getTodos ],

  // the results of each selector are passed as arguments
  // these selector results are used to automanage the cache for the memoize function
  (visibilityFilter, todos) => {

    // perform a potentially  expensive action on a list
    // the return value is automatically cached by the memoize function
    switch (visibilityFilter) {
      case 'SHOW_ALL':
        return todos
      case 'SHOW_COMPLETED':
        return todos.filter(t => t.completed) // <-- Array.prototype.filter is considered "expensive"
      case 'SHOW_ACTIVE':
        return todos.filter(t => !t.completed)
    }
  }
)
```

#### Using a reselect selector in a container
While a reselect selector looks more complicated in our module it's just as easy to use as a standard selector function.

```js
// ... snippet from src/routes/Todos/containers/VisibleTodoList.js

// import a reselect selector into a container
import { getVisibleTodos } from '../modules/todos'

const mapStateToProps = (state) => {
  return {

    // pass the state, get back a memoized result
    // the result stays fresh as the store updates
    todos: getVisibleTodos(state)
  }
}
```

## Using Sagas
We did a fairly thorough job of exploring sagas above. It might seem complicated but in practice sagas are really easy. We're going to review a saga for adding a new todo with a slight delay. Why? Because introducing a delay is enough to show how much a saga can do for us. When you're working with async actions it's good practice to track the progress in the app. We're going to call that 'pending' because any time we add a new todo we will wait 1 second before actually adding it. We're going to be updating our app in a few places to make it obvious how elegant the solution is. You may want to read about [how `yield *` works](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield*).

```js
// ... sagas snippet from src/routes/Todos/modules/todos.js

import { takeEvery, delay } from 'redux-saga'
import { put } from 'redux-saga/effects'

// Sagas

// a saga is just a generator function
// all it does is dispatch actions
// within a saga you need to yield effects
export function * addTodoAsyncSaga ({ payload }) {

  // put returns the action object created by the addPendingTodo action creator
  // put is the same as dispatch, but designed to work with a saga
  // put generates an effect
  const pending = yield put( addPendingTodo(payload) )

  // we use a delay to pretend we're autosaving the todo on the server
  // redux-saga comes with a delay function that pauses the function
  // delay also generates an effect, just like put
  yield delay(1000)

  // after 1 second we dispatch the normal addTodo action
  yield put( addTodo(payload) )

  // to wrap things up we clear our pending todo
  yield put( removePendingTodo(pending.payload) )
}

// Root Saga

// we need to run our sagas
// it's easy to combine them in the array yielded by our rootSaga array
// yielding an array is how you run sagas in parallel
// usually you use takeEvery to map actions to sagas
// typically our route controls when our rootSaga runs
export function * rootSaga () {
  yield [

    // subscribe to start a saga when an action occurs
    yield * takeEvery('ADD_TODO_ASYNC', addTodoAsyncSaga)
  ]
}
```

#### Creating actions for our saga

```js
// ... actions snippet from src/routes/Todos/modules/todos.js

import { createAction, handleActions } from 'redux-actions'
import { v4 as uuid } from 'node-uuid'

// Constants

// we're adding a new async constant
export const ADD_TODO_ASYNC = 'ADD_TODO_ASYNC' 

// we don't need to export these constants if we don't use them outside this file
const ADD_TODO = 'ADD_TODO'
const ADD_PENDING_TODO = 'ADD_PENDING_TODO'   // <-- start
const REMOVE_PENDING_TODO = 'REMOVE_PENDING_TODO' // <-- end

export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'
export const TOGGLE_TODO = 'TOGGLE_TODO'

// Action Creators

// usage: addTodoAsync(text)
// we need to export this action creator
export const addTodoAsync = createAction(ADD_TODO_ASYNC)

// the other action creators are for internal use only and we don't need to export them
// it's good practice to only export what you intend people to use from the outside
// it's pretty easy to add export later if you find that you need to use it elsewhere

// usage: addPendingTodo(text)
// generate a uuid for each pending todo
// we use the id to track the pending todo
const addPendingTodo = createAction(ADD_PENDING_TODO, text => ({ id: uuid(), text }))

// usage: addTodo(text)
// generate a uuid for each todo we add
// we use a different uuid from the pending todo
// presumeably we'd get the ID from the server if we created
const addTodo = createAction(ADD_TODO, text => ({ id: uuid(), text }))

// usage: removePendingTodo({ id })
const removePendingTodo = createAction(REMOVE_PENDING_TODO)

export const setVisibilityFilter = createAction(SET_VISIBILITY_FILTER)
export const toggleTodo = createAction(TOGGLE_TODO)
```

#### Using an async action in a container component
We use actions from the `mapDispatchToProps` portion of a container component. All you really need to do is dispatch an action that we've configured our rootSaga to listen for. If our rootSaga sees our async action it will kick off our saga. This makes it really easy to implement async functionality from a containers perspective. It's good practice to capture complex async logic in your module so that you can control how people interact with your data in a centralized place.

```js
// ... fake snippet from src/routes/Todos/containers/AddTodo.js

// import an async action in a container component
import { addTodoAsync } from '../modules/todos'

// you dispatch actions in here
const mapDispatchToProps = (dispatch) => {
  return {
    addTodo: (text) => {

      // when this gets dispatched our addTodoAsyncSaga will capture it
      dispatch( addTodoAsync(text) )
    }
  }
}
```

#### Creating reducers

```js
// ... reducers snippet from src/routes/Todos/modules/todos.js

import { createAction, handleActions } from 'redux-actions'

// Reducers

// ...

// we add a reducer for creating a pending todo
const pendingTodo = handleActions({
  [ADD_PENDING_TODO]: (state, { payload }) => ({
    id: payload.id,
    text: payload.text,
    completed: false,
    pending: true // <-- pending todos have a pending prop set to true
  })
})

// we need to handle adding and removing pending todos
export const pendingTodos = handleActions({

  // we create a new pending todo and add it to our pending array
  [ADD_PENDING_TODO]: (state, action) => ([
    ...state,
    pendingTodo(undefined, action)
  ]),

  // we remove the pending todo from the pending array by id
  [REMOVE_PENDING_TODO]: (state, { payload }) => state.filter(t => t.id !== payload.id)
}, [])

// Combined Reducer
export default combineReducers({
  todos,
  visibilityFilter,

  // we need to combine our reducer with the others
  // this becomes state.todosApp.pendingTodos
  pendingTodos
})
```

#### Using our pending todos in a container

```js
// ... real snippet from src/routes/Todos/containers/AddTodo.js

import { addTodoAsync } from '../modules/todos'

let AddTodo = ({ dispatch }) => {
  // ... 

  const onSubmit = e => {
    // ...

    // the only change is to dispatch the async action instead of the sync action
    dispatch( addTodoAsync(input.value) )

    // ...
  }

  // ...
}

// ...
```

```js
// ... snippet from src/routes/Todos/containers/VisibleTodoList.js

import { getVisibleTodos, getPendingTodos } from '../modules/todos'

const mapStateToProps = (state) => {

  // get our visible and pending todos
  const visible = getVisibleTodos(state)
  const pending = getPendingTodos(state)

  return {

    // merge the visible and pending todos into a single list
    todos: visible.concat(pending)
  }
}

// ...
```

#### Seeing our pending todos in a component
We need to alter our `TodoList` and our `Todo` components.

```jsx
// src/routes/components/TodoList.js

// ...

TodoList.propTypes = {
  todos: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired,
    pending: PropTypes.bool, // <-- we need to add a pending PropType
    text: PropTypes.string.isRequired
  }).isRequired).isRequired,
  onTodoClick: PropTypes.func.isRequired
}

export default TodoList
```

```jsx
// src/routes/components/Todos.js

import React, { PropTypes } from 'react'

const Todo = ({ onClick, completed, pending, text }) => (
  <li
    onClick={onClick}
    style={{
      textDecoration: completed ? 'line-through' : 'none',
      fontStyle: pending ? 'italic' : 'normal', // <-- change style when pending
      color: pending ? 'gray' : 'inherit'
    }}
  >
    {text}
    {pending ? ' - Waiting' : '' /* we add a label if it's pending */}
  </li>
)

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  completed: PropTypes.bool.isRequired,
  pending: PropTypes.bool, // <-- we need to add a pending PropType
  text: PropTypes.string.isRequired
}

export default Todo
```


