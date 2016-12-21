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
  - modules/myRoute/index.js <-- should return a reducer by default
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

      cb(null, MyRouteView)
    }, 'myRoute')
  }
})
```
