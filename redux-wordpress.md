# Using WordPress with the react-redux
We can use the [wp-api](http://v2.wp-api.org/) plugin to fetch WordPress data over an API. This allows you build a react-redux application as you normally would and use WordPress primarily as a CMS -- meaning you don't have to use WordPress or PHP as your rendering engine. In our case we intend to use react-redux as our rendering engine. We're only going to be using WordPress as a data store and an admin tool. Of course this approach means that your frontend site won't look very much like a WordPress theme. If you really love WordPress themes, then this probably isn't for you. If, however, you prefer to build your website using state of the art frontend best practices... keep reading.

Of course this approach doesn't require using react-redux. You could do something similar with Ember or Angular or any other frontend app.

This is written assuming you've familiarized yourself with [the basic todos app](http://redux.js.org/docs/basics/index.html), [how it looks in react-redux-starter-kit](./react-redux-starter-kit-todos.md) and [using it with redux-saga](./redux-sagas-todos.md).

### WordPress has a demo API
You can use [the demo wp-api](http://demo.wp-api.org/wp-json/wp/v2/) for this tutorial. It's the same as what you'd get if you installed the API on your own WordPress installation. For the purposes of this tutorial we'll be ignoring the authentication step. This may be addressed in a future tutorial.

### Install react-router-scroll
This is seperate from the core use of the WordPress API with react-redux. Later in the tutorial we're going to be building out a few pages with a list-detail relationship. Annoyingly, when you scroll down the list and click through to the detail page it doesn't scroll to the top. There is an issue where [react-router won't scroll to the top on route change](https://github.com/reactjs/react-router/issues/2019). Thankfully, there's a plugin that manages [react-router scroll](https://github.com/taion/react-router-scroll) behavior.

```bash
npm install --save react-router-scroll
```

##### `src/main.js`
```jsx
// ... router snippet from main.js

// the react-router-scroll manual hints at using a different browser history
// however, we'll use the browser history that ships with react-redux-starter-kit
import createBrowserHistory from 'history/lib/createBrowserHistory'

// we need to apply the scroll behavior using react router's middleware
import { applyRouterMiddleware, Router, useRouterHistory } from 'react-router'

// import the middleware for the scroll behavior
import useScroll from 'react-router-scroll'

// ...
        // we apply the router middleware on render
        <Router history={history} children={routes} key={key} render={applyRouterMiddleware(useScroll())} />
// ...
```

# What are we building?
We're going to build the simplest example with the WordPress API to illustrate how to read data and display it in a react-redux app. We're going to read [the posts API](http://v2.wp-api.org/reference/posts/) and show examples of using the API to fetch posts by ID and by slug. We're also going to be adding two routes to our app. A route that lists out posts and a route that shows a post detail page. We'll set them up as nested routes. Finally we'll use the store to persist our requests.

A future addition to this tutorial might cover things like pagination and preloading the posts in a universal app.

## Adding a route to our starter-kit app
Let's add a route that will show all of the posts returned by the API. You can read more about [creating routes in the previous tutorial](./react-redux-starter-kit-todos.md#create-a-new-route).

Here's some commands for creating the necessary new files.

```bash
mkdir -p src/routes/Posts/components
mkdir -p src/routes/Posts/modules
touch src/routes/Posts/components/PostsView.js
touch src/routes/Posts/modules/posts.js
touch src/routes/Posts/index.js
```

### Create Posts route
We need to create a Posts route. Below is the boilerplate for a route, here we see a typical route that imports [`injectReducer`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/reducers.js#L12) and [`injectSaga`](./redux-sagas-todos.md#srcstoresagasjs) in order to register a module that will be used by connected components (containers). It also returns the default view. We'll see later that we need to use a layout instead of a view in order to support the detail page, but we'll ignore that for now.

##### `src/routes/Posts/index.js`
```js
import { injectReducer } from '../../store/reducers'
import { injectSaga } from '../../store/sagas'

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

### Add Posts route to index
We need to add our new Posts route to the index route. For this tutorial We're starting with the index route that ships with react-redux-starter-kit. We need to import our new Posts route and add it to the [`childRoutes`](https://github.com/reactjs/react-router/blob/master/docs/API.md#childroutes) array.

*Note:* By default react-redux-starter-kit uses the [PlainRoute](https://github.com/reactjs/react-router/blob/master/docs/API.md#plainroute) style. This is highly recommneded because it gives you greater control over [things like code-splitting](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure#code-splitting-anatomy). Most react-router examples use the JSX interface so you will have to translate some of that to this different style. In practice it's not that hard but it's frustrating because the PlainRoute interface is sparsely documented. You can [use `createRoute()` to import JSX routes](https://github.com/reactjs/react-router/blob/master/docs/API.md#createroutesroutes) if you need ([read "Usage with JSX" here](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure#code-splitting-anatomy)).

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
    .map((type) => createWatcher(type, sagas[type])())

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

## Update posts route to include child route
This is easier said than done. There are a few gotchas when setting up nested routes. The most confusing aspect is [how to display the childRoutes view](https://github.com/davezuko/react-redux-starter-kit/issues/808). We'll see that you have to use a layout when you have childRoutes and that takes [extra configuration](https://github.com/davezuko/react-redux-starter-kit/issues/797). We're going to have to make some big changes to our Posts route.

We need to configure our Posts list view as the [`IndexRoute`](https://github.com/reactjs/react-router/blob/master/docs/API.md#indexroute-1) and our Posts detail route as the [`childRoute`](https://github.com/reactjs/react-router/blob/master/docs/API.md#childroutes). We could add additional child routes if we desired. A classic example would be an "add" route for adding new posts. For now we'll simply set up our list and detail routes.

*Note:* Some examples show usage of [`getChildRoutes()`](https://github.com/reactjs/react-router/blob/master/docs/API.md#getchildrouteslocation-callback) to asynchronously load routes. **Don't use `getChildRoutes()` for code-splitting**. The react-redux-starter-kit has a note about [using `childRoutes` to load child routes synchonously](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/index.js#L18). In short, asynchronous code-splitting should happen in [`getComponent()`](https://github.com/reactjs/react-router/blob/master/docs/API.md#getcomponentnextstate-callback).

You can see that we're also using code-splitting to load the IndexRoute using [`getIndexRoute()`](https://github.com/reactjs/react-router/blob/master/docs/API.md#getindexroutelocation-callback).

##### `src/routes/Posts/index.js`
```js
import { injectReducer } from '../../store/reducers'
import { injectSaga } from '../../store/sagas'
import PostRoute from './routes/Post' // <-- import the route for a single post

export default (store) => ({
  path: 'posts',
  getIndexRoute (location, next) { // <-- lazy load the post list view component
    require.ensure([], (require) => {
      const PostsView = require('./components/PostsView').default
      next(null, { component: PostsView }) // <-- the view becomes the IndexRoute
    })
  },
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const PostsLayout = require('./layouts/PostsLayout').default // <-- import a layout
      const { default: reducer, rootSaga: saga } = require('./modules/posts')

      injectReducer(store, { key: 'postsApp', reducer })
      injectSaga({ name: 'postsApp', saga })

      cb(null, PostsLayout) // <-- update the route component to be a layout
    }, 'posts')
  },
  childRoutes: [
    PostRoute(store) // <-- initialize the detail route
  ]
})
```

### Create a posts layout
Because we're nesting routes we need to use a layout. The [react-redux-starter-kit uses a layout](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/index.js#L11) to display the home page and the counter app. A layout component simply wraps child routes. You can use a layout to include common components that are visible on all child routes. The [react-redux-starter-kit's core layout includes the global navigation](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/layouts/CoreLayout/CoreLayout.js#L8). Our posts layout will simply render the routes inside of an empty div. To counteract the default styles that ship with rect-redux-starter-kit we'll change the text alignment to make things easier to read.

*Note:* We don't need to make any changes to the PostsView to make it work with the new layout. By returning the list view as the IndexRoute we tell react-router to show this view by default. Otherwise it will show a matching child route. In our case that means `/posts` will render the list view and `/posts/:slug` will render the detail view.

```bash
mkdir -p src/routes/Posts/layouts
touch src/routes/Posts/layouts/PostsLayout.js
```

##### `src/routes/Posts/layouts/PostsLayout.js`
```jsx
import React from 'react'

const PostsLayout = ({ children }) => (
  <div style={{textAlign: 'left'}}>
    {children /* <-- index/child route components end up here */}
  </div>
)

PostsLayout.propTypes = {
  children: React.PropTypes.element.isRequired
}

export default PostsLayout
```

