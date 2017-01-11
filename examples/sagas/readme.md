# Adding sagas to the starter kit
This is a simplified version of getting started with sagas.

## Install
You need to install redux-saga

```
yarn add redux-saga
```

## Add store/sagas
We want to work with sagas similarly to how we work with reducers. We need to add a `src/store/sagas.js` file.

**Create the file `src/store/sagas.js` with the following content.**

```js
import createSagaMiddleware from 'redux-saga'

// sync sagas
import { testSaga } from 'modules/test' // <-- import our test saga

export const sagaMiddleware = createSagaMiddleware()
export const runSaga = saga => sagaMiddleware.run(saga)

// use injectSaga() to add a saga from a route
const tasks = {}
export const injectSaga = ({ key, saga }) => {
  let { task, prevSaga } = tasks[key] || {}

  if (task && prevSaga !== saga) {
    cancelTask(key)
    task = undefined
  }

  if (!task || !task.isRunning()) {
    tasks[key] = {
      task: sagaMiddleware.run(saga),
      prevSaga: saga
    }
  }
}

export const cancelTask = (key) => {
  const { task } = tasks[key]
  if (task) { task.cancel() }
  delete tasks[key]
}

export function * rootSaga () {
  yield [
    testSaga() // <-- you have to invoke the function when you add it
  ]
}

export default sagaMiddleware

```

## Update store/createStore
We need to add our sagas to `src/store/createStore.js`. This is no different than adding any other middleware. Compare to the starter kit version of [`src/store/createStore.js`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/createStore.js).

**Update the file `src/store/createStore.js` with the following content.**

```js
// ...

import saga, { rootSaga } from 'store/sagas' // <-- import store/sagas

// ...

export default (initialState = {}) => {
  // ======================================================
  // Middleware Configuration
  // ======================================================
  const middleware = [thunk, saga] // <-- add saga to the list of middleware

  // ...

  saga.run(rootSaga) // <-- run our saga
  return store
}

```

## Add utils/sagas
We want to add some utils that make working with sagas more like working with `redux-actions`. We're modelling `watchActions` after `handleActions`.

**Create the file `src/utils/sagas.js` with the following content.**

```js
import { takeEvery } from 'redux-saga/effects'

export const createWatcher = (actionType, saga) => {
  return function * () {
    yield takeEvery(actionType, saga)
  }
}

export const watchActions = (sagas) => {
  const watchers = Object.keys(sagas)
    .map(actionType => createWatcher(actionType, sagas[actionType])())
  return function * rootSaga () {
    yield watchers
  }
}

```
## Usage

### Usage of watchActions
You would use `watchActions` similalrly to `handleActions`.

**Create the file `src/modules/test.js` with the following content.**

```js
import { createAction } from 'redux-actions'
import { watchActions } from 'utils/sagas'

// for this example we need to have an action
export const SOME_ACTION = 'SOME_ACTION'
export const someAction = createAction(SOME_ACTION)

// our saga is a generator function
export const takeSomeAction = function * (action) {
  console.log('worked!')
}

// we can map actions to sagas very easily
export const testSaga = watchActions({
  [SOME_ACTION]: takeSomeAction
})

// NOTE: in a container, dispatch(someAction()) --> worked!

```

### Usage of injectSaga
Typically you would use `injectSaga` in a route. Below you can see a route that lazy-loads the reducer and the saga. In the following example we're presuming that the `modules/myRoute` exports its reducer by default and exports a `rootSaga` generator function. Compare this to a [standard route](../routes/standard-route.md).

**Create the file `src/routes/MyRoute/index.js` with the following content.**

```js
import { injectReducer } from 'store/reducers'
import { injectSaga } from 'store/sagas' // <--

export default (store) => ({
  path : 'myRoute',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const { default: reducer, rootSaga: saga } = require('./modules/myRoute') // <--
      const MyRouteView = require('./components/MyRouteView').default

      injectReducer(store, { key: 'myRoute', reducer })
      injectSaga({ key: 'myRoute', saga }) // <--

      cb(null, MyRouteView)
    }, 'myRoute')
  }
})

```
