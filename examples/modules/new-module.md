## New module with single reducer

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

// export default reducer
// NOTE: uncomment if you only have one reducer
// export default routeNameReducer


export default combineReducers({
 routeName: routeNameReducer,
 another: anotherReducer
})
```
