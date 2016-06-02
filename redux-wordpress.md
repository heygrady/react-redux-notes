# Using Wordpress with the react-redux
We can use the [wp-api](http://v2.wp-api.org/) plugin to fetch Wordpress data over an API. This allows you build a react-redux application as you normally would and use Wordpress primarily as a CMS. Of course this approach means that your frontend site won't look very much like a Wordpress theme. If you really love Wordpress themes, then this probably isn't for you. If, however, you prefer to build your website using state of the art frontend best practices keep reading.

This is written assuming you've familiarized yourself with [the basic todos app](http://redux.js.org/docs/basics/index.html), [how it looks in react-redux-starter-kit](./react-redux-starter-kit-todos.md) and [using it with redux-saga](./redux-sagas-todos.md).

### Wordpress has a demo API
You can use [the demo wp-api](http://demo.wp-api.org/wp-json/wp/v2/) for this tutorial. It's the same as what you'd get if you installed the API on your own Wordpress installation. For the purposes of this tutorial we'll be ignoring the authentication step. This may be addressed in a future tutorial.

### Install react-router-scroll

```bash
npm install --save react-router-scroll
```

##### `src/main.js`
```jsx
// ... router snippet from main.js

import createBrowserHistory from 'history/lib/createBrowserHistory'
import { applyRouterMiddleware, Router, useRouterHistory } from 'react-router'
import { syncHistoryWithStore } from 'react-router-redux'
import useScroll from 'react-router-scroll'

// ...

let render = (key = null) => {
  // ...
        <Router history={history} children={routes} key={key} render={applyRouterMiddleware(useScroll())} />
  // ...
}
```

## Adding a route to our starter-kit app
Let's add a route that will show all of the posts returned by the API. You can read more about [creating routes in the previous tutorial](./react-redux-starter-kit-todos.md#create-a-new-route).

Here's some commands for creating the necessary new files

```bash
mkdir -p src/routes/Posts/components
mkdir -p src/routes/Posts/modules
touch src/routes/Posts/components/PostsView.js
touch src/routes/Posts/modules/posts.js
touch src/routes/Posts/index.js
```

We need to create a Posts route:

##### `src/routes/Posts/index.js`
```js
import { injectReducer } from '../../store/reducers'
import { injectSaga, cancelTask } from '../../store/sagas'

export default (store) => ({
  path: 'posts',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const PostsView = require('./components/PostsView').default
      const { default: reducer, rootSaga: saga } = require('./modules/posts')

      injectReducer(store, { key: 'postsApp', reducer })
      injectSaga({ name: 'postsApp', saga })

      cb(null, PostsView)
    }, 'posts')
  }
})
```

We need to add our new Posts route to the index route.

##### `src/routes/index.js`
```js
// ...
import PostsRoute from './Posts'

export const createRoutes = (store) => ({
  path: '/',
  component: CoreLayout,
  indexRoute: Home,
  childRoutes: [
    // ...

    PostsRoute(store)
  ]
})
```

## Adding Components to the Posts route

We need to create a Posts view for displaying our list:

##### `src/routes/Posts/components/PostsView.js`
```jsx
import React from 'react'
import ConnectedPostList from '../containers/ConnectedPostList'

const PostsView = () => (
  <div>
    <ConnectedPostList />
  </div>
)

export default PostsView
```

We need to create a Posts module for managing our posts in the store. We'll fill it in later.

##### `src/routes/Posts/modules/posts.js`
```js
import { combineReducers } from 'redux'
import { watchActions } from '../../store/sagas'

// selectors

// constants

// action creators

// sagas

// combine sagas
export const rootSaga = watchActions({
  // combine all of your module's sagas
})

// reducers

// combine reducers
export default combineReducers({
  // combine all of your module's reducers
})
```

## Adding a list component to our route
Here's some commands for creating the necessary new files

```bash
mkdir -p src/routes/Posts/components
mkdir -p src/routes/Posts/containers
touch src/routes/Posts/components/PostList.js
touch src/routes/Posts/containers/ConnectedPostList.js
```

##### `src/routes/Posts/container/ConnectedPostList.js`
```js
import { connect } from 'react-redux'
import { fetchAll, getAll, isLoadingPosts, makePostSelectors } from '../modules/posts'
import PostList from '../components/PostList'

const mapStateToProps = (state) => {
  const posts = getAll(state)
  return {
    posts,
    makePostSelectors,
    isLoading: isLoadingPosts(state),
    hasPosts: !!posts.length
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    fetchPosts: () => dispatch(fetchAll())
  }
}

const ConnectedPostList = connect(
  mapStateToProps,
  mapDispatchToProps
)(PostList)

export default ConnectedPostList
```

##### `src/routes/Posts/components/PostList.js`
```jsx
import React, { Component, PropTypes } from 'react'
import Post from './Post'

class PostList extends Component {
  componentDidMount () {
    const { hasPosts, isLoading, fetchPosts } = this.props
    if (!hasPosts && !isLoading) {
      fetchPosts()
    }
  }

  render () {
    const { posts, isLoading, makePostSelectors } = this.props
    return (
      <ul style={{ textAlign: 'left' }}>
        {isLoading
        ? <li>Loading...</li>
        : null}
        {posts.map(post =>
          <Post
            key={post.id}
            {...post}
            {...makePostSelectors(post)}
          />
        )}
      </ul>
    )
  }
}

PostList.propTypes = {
  posts: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number.isRequired
  }).isRequired).isRequired,
  fetchPosts: PropTypes.func.isRequired,
  makePostSelectors: PropTypes.func.isRequired,
  isLoading: PropTypes.bool.isRequired,
  hasPosts: PropTypes.bool.isRequired
}

export default PostList
```

##### `src/routes/Posts/components/Post.js`
```jsx
import React, { PropTypes } from 'react'
import { Link } from 'react-router'

const createMarkup = (htmlString) => ({ __html: htmlString })
const formatDate = (dateString) => dateString

const Post = ({ get }) => (
  <li>
    <h2>
      <Link to={`/posts/${get('slug')}`}>
        {get('title')}
      </Link>
    </h2>
    <p>{formatDate(get('date'))} - {get('author').name}</p>
    <div dangerouslySetInnerHTML={createMarkup(get('excerpt'))} />
  </li>
)

Post.propTypes = {
  id: PropTypes.number.isRequired,
  get: PropTypes.func.isRequired
}

export default Post
```

## Adding selectors, constants and actions to our module

##### `src/routes/Posts/modules/posts.js`
```js
// ... interface snippet from src/routes/Posts/modules/posts.js
import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'
import { createWatcher } from '../../store/sagas'

// selectors
export const getAppState = (state) => state.postsApp
export const getAll = (state) => (getAppState(state).data || [])
export const getMeta = (state) => (getAppState(state).meta || {})
export const isLoadingPosts = (state) => getMeta(state).loading

export const makePostSelectors = (post) => ({
  get: (key) => post.attributes[key],
  getAttributes: () => post.attributes
})

// constants
export const FETCH_ALL_POSTS = 'FETCH_ALL_POSTS'

// action creators
export const fetchAll = createAction(FETCH_ALL_POSTS)
```

## Adding Sagas to our module

##### `src/routes/Posts/modules/posts.js`
```js
// ... sagas snippet from src/routes/Posts/modules/posts.js
import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'
import { put, call } from 'redux-saga/effects'
import fetch from 'isomorphic-fetch'
import { watchActions } from '../../store/sagas'

// constants (for sagas)
const SET_FETCH_META_KEY = 'SET_FETCH_META_KEY'
const RECEIVE_ALL_POSTS = 'RECEIVE_ALL_POSTS'

// actions (for sagas)
const setMeta = createAction(SET_FETCH_META_KEY, (key, value) => ({ key, value }))
const receivePosts = createAction(RECEIVE_ALL_POSTS)

// sagas
const postsUrl = 'https://demo.wp-api.org/wp-json/wp/v2/posts'
function * fetchAllPosts (action) {
  yield put(setMeta('loading', true))
  try {
    const response = yield call(fetch, `${postsUrl}?_embed`)
    const json = yield call(() => response.json())
    yield put(receivePosts(json))
  } catch (error) {
    yield put(setMeta('error', error))
  }
  yield put(setMeta('loading', false))
}

// combine sagas
export const rootSaga = watchActions({
  [FETCH_ALL_POSTS]: fetchAllPosts
})
```

##### `../../store/sagas`
```js

export const createWatcher = (actionType, saga) => {
  return function * () {
    yield * takeEvery(actionType, saga)
  }
}

export const watchActions = (sagas) => {
  const watchers = Object.keys(sagas)
    .map((type) => createWatcher(type, sagas[type]))

  return function * rootSaga () {
    yield watchers
  }
}
```

## Adding Reducers to our module
##### `src/routes/Posts/modules/posts.js`
```js
// ... reducers snippet from src/routes/Posts/modules/posts.js

// reducers
const postsMeta = handleActions({
  [SET_POSTS_META_KEY]: (state, { payload }) => ({
    ...state,
    [payload.key]: payload.value
  })
})

const rawPost = (p) => {
  const { id, slug, date, type, title, author: authorId, excerpt, content, _links: links, _embedded: embedded } = p
  const author = embedded.author.find(a => a.id === authorId)

  return {
    id,
    type,
    attributes: {
      slug,
      date,
      title: title.rendered,
      author: {
        id: author.id,
        name: author.name,
        slug: author.slug,
        avatarUrls: author.avatarUrls,
        meta: { links: author._links }
      },
      content: content.rendered,
      excerpt: excerpt.rendered
    },
    meta: {
      loading: false,
      saving: false,
      links
    }
  }
}

const postsData = handleActions({
  [RECEIVE_ALL_POSTS]: (state, action) => (action.payload.map((p) => rawPost(p)))
})

const posts = handleActions({
  [SET_POSTS_META_KEY]: (state, action) => ({
    ...state,
    meta: postsMeta(state.meta, action)
  }),
  [RECEIVE_ALL_POSTS]: (state, action) => ({
    ...state,
    data: postsData(state.data, action)
  })
}, { data: [], meta: {} })

// combine reducers
export default combineReducers({
  posts
})
```

## Add a post detail route

```bash
mkdir -p src/routes/Posts/routes/Post/components
mkdir -p src/routes/Posts/routes/Post/containers
touch src/routes/Posts/routes/Post/components/PostView.js
touch src/routes/Posts/routes/Post/containers/ConnectedPostView.js
touch src/routes/Posts/routes/Post/index.js
```

##### `src/routes/Posts/Post/index.js`
```js
export default (store) => ({
  path: ':slug',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const PostView = require('./containers/ConnectedPostView').default

      cb(null, PostView)
    }, 'post')
  }
})
```

##### `src/routes/Posts/routes/Post/containers/ConnectedPostView.js`
```js
import { connect } from 'react-redux'
import PostView from '../components/PostView'
import { getPostBySlug, makePostSelectors, fetchPost } from '../../../modules/posts'

const mapStateToProps = (state, ownProps) => {
  const slug = ownProps.params.slug
  const post = getPostBySlug(state, slug)
  return {
    slug,
    makePostSelectors,
    post,
    hasPost: !!post,
    isLoading: post && post.meta ? !!post.meta.loading : false
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  const slug = ownProps.params.slug
  return {
    fetchPost: () => dispatch(fetchPost(slug))
  }
}
const ConnectedPostView = connect(
  mapStateToProps,
  mapDispatchToProps
)(PostView)

export default ConnectedPostView
```

##### `src/routes/Posts/routes/Post/components/PostView.js`
```jsx
import React, { Component, PropTypes } from 'react'
import Post from './Post'

class PostView extends Component {
  componentDidMount () {
    const { hasPost, isLoading, fetchPost } = this.props
    if (!hasPost && !isLoading) {
      fetchPost()
    }
  }

  render () {
    const { post, isLoading, makePostSelectors } = this.props
    return (
      <div>
        {isLoading || !post
        ? <p>Loading...</p>
        : <Post key={post.id} {...post} {...makePostSelectors(post)} />}
      </div>
    )
  }
}

PostView.propTypes = {
  post: PropTypes.shape({
    id: PropTypes.number.isRequired
  }),
  makePostSelectors: PropTypes.func.isRequired,
  hasPost: PropTypes.bool.isRequired,
  isLoading: PropTypes.bool.isRequired,
  fetchPost: PropTypes.func.isRequired,
  slug: PropTypes.string.isRequired
}

export default PostView
```

### Update posts route to include child route

```bash
mkdir -p src/routes/Posts/layouts
touch src/routes/Posts/layouts/PostsLayout.js
```

##### `src/routes/Posts/index.js`
```js
import { injectReducer } from '../../store/reducers'
import { injectSaga } from '../../store/sagas'
import PostRoute from './routes/Post' // <-- import the route for a single post

export default (store) => ({
  path: 'posts',
  getIndexRoute (location, next) { // <-- lazy load the component for a list of posts
    require.ensure([], (require) => {
      const PostsView = require('./components/PostsView').default
      next(null, { component: PostsView })
    })
  },
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const PostsLayout = require('./layouts/PostsLayout').default // <-- use a layout
      const { default: reducer, rootSaga: saga } = require('./modules/posts')

      injectReducer(store, { key: 'postsApp', reducer })
      injectSaga({ name: 'postsApp', saga })

      cb(null, PostsLayout)
    }, 'posts')
  },
  childRoutes: [
    PostRoute(store) // <-- initialize the route for a single post
  ]
})
```

##### `src/routes/Posts/layouts/PostsLayout.js`
```jsx
import React from 'react'

const PostsLayout = ({ children }) => (
  <div>{children}</div>
)

PostsLayout.propTypes = {
  children: React.PropTypes.element.isRequired
}

export default PostsLayout
```

