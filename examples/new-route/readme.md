## Folder structure

```md
- routes/MyRoute
  - index.js
  - components/
    - MyRouteLayout.js <-- Only when child routes exist
    - MyRouteView.js
    - SomeComponent.js
  - containers/
    - SomeComponentContainer.js
  - modules/myRoute.js
```

## Route Index

`src/routes/MyRoute/index.js`

```js
import { injectReducer } from '../../store/reducers'

export default (store) => ({
  path : 'myRoute',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const reducer = require('./modules/myRoute').default
      const MyRouteView = require('./components/MyRouteView').default

      injectReducer(store, { key: 'myRoute', reducer })

      cb(null, MyRouteLayout)
    }, 'myRoute')
  }
})
```

## Route Module

`src/routes/MyRoute/modules/myRoute.js`

```js
// import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'

// make selectors
export const makeSelectors = (selector) => ({
  selectSomeThing: (state, ...args) => selectSomeThing(selector(state), ...args)
})

// selectors
export const selectSomeThing = (state) => state.someThing

// constants
export const MY_ROUTE_ACTION = 'MY_ROUTE_ACTION'

// action creators
export const myRouteAction = createAction(MY_ROUTE_ACTION)

// reducers
export const myRouteReducer = handleActions({
  [MY_ROUTE_ACTION]: (state, action) => {
    const { payload } = action
    return {
      ...state,
      someThing: !!state.someThing,
      payload
    }
  }
}, { someThing: false })

// export default reducer
export default myRouteReducer

// NOTE: uncomment to combine all of your module's reducers
// export default combineReducers({
//  myRouteReducer
// })

```
