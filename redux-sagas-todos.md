# Redux Saga Todos Tutorial
In this tutorial we're going to implement [Redux Saga](http://yelouafi.github.io/redux-saga/) in the same [Todos app from the previous document](./react-redux-starter-kit-todos.md). In fact, we're promarily going to be editing [`modules/todos.js`](./react-redux-starter-kit-todos.md#todos-module-srcroutestodosmodulestodosjs) to replace the reducers with sagas.

It's going to be lightly superficial to remove reducers and use sagas instead. The power of sagas doesn't become aparent until you hook it to an API, which we'll be doing in the nest step. This time around we're going to get sagas integrated into our app so that we can focus on the API portion of it later. If you try to use Redux Saga for the first time it can be intimidating because it jumps right into using the API. We're going to work up to it slowly.

If you want to get into Redux Saga, they have a [great beginner's tutorial](http://yelouafi.github.io/redux-saga/docs/introduction/BeginnerTutorial.html). You may also want to read some of their [saga background links](http://yelouafi.github.io/redux-saga/docs/introduction/SagaBackground.html).

We'll cover it more later but Redux Saga makes extensive use of [generator functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*). You can read [in depth about generators](http://www.2ality.com/2015/03/es6-generators.html) if you'd like but the basics become clear quite quickly.

## Install Redux Saga
Let's install [redux-saga](https://github.com/yelouafi/redux-saga):

```bash
npm install redux-saga --save
```

## Apply Saga Middleware
Redux Saga is middleware. If you use only Redux Saga you should not need [Redux Thunk](https://github.com/gaearon/redux-thunk), the middleware that comes with react-redux-starter kit. When you're working with Redux you need middleware to handle asynchronous actions, these are called side effects. That's why you see alternates to Redux Saga named things like [redux-effects](https://github.com/redux-effects/redux-effects) and [redux-side-effects](https://github.com/gregwebs/redux-side-effect). Redux Thunk works well for side effects but the other effects libraries make it [much easier to chain your effects](http://stackoverflow.com/questions/32925837/how-to-handle-complex-side-effects-in-redux). The biggest advantage of Redux Saga is that it uses native generator functions that are perfectly suited to the types of complex chained actions that you need for making asynchronous requests.

### Create Store: `src/store/createStore.js`


```js
// ...

import createSagaMiddleware from 'redux-saga'

import reducers from './reducers'
import rootSaga from './sagas'

export const sagaMiddleware = createSagaMiddleware()

export default (initialState = {}, history) => {
  let middleware = applyMiddleware(thunk, routerMiddleware(history), sagaMiddleware)

  // ...

  sagaMiddleware.run(rootSaga)

  return store
}
```

https://github.com/yelouafi/redux-saga/issues/76
https://github.com/yelouafi/redux-saga/releases/tag/v0.7.0
https://github.com/yelouafi/redux-saga/releases

