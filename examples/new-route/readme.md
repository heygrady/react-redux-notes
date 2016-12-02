## Folder structure

```md
- routes/MyRoute
  - index.js
  - components/
    - MyRouteView.js
    - SomeComponent.js
  - containers/
    - SomeComponentContainer.js
  - layouts/ <-- Only when child routes exist
    - MyRouteLayout.js
  - modules/
    - myRoute.js
  - routes/childRoute/ <-- Only when child routes exist
```

## Route Index (without child routes)

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

## Route Index (with child routes)

`src/routes/MyRoute/index.js`

```js
import { injectReducer } from '../../store/reducers'

export default (store) => ({
  path : 'myRoute',
  getIndexRoute(location, next) { <-- Necessary when nesting child routes
    require.ensure([], (require) => {
      const MyRouteView = require('./components/MyRouteView').default
      next(null, { component: MyRouteView })
    })
  },
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const reducer = require('./modules/myRoute').default
      const MyRouteLayout = require('./layouts/MyRouteLayout').default

      injectReducer(store, { key: 'myRoute', reducer })

      cb(null, MyRouteLayout)
    }, 'myRoute')
  },
  childRoutes: [
    childRoute(store) <-- Pass down store from parent to the child route
  ]
})
```

## Route Components
`src/routes/MyRoute/components/MyRouteView.js`

```js
import React from 'react'
import SomeComponent from './SomeComponent'

export const MyRouteView = () => (
  <div>
    MyRouteView contents
    <SomeComponent />
  </div>
)

export default MyRouteView
```

`src/routes/MyRoute/components/SomeComponent.js`

```js
import React from 'react'

export const SomeComponent = () => (
  <div>
    SomeComponent contents
  </div>
)

export default SomeComponent
```

## Route Container
`src/routes/MyRoute/containers/SomeComponentContainer.js`

```js
import { connect } from 'react-redux'
import SomeComponent from '../components/SomeComponent'
const mapStateToProps = (state, ownProps) => {
  return {
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
  }
}

const SomeComponentContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(SomeComponent)

export default SomeComponentContainer
```

## Route Layout
`src/routes/MyRoute/layouts/MyRouteLayout.js`

```js
import React from 'react'

const MyRouteLayout = ({ children }) => (<div>{children}</div>)

MyRouteLayout.propTypes = {
  children: React.PropTypes.element.isRequired
}

export default MyRouteLayout
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
