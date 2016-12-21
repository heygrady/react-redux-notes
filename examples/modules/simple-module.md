## Simple module with single reducer
A simple module keeps everything in a single file. It must export a default reducer. Simple modules are useful when your module doesn't do very much. Any reasonably complex module should be exploded into multiple files. However, in cases where your module is very small, it's just fine to keep it in a single file. As your app develops you'll notice that your simple module gets too big and unwieldy&mdash;when it's getting too full it's time to explode it.

```js
// import { combineReducers } from 'redux' // <-- for combining multiple reducers
import { createAction, handleActions } from 'redux-actions'

// make selectors
export const makeSelectors = (selector) => ({
  selectSomeThing: (state, ...args) => selectSomeThing(selector(state), ...args)
})

// selectors
export const selectSomeThing = (state) => state.someThing

// constants
export const ROUTE_NAME_ACTION = 'ROUTE_NAME_ACTION'

// action creators
export const routeNameAction = createAction(ROUTE_NAME_ACTION)

// reducers
export const routeNameReducer = handleActions({
  [ROUTE_NAME_ACTION]: (state, action) => {
    const { payload } = action
    return {
      ...state,
      someThing: !!state.someThing,
      payload
    }
  }
}, { someThing: false })

// export default reducer
export default routeNameReducer

// NOTE: uncomment to combine all of your module's reducers
// export default combineReducers({
//  routeNameReducer
// })
```

## New module with multiple reducers

```js
import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'

// make selectors
export const makeSelectors = (selector) => ({
  selectSomeThing: (state, ...args) => selectSomeThing(selector(state), ...args)
})

// selectors
export const selectSomeThing = (state) => state.someThing

// constants
export const ROUTE_NAME_ACTION = 'ROUTE_NAME_ACTION'
export const ANOTHER_ACTION = 'ANOTHER_ACTION'

// action creators
export const routeNameAction = createAction(ROUTE_NAME_ACTION)
export const anotherAction = createAction(ANOTHER_ACTION)

// reducers
export const routeNameReducer = handleActions({
  [ROUTE_NAME_ACTION]: (state, action) => {
    const { payload } = action
    return {
      ...state,
      someThing: !!state.someThing,
      payload
    }
  }
}, { someThing: false })


export const anotherReducer = handleActions({
  [ANOTHER_ACTION]: (state, action) => {
    const { payload } = action
    return {
      ...state,
      anotherThing: !!state.anotherThing,
      payload
    }
  }
}, { anotherThing: false })

export default combineReducers({
 routeName: routeNameReducer,
 another: anotherReducer
})
```
