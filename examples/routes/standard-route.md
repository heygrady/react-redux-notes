# Standard route
This is a simple route that has a view and a reducer. Not every route needs a reducer.

```js
import { injectReducer } from 'store/reducers'

export default (store) => ({
  path : 'myRoute',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const reducer = require('./modules/myRoute').default
      const MyRouteView = require('./components/MyRouteView').default

      injectReducer(store, { key: 'myRoute', reducer })

      cb(null, MyRouteView)
    }, 'myRoute')
  }
})
```
