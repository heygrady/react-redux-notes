# Nested route

```md
- routes/
    Posts/
      components/PostsView.js
      layouts/PostsLayout.js
      modules/posts/index.js
      routes/
        components/PostDetailView.js
        PostDetail/
          index.js <-- child route
      index.js <-- parent route
```

## Parent route
The parent route has one or more child routes. Where there are child routes you need to have a layout and an `indexRoute`.

```js
import { injectReducer } from 'store/reducers'
import PostDetailRoute from './routes/PostDetail'

export default (store) => ({
  path : 'posts',
  getIndexRoute (location, next) {
    require.ensure([], (require) => {
      const PostsView = require('./components/PostsView').default
      next(null, { component: PostsView })
    })
  },
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const reducer = require('./modules/posts').default
      const PostsLayout = require('./layouts/PostsLayout').default

      injectReducer(store, { key: 'posts', reducer })

      cb(null, PostsLayout)
    }, 'posts')
  },
  childRoutes: [
    PostDetailRoute(store)
    // <-- add other children here
  ]
})

```

## Child route
A child route is nested below a parent. It may not need a reducer, it might rely on the parent module instead. If the child route also has nested routes, it would need to follow the pattern above.

```js
// import { injectReducer } from 'store/reducers'

export default (store) => ({
  path : ':postId',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      // const reducer = require('./modules/post').default
      const PostView = require('./components/PostView').default

      // injectReducer(store, { key: 'post', reducer })

      cb(null, PostView)
    }, 'post')
  }
})
```
