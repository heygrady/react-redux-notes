# Using WordPress with react-redux
We can use the [wp-api](http://v2.wp-api.org/) plugin to fetch WordPress data over an API. This allows you build a react-redux application as you normally would and use WordPress primarily as a CMS -- meaning you don't have to use WordPress or PHP as your rendering engine. In our case we intend to use react-redux as our rendering engine. We're only going to be using WordPress as a data store and an admin tool. Of course this approach means that your frontend site won't look very much like a WordPress theme. If you really love WordPress themes, then this probably isn't for you. If, however, you prefer to build your website using state of the art frontend best practices... keep reading.

Of course this approach doesn't require using react-redux. You could do something similar with Ember or Angular or any other frontend app.

This is written assuming you've familiarized yourself with [the basic todos app](http://redux.js.org/docs/basics/index.html), [how it looks in react-redux-starter-kit](./react-redux-starter-kit-todos.md) and [using it with redux-saga](./redux-sagas-todos.md).

### WordPress has a demo API
You can use [the demo wp-api](http://demo.wp-api.org/wp-json/wp/v2/) for this tutorial. It's the same as what you'd get if you installed the API on your own WordPress installation. For the purposes of this tutorial we'll be ignoring the authentication step. This may be addressed in a future tutorial.


### Quick install react-redux-starter-kit
Use [redux-cli](https://github.com/SpencerCDixon/redux-cli) to create a new redux app and install a few useful modules that don't ship with the starter kit. We recommend using [redux-actions](https://github.com/acdlite/redux-actions) for actions and reducers, [redux-saga](https://github.com/yelouafi/redux-saga) for asynchronous side effects, and [reselect](https://github.com/reactjs/reselect) for making complex memoized selectors. We use [isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch) for AJAX calls.

We won't be using [node-uuid](https://github.com/broofa/node-uuid) for this example project but it's useful for creating globally unique identifiers.

```bash
redux create posts-app
cd posts-app
npm install
npm install --save \
  redux-actions \
  redux-saga \
  reselect \
  isomorphic-fetch \
  node-uuid
```

### Install react-router-scroll
This is seperate from the core use of the WordPress API with react-redux. Later in the tutorial we're going to be building out a few pages with a list-detail relationship. Annoyingly, when you scroll down the list and click through to the detail page it doesn't scroll to the top. There is an issue where [react-router won't scroll to the top on route change](https://github.com/reactjs/react-router/issues/2019). Thankfully, there's a plugin that manages [react-router scroll](https://github.com/taion/react-router-scroll) behavior.

```bash
npm install --save react-router-scroll
```

##### `src/containers/AppContainer.js`
(compare to the [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/containers/AppContainer.js))
```jsx
// ... router snippet src/containers/AppContainer.js

// we need to apply the scroll behavior using react router's middleware
import { applyRouterMiddleware, Router } from 'react-router'

// we need to import the middleware for the scroll behavior
import useScroll from 'react-router-scroll'

// ...

// we apply the router middleware on render
<Router history={history} children={routes} key={routerKey} render={applyRouterMiddleware(useScroll())} />

// ...
```

### Install redux-saga
As discussed in our redux-saga note we need to add a [`src/store/sagas.js`](./redux-sagas-todos.md#srcstoresagasjs) file and use it in [`src/store/createStore.js`](redux-sagas-todos.md#srcstorecreatestorejs). Please follow those links to configure redux-saga.


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
We need to create a Posts route. Below is the boilerplate for a route. Here we see a typical route that imports [`injectReducer()`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/reducers.js#L12) and [`injectSaga()`](./redux-sagas-todos.md#srcstoresagasjs) in order to register a module that will be used by connected components (containers). It also returns the default view for the route. We'll see later that we need to use a layout instead of a view in order to support the detail page, but we'll start with a view for now.

##### `src/routes/Posts/index.js`
```js
import { injectReducer } from '../../store/reducers'
import { injectSaga } from '../../store/sagas'

export default (store) => ({ // <-- export the route creator
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

### Add route to navigation
The starter-kit app comes with some example code to help get you started. We'll be leaving that in place and simply adding our code along side what's already there. We need to add a link to our new route in the header navigtion.

##### `src/components/Header/Header.js`
(compare to the [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/components/Header/Header.js))

```jsx
// ...
    {' · '}
    <Link to='/posts' activeClassName={classes.activeRoute}>
      Posts
    </Link>
// ...
```

### Add Posts route to index
We need to add our new Posts route to the index route. For this tutorial we're starting with the index route that ships with react-redux-starter-kit. We need to import our route and add it to the [`childRoutes`](https://github.com/reactjs/react-router/blob/master/docs/API.md#childroutes) array.

*Note:* By default react-redux-starter-kit uses the [PlainRoute](https://github.com/reactjs/react-router/blob/master/docs/API.md#plainroute) style. This is highly recommended because it gives you greater control over [things like code-splitting](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure#code-splitting-anatomy). Most react-router examples use the JSX interface so you will have to translate some of that to this different style. In practice it's not that hard but it's frustrating because the PlainRoute interface is sparsely documented. You can [use `createRoute()` to import JSX routes](https://github.com/reactjs/react-router/blob/master/docs/API.md#createroutesroutes) if you need ([read "Usage with JSX" here](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure#code-splitting-anatomy)).

##### `src/routes/index.js`
(compare to the [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/index.js))

```js
// ...

import PostsRoute from './Posts' // <-- import the route creator

export const createRoutes = (store) => ({
  path: '/',
  component: CoreLayout,
  indexRoute: Home,
  childRoutes: [
    // ...

    PostsRoute(store) // <-- add it to the list
  ]
})
```

## Adding a view
We need to create a Posts view for displaying our list.

##### `src/routes/Posts/components/PostsView.js`
```jsx
import React from 'react'
import PostListContainer from '../containers/PostListContainer'

const PostsView = () => (
  <div>
    <PostListContainer />
  </div>
)

export default PostsView
```

## Adding a module
We need to create a Posts module for managing our posts in the store. We'll fill it in later. Below you see a very sparsely populated module with all of the main sections left empty.

- *Selectors* are for reading from the state. A typical selector is a function that takes the state as an argument and returns a part of the state. Like `(state) => state.some.path`.
- *Constants* and *Action Creators* are for dispatching actions.
- *Sagas* are for managing complex asynchronous tasks.
- *Reducers* are for updating the store.

##### `src/routes/Posts/modules/posts.js`
```js
import { combineReducers } from 'redux'
import { watchActions } from '../../../store/sagas'

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
Before we fill in our module we need to create the list component referenced in our view. We need to create a "connected" component to display our list because we want to manage the list of posts with redux.

Here's some commands for creating the necessary new files.

```bash
mkdir -p src/routes/Posts/components
mkdir -p src/routes/Posts/containers
touch src/routes/Posts/components/Post.js
touch src/routes/Posts/components/PostList.js
touch src/routes/Posts/containers/PostListContainer.js
```
### Post list container
In order to connect our list component to redux we need to create a container. We're going to be interfacing with the store in our container component. Later we'll see that we actually display the list of posts in a regular component that is wrapped by our container. This is a strict separation suggested by redux. Container components allow a developer to map logic from a module (which works with the redux store) to a component (which simply displays data).

In redux, the `connect()` function provides functionality that ensures your component is re-rendered every time the state changes. Keeping components, containers and modules separated allows redux to efficiently keep your app up to date. In large applications this separation also allows developers to work on different parts of the code with minimal friction.

We'll go through this in detail below.

##### `src/routes/Posts/container/PostListContainer.js`
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

const PostListContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(PostList)

export default PostListContainer
```

1. We need to import selectors and actions from our module. Selectors like `getAll()` and `isLoadingPosts()` allow you to easily read value from the redux state without knowing very much about how they're stored. That makes it easy to manage the state logic in the module and solely focus on hooking the module to the component. An action creator like `fetchAll()` can be used to dispatch an action that will update the state. We'll cover `makePostSelectors()` in more detail below.

  ```js
  import { fetchAll, getAll, isLoadingPosts, makePostSelectors } from '../modules/posts'
  ```

2. `mapStateToProps()` is a function that takes the current redux state and uses it to populate the props of a component. Our component is a list of posts. So we need to map the list of posts form the state a `posts` prop on the component. You can see that we're calling our selector functions with the redux state as the first argument. There are other ways to read the redux state but it's strongly recommended to only read from the state inside of a `mapStateToProps()` function.

  ```js
  // this is the correct place to read form the redux store
  const mapStateToProps = (state) => {
    const posts = getAll(state) // <-- get all posts from the state
    return {
      posts, // <-- map the posts to the props of the list component
      makePostSelectors, // <-- pass a function as a prop if you want

      isLoading: isLoadingPosts(state), // <-- read a boolean from the state

      // create custom values based on the current state
      // here we're checking if there are any posts in the list
      hasPosts: !!posts.length // <-- this is a boolean
    }
  }
  ```

  *Note:* You may want to read about [double exclaimation points](http://stackoverflow.com/a/9284677/5149458).

3. `mapDispatchToProps()` is a function that allows you to create functions that dispatch actions and map them to props of a component. Our list component needs to request posts if they haven't been loaded yet. The `fetchAll()` action starts the AJAX request for loading posts from the server. When that completes the list component will reload and display the posts from the store.

  ```js
  // this is the correct place to dispatch actions
  const mapDispatchToProps = (dispatch) => {
    return {
      // this will be available in the list component as this.props.fetchPosts()
      fetchPosts: () => dispatch(fetchAll()) // <-- dispath the fetch all action
    }
  }
  ```

### Post list component
Typicall you want to create components as a pure function, we'll see an example of that below. Because our list component needs to request posts if they aren't already loaded we need to create our component as a class. Using the class syntax makes [component lifecycle hooks](https://facebook.github.io/react/docs/component-specs.html#lifecycle-methods) available. Below you can see that we use the `componentDidMount()` hook to check for posts after the component has been added to the page. In a universal app you might want to pre-load posts, that's an implementation detail outside the scope of this tutorial.

You can read an overview of [best practices regarding components](./readme.md#components-should-be-simple-templates).

*Note:* Starting in React 0.14 it's common practice to define components as [stateless functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components).

*Note:* You should only need to use the [fancier ES6 class syntax](http://facebook.github.io/react/blog/2015/01/27/react-v0.13.0-beta-1.html#es6-classes) (introduced in React 0.13) in special cases, like when you need to use component lifecycle hooks.

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
        : posts.map(post =>
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

1. `componentDidMount()` fires when the component is loaded in the page. We check if we have posts and if posts are already loading. If not, we request that posts be fetched from the server. Once posts have been loaded the container will re-render this component.
  
  ```js
  // this function fires after every time the component is rendered
  componentDidMount () {
    // grab what we need form the component's props
    const { hasPosts, isLoading, fetchPosts } = this.props

    // check if we have posts
    // we don't want to fetch posts if we're currently loading them
    if (!hasPosts && !isLoading) {
      fetchPosts() // <-- this will dispatch an action from the container
    }
  }
  ```

2. `render()` is should return a template. Here we're checking if the posts are loading or not. If they're loading we show a loading message. If they've already loaded we render a list of posts. You can use the spread operator to map props from an individual post to a Post component. We're using the `makePostSelectors(post)` function to create an object that provides selectors that make it convenient to read value from a post. We simply spread those onto the component as well.

  ```js
  // this function fires every time a component is rendered

  render () {
    // grab what we need form the component's props
    const { posts, isLoading, makePostSelectors } = this.props

    // return a template
    return (
      <ul style={{ textAlign: 'left' }}>
        {isLoading /* <-- check if we're currently loading posts */
        ? <li>Loading...</li> {/* <-- show a loading message */}
        : posts.map(post => /* <-- or render a Post for every item in the list */
          <Post
            key={post.id /* <-- key a post by id */}
            {...post /* <-- spread a post's props as component props */}
            {...makePostSelectors(post) /* <-- spread selectors as well */}
          />
        )}
      </ul>
    )
  }
  ```

### Post component
Finally we need to create a component for displaying an individual item in the list. We actually only need the `get()` selector that we made with `makePostSelectors(post)` in the list component above. Because the WordPress API returns rendered HTML, we need to use [`dangerouslySetInnerHTML`](https://facebook.github.io/react/tips/dangerously-set-inner-html.html) to display the except of the post in our list.

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

1. `createMarkup()` is the recommended way to ensure that you're writing markup that you trust. There is a potential for cross-site scripting attacks when rendering arbitrary HTML string from untrusted sources.

2. `formatDate()` doesn't do anything. It is included here to show where you would put logic for formatting a date if your app required it.

3. `get()` is a convenience function for reading an attribute from a post. We'll see how that works in more detail when we create our module for reading values from the redux store.

## Fill in our module
We're finally ready to fill in our module. At this point we should have a mostly functioning app that throws errors because our module is empty.

We'll go through this in detail below.

##### `src/routes/Posts/modules/posts.js`
```js
import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'
import { put, call } from 'redux-saga/effects'
import fetch from 'isomorphic-fetch'
import { watchActions } from '../../../store/sagas'

// selectors
export const getAppState = (state) => state.postsApp
export const getAll = (state) => (getAppState(state).posts.data || [])
export const getMeta = (state) => (getAppState(state).posts.meta || {})
export const isLoadingPosts = (state) => !!getMeta(state).loading

export const makePostSelectors = (post) => ({
  get: (key) => post.attributes[key],
  getAttributes: () => post.attributes
})

// constants
export const FETCH_ALL_POSTS = 'FETCH_ALL_POSTS'
const SET_POSTS_META_KEY = 'SET_POSTS_META_KEY'
const RECEIVE_ALL_POSTS = 'RECEIVE_ALL_POSTS'

// action creators
export const fetchAll = createAction(FETCH_ALL_POSTS)
const setMeta = createAction(SET_POSTS_META_KEY, (key, value) => ({ key, value }))
const receivePosts = createAction(RECEIVE_ALL_POSTS)

// sagas
const postsUrl = 'https://demo.wp-api.org/wp-json/wp/v2/posts'
function *fetchAllPosts (action) {
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

### Adding selectors
```js
// ... snippet from src/routes/Posts/modules/posts.js

// selectors
export const getAppState = (state) => state.postsApp
export const getAll = (state) => (getAppState(state).posts.data || [])
export const getMeta = (state) => (getAppState(state).posts.meta || {})
export const isLoadingPosts = (state) => !!getMeta(state).loading

export const makePostSelectors = (post) => ({
  id: post.id,
  type: post.type,
  meta: post.meta,
  ...post.attributes,
  get: (key) => post.attributes[key],
  getAttributes: () => post.attributes
})
```

### Adding constants and actions
```js
// ... snippet from src/routes/Posts/modules/posts.js

// constants
export const FETCH_ALL_POSTS = 'FETCH_ALL_POSTS'
const SET_POSTS_META_KEY = 'SET_POSTS_META_KEY'
const RECEIVE_ALL_POSTS = 'RECEIVE_ALL_POSTS'

// action creators
export const fetchAll = createAction(FETCH_ALL_POSTS)
const setMeta = createAction(SET_POSTS_META_KEY, (key, value) => ({ key, value }))
const receivePosts = createAction(RECEIVE_ALL_POSTS)
```

### Adding sagas
```js
// ... snippet from src/routes/Posts/modules/posts.js

// sagas
const postsUrl = 'https://demo.wp-api.org/wp-json/wp/v2/posts'
function *fetchAllPosts (action) {
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

### Adding reducers
```js
// ... snippet from src/routes/Posts/modules/posts.js

// reducers
const postsMeta = handleActions({
  [SET_POSTS_META_KEY]: (state, { payload }) => ({
    ...state,
    [payload.key]: payload.value
  })
})

// ...

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

### Normalizing Data
see [normalizr](https://github.com/paularmstrong/normalizr)

```js
// ... snippet from src/routes/Posts/modules/posts.js
// processing a raw post
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

