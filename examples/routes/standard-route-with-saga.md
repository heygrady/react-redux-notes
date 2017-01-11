# Standard route with saga
This is a simple route that has a `view`, a `reducer` and a `rootSaga`. Not every route needs a reducer or a rootSaga. In the following example we're presuming that the `modules/myRoute` exports its reducer by default and exports a `rootSaga` generator function.

```js
import { injectReducer } from 'store/reducers'
import { injectSaga } from 'store/sagas'

export default (store) => ({
  path : 'myRoute',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const { default: reducer, rootSaga: saga } = require('./modules/myRoute')
      const MyRouteView = require('./components/MyRouteView').default

      injectReducer(store, { key: 'myRoute', reducer })
      injectSaga({ key: 'myRoute', saga })

      cb(null, MyRouteView)
    }, 'myRoute')
  }
})
```
