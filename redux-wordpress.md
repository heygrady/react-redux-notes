# Using WordPress with react-redux

We can use the [WP-API WordPress plugin](http://v2.wp-api.org/) to fetch WordPress data over an API. This allows you build a react-redux application as you against the API &mdash; meaning you don't have to use WordPress or PHP as your rendering engine. In our case we intend to use react-redux as our rendering engine. We're only going to be using WordPress as a data store and an admin tool. Of course, this approach means that your frontend site won't necessarily look very much like a WordPress theme. If you really love WordPress themes, then this probably isn't for you. If, however, you prefer to build your website using [state-of-the-art frontend best practices](https://medium.com/javascript-and-opinions/state-of-the-art-javascript-in-2016-ab67fc68eb0b)... keep reading.

Building a fronted app against the API doesn't require using react-redux. You could use use the WP-API with an Ember, Angular or any other frontend framework.

This is written assuming you've familiarized yourself with [the todos app](http://redux.js.org/docs/basics/index.html) from the redux manual, [how it looks in react-redux-starter-kit](./react-redux-starter-kit-todos.md) and [using it with redux-saga](./redux-sagas-todos.md). You might also like to read the note I made as I was [first learning react-redux-starter-kit](https://github.com/heygrady/react-redux-notes).

### WordPress has a demo API

You can use [the demo wp-api](http://demo.wp-api.org/wp-json/wp/v2/) for this tutorial. It's the same as what you'd get if you installed the API on your own WordPress installation. Much of the API can be used without any authentication. For the purposes of this tutorial we'll be ignoring authentication altogether. This may be addressed in a future tutorial.

You may like to read the [WP-API documentation](http://v2.wp-api.org/).

### Quick install react-redux-starter-kit

For this tutorial we are going to rely on a starter kit to set up the initial application. We've been enjoying [react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit) and the redux-cli tool.

Use [redux-cli](https://github.com/SpencerCDixon/redux-cli) to create a new redux app. We also need to install a few useful modules that don't ship with the starter kit by default. We recommend using [redux-actions](https://github.com/acdlite/redux-actions) for actions and reducers, [redux-saga](https://github.com/yelouafi/redux-saga) for asynchronous side effects, and [reselect](https://github.com/reactjs/reselect) for making complex memoized selectors. We use [isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch) for AJAX calls.

We won't be using [node-uuid](https://github.com/broofa/node-uuid) for this example project but it's useful for creating globally unique identifiers. You'll need this in your app if allow users to create new things. You may like reviewing an [example of using uuid in a redux module](./react-redux-starter-kit-todos.md#srcroutestodosmodulestodosjs).

Here's some commands for initializing a redux app and installing some commonly used modules.

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

This is separate from the core use of the WordPress API with react-redux. Later in the tutorial we're going to be building out a few pages with a [master-detail](https://react.rocks/tag/MasterDetail) relationship. Annoyingly, when you scroll down the list and click through to the detail page it stays scrolled to the bottom of the page. There is an issue where [react-router won't scroll to the top on a route change](https://github.com/reactjs/react-router/issues/2019). Thankfully, there's a plugin that manages [react-router scroll](https://github.com/taion/react-router-scroll) behavior. Guess what it's called?

You may want to read up on best practices for [integrating middleware with react-router in react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit/issues/849).

We're going to need to install an additional NPM module and alter a few files that ship with react-redux-starter-kit. Because of [recent changes to `src/main.js`](https://github.com/davezuko/react-redux-starter-kit/commit/cf2f7f2bfcaa1cd9bb6fdccd509a99eb10498789) we need to make changes in two files.

```bash
npm install --save react-router-scroll
```

###### `src/main.js` (compare to the [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/main.js))

```js
// ... router-specific snippets from main.js

import { applyRouterMiddleware, useRouterHistory } from 'react-router' // <-- need to import applyRouterMiddleware
import useScroll from 'react-router-scroll' // <-- import the scrolling middleware

// ...

let render = (routerKey = null) => {
  const routes = require('./routes/index').default(store)

  ReactDOM.render(
    <AppContainer
      {/* ... */}
      render={applyRouterMiddleware(useScroll())} {/* <-- apply the scroll middleware to the router */}
    />,
    MOUNT_NODE
  )
}

// ...
```

###### `src/containers/AppContainer.js` (compare to the [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/containers/AppContainer.js))

```js
// ... snippet from src/containers/AppContainer.js

class AppContainer extends React.Component {
  static propTypes = {
    // ...

    render: PropTypes.func // <-- allow middleware to be injected
  }

  render () {
    // notice `render` is grabbed from `this.props`
    const { history, routes, routerKey, render, store } = this.props

    return (
      <Provider store={store}>
        <div style={{ height: '100%' }}>
          {/* notice the `render={render}` for injecting middleware into the router */}
          <Router history={history} children={routes} key={routerKey} render={render} />
        </div>
      </Provider>
    )
  }
}

export default AppContainer
```

### Install redux-saga

As discussed in our [redux-saga note](./redux-sagas-todos.md) we will need to add a [`src/store/sagas.js`](./redux-sagas-todos.md#srcstoresagasjs) file and use it in [`src/store/createStore.js`](redux-sagas-todos.md#srcstorecreatestorejs).

Please follow those links to configure redux-saga. In the previous tutorial sagas are discussed in great detail.

# What are we building?

We're going to build a simple example with the WordPress API to illustrate how to read a list of posts and display them in a react-redux app. We're going to fetch data using [the posts API](http://v2.wp-api.org/reference/posts/) and show examples of using the API to fetch posts by slug. We're also going to be adding two routes to our app. One route that displays our list of posts and another that shows a post detail page. We'll set them up as nested routes. We'll also go into detail on how to create a module for storing and retrieving data from the redux store.

A future addition to this tutorial might cover things like pagination and preloading the posts in a universal app.

We're going to start with the interface first and then work our way back to our redux module where we'll actually work with the API.

## Adding a route to our starter-kit app

Let's add a route that will show a list of the posts returned by the API.

You can read more about [creating routes in the previous tutorial](./react-redux-starter-kit-todos.md#create-a-new-route).

Here's some commands for creating the necessary new files.

```bash
mkdir -p src/routes/Posts/components
mkdir -p src/routes/Posts/modules
touch src/routes/Posts/components/PostsView.js
touch src/routes/Posts/modules/posts.js
touch src/routes/Posts/index.js
```

### Create Posts route

We need to create a Posts route. Below is the boilerplate for a route. Here we see a typical route that imports [`injectReducer()`](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/store/reducers.js#L12) and [`injectSaga()`](./redux-sagas-todos.md#srcstoresagasjs). We use those functions to inject the root reducer and root sage from the route's module.

It also returns the default view for the route. We'll see later that we need to use a layout instead of a view in order to support the detail page, but we'll start with a view for now.

You may want to read an [explanation of how this route works](./react-redux-starter-kit-todos.md#todos-route).

###### `src/routes/Posts/index.js` (compare to the [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/Counter/index.js) and [saga example](./redux-sagas-todos.md#srcroutestodosindexjs))

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

### Add Posts route to index

We need to add our new Posts route to the index route. For this tutorial we're simply adding our route to the react-redux-starter-kit's index route. We need to import our route and add it to the [`childRoutes`](https://github.com/reactjs/react-router/blob/master/docs/API.md#childroutes) array.

*Note:* By default react-redux-starter-kit uses the [PlainRoute](https://github.com/reactjs/react-router/blob/master/docs/API.md#plainroute) style instead of the JSX style. This is highly recommended because it gives you greater control over [things like code-splitting](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure#code-splitting-anatomy). Most react-router examples use the JSX interface.

If you've been working with react-router for a while you may need to translate some of that knowledge to this different style. In practice it's not that hard but be aware that the PlainRoute interface is sparsely documented. If you have a situation where you need to use JSX-style routes you can [use `createRoute()`](https://github.com/reactjs/react-router/blob/master/docs/API.md#createroutesroutes) ([read "Usage with JSX"](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure#code-splitting-anatomy) in the react-redux-starter-kit wiki for more).

###### `src/routes/index.js` (compare to the [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/index.js))

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

### Add route to navigation

The starter-kit app comes with some example code to help get you started. We'll be leaving that in place and simply adding our code along side what's already there. We need to add a link to our new route in the header navigtion.

###### `src/components/Header/Header.js` (compare to the [starter-kit version](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/components/Header/Header.js))

```js
// ...
    {' · '}
    <Link to='/posts' activeClassName={classes.activeRoute}>
      Posts
    </Link>
// ...
```

## Adding a view

We need to create a Posts view for displaying our list. A view is just a functional component that includes all of the components used in your route. A view could be considred a route's main template. Here we're simply including the `PostListContainer`. Notice that the view doesn't connect to the store because it doesn't need anything from the store, that's handled in the container component.

###### `src/routes/Posts/components/PostsView.js`

```js
import React from 'react'
import PostListContainer from '../containers/PostListContainer'

const PostsView = () => (
  <div>
    <PostListContainer />
  </div>
)

export default PostsView
```

## Starting a module

We need to create a Posts module for managing our posts in the store. We'll fill it in later. If you're comfortable you may wish to add code to the module as you go along to keep your app from throwing errors.

If you were building an app from scratch that's exactly what you'd do. It's common practice to build redux app organically. Meaning, you can sketch out the parts you need and make them do something later.

A [module](./readme.md#modules-control-specific-parts-of-the-store) controls specific parts of the store. A module is made up of selectors, constants, actions creators, sagas and reducers.

- **Selectors** are for reading from the state. A typical selector is a function that takes the state as an argument and returns a part of the state. Like `(state) => state.some.path`.
- **Constants** and **Action Creators** are for dispatching actions.
- **Sagas** are for managing complex asynchronous tasks.
- **Reducers** are for updating the store.

Below you see a very sparsely populated module with all of the main sections left empty.

###### `src/routes/Posts/modules/posts.js` (empty)

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

We need to create a "connected" containter component to display our list because we want to manage the list of posts from the redux store.

Before we fill-in our module we need to create the list component referenced in our view. When you are building an app you should fill in the module for a container as you go. Meaning you don't really know what a module needs to do until you start connecting it to a container. Patterns will emerge as your module is used in more places. It is typically easy to refactor a module even when it's used in more than one container.

Here's some commands for creating the necessary new files.

```bash
mkdir -p src/routes/Posts/components
mkdir -p src/routes/Posts/containers
touch src/routes/Posts/components/Post.js
touch src/routes/Posts/components/PostList.js
touch src/routes/Posts/containers/PostListContainer.js
```

### Post list container

In a redux application you connect to a module from a container. In order to connect our list component to redux we need to also create a container component. We're going to be interfacing with the store from within our container component. Later we'll see that we actually display the list of posts in a regular component that is wrapped by our container. This is a strict separation suggested by redux. Container components allow a developer to map logic from a module (which works with the redux store) to a component (which simply displays data).

As a rule of thumb, if you want to read from the store you need a container component.

In react-redux, the [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) function provides functionality that ensures your component is re-rendered every time the state changes. Keeping components, containers and modules separated allows redux to efficiently keep your app up to date. In large applications this separation also allows developers to work on different parts of the code with minimal friction.

Containers make it easier to refactor a module because it abstracts the logic for dealing with store from the data needed to render the underlying component.

You may like reading about [how container components work](./readme.md#containers-connect-to-the-store).

We'll go through this in detail below.

###### `src/routes/Posts/container/PostListContainer.js`

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

1. We need to import selectors and actions from our module. Selectors like `getAll()` and `isLoadingPosts()` allow you to easily read values from the redux state without knowing very much about how they're stored. That makes it easy to manage complex state logic in the module. A container is solely focused on hooking the module to the component. An action creator like `fetchAll()` can be used to dispatch an action that will update the state. We'll cover `makePostSelectors()` in more detail below. *Selectors* and *action creators* form the *interface* of a module.

  ```js
  import {
    fetchAll, // <-- action creator returns { type: FETCH_ALL_POSTS }
    getAll, // <-- selector returns [ ...posts ]
    isLoadingPosts, // <-- selector returns boolean
    makePostSelectors // <-- returns a set of selectors
  } from '../modules/posts'
  ```

2. `mapStateToProps()` is a function that takes the current redux state and uses it to populate the props of a component. Our component is a list of posts. So we need to map the list of posts from the state to a `posts` prop on the component. You can see that we're calling our selector functions with the redux state as the first argument. There are other ways to read the redux state but it's strongly recommended to only read from the state inside of a `mapStateToProps()` function. This function is used by the `connect()` function and is called every time the state changes.

  ```js
  // this is the correct place to read form the redux store
  const mapStateToProps = (state) => {
    const posts = getAll(state) // <-- select all posts from the state
    return {
      posts, // <-- map the posts to the props of the list component
      makePostSelectors, // <-- pass a function as a prop if you want

      isLoading: isLoadingPosts(state), // <-- read a boolean from the state and rename it

      // create custom values based on the current state
      // here we're checking if there are any posts in the list
      hasPosts: !!posts.length // <-- this is a boolean
    }
  }
  ```

  *Note:* You may want to read about [double exclamation points](http://stackoverflow.com/a/9284677/5149458).

3. `mapDispatchToProps()` is a function that allows you to dispatch actions from a component. Our list component needs to request posts if they haven't been loaded yet. So we're creating a custom `fetchPosts()` function that will be available in the child component as `this.props.fetchPosts()`. We use fetchPosts to dispatch the fetchAll action. The `fetchAll()` action starts saga for fetching posts from the server. When that completes, the list component will reload and display the posts from the store.

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

Typically you want to create a component as a pure function; we'll see an example of that further below. Because our list component needs to request posts on load... we need to create our component using a class instead. Using the class syntax makes the [component lifecycle hooks](https://facebook.github.io/react/docs/component-specs.html#lifecycle-methods) available. Below you can see that we use the [`componentDidMount()`](https://facebook.github.io/react/docs/component-specs.html#mounting-componentdidmount) hook to check for posts after the component has been added to the page. In a universal app you might want to pre-load posts, that's an implementation detail outside the scope of this tutorial.

You may like to read an overview of [best practices regarding components](./readme.md#components-should-be-simple-templates).

*Note:* Starting in React 0.14 it's common practice to define components as [stateless functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components).

*Note:* You should only need to use the [fancier ES6 class syntax](http://facebook.github.io/react/blog/2015/01/27/react-v0.13.0-beta-1.html#es6-classes) (introduced in React 0.13) in special cases, like when you need to use component lifecycle hooks.

###### `src/routes/Posts/components/PostList.js`

```js
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

1. `componentDidMount()` fires when the component is loaded in the page. It also fires every time the component is updated by the container component &mdash; every time the state changes. We need to check if we have posts and check if posts are already loading. If not, we dispatch a request to fetch posts from the server. Once posts have been loaded the container will re-render this component. It's crucial to implement the `isLoading` logic because the state changes several times as the saga deals with fetching the posts. Every time the state changes the component is remounted. If we didn't check if we were in the process of loading we might accidentally send a bunch of requests.

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

2. `render()` should return a template. Here we're checking if the posts are loading or not. If they're loading we show a loading message. If they've already loaded we render a list of posts. You can use the spread operator to map props from an individual post to a `<Post>` component. Something like `<Post {...post}>` makes props available in the component like `this.props.id`.

  We're using the `makePostSelectors(post)` function to create an object that provides selectors that make it convenient to read values from an individual post. We simply spread that object onto the component like we do with the post. Importantly, `{...makePostSelectors(post)}` makes the `get()` selector available as `this.props.get()` in the component. We'll see how this is used in the `<Post>` component below.

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

Finally we need to create a `<Post>` component for displaying an individual item in the list of posts. The purpose of this component is to display all of the attributes of a post in a template. We actually only need the `get()` selector that we made with `makePostSelectors(post)` above. The `get` selector simply returns attributes from a post by key.

You can see in the `<PostList>` above that the attributes of a post are also copied onto the component. You could read the attributes from the components props directly as `attributes[key]`. Simply for the sake of convenience we prefer to access these attributes using a selector function. As you refactor your app it is convenient to be able to update a selector in a central place (the module) instead of needing to update multiple components.

If you wanted to you could replace `get('slug')` below with `props.attributes.slug`. All of this is an implementation detail of your app &mdash; your module's interface is totally up to you and is not dictated by redux or the WordPress API. You may wish to implement something slightly different in your app.

Because the WordPress API returns rendered HTML, we need to use [`dangerouslySetInnerHTML`](https://facebook.github.io/react/tips/dangerously-set-inner-html.html) to display the excerpt of the post in our list. The awkward code is purposeful to ensure that developers think twice before rendering untrusted HTML strings.

###### `src/routes/Posts/components/Post.js`

```js
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

1. `createMarkup()` is the recommended way to ensure that you're writing markup that you trust. There is a potential for [cross-site scripting](https://en.wikipedia.org/wiki/Cross-site_scripting) attacks when rendering arbitrary HTML string from untrusted sources.

2. `formatDate()` doesn't do anything. It is included here to show where you would put logic for formatting a date if your app required it.

3. `get()` is a convenience function for reading an attribute from a post. We'll see how that works in more detail when we create our module for reading values from the redux store. We'll see later how the posts object is constructed in a reducer.

## Fill in our module
We're finally ready to fill in our module. At this point we should have a mostly functioning app that throws errors because our module is empty. If you were more adventurous as you followed along you may already have something that *at least* doesn't throw errors :)

If you've been following along closely above you might have some ideas of the kinds of things that our module needs to do. After reviewing the code in the `<PostListContainer>`, `<PostList>` and `<Post>` we already know the selectors and actions that our posts module needs. We'll start with the `fetchAll`, `getAll`, `isLoadingPosts`, `makePostSelectors` functions and then fill in the sagas and reducers needed to make these functions work.

Below is a complete module for you to copy and past into your app.

We'll go through this in detail further below.

###### `src/routes/Posts/modules/posts.js`

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
  [RECEIVE_ALL_POSTS]: (state, { payload }) => (payload.map((p) => rawPost(p)))
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

The bulk of what your containers will include from your module will be selectors. Selectors are in charge of returning portions of the state that are relevant to your app. Below you'll see all of the selectors we're using in the `PostListContainer`. We'll go through them.

You'll notice that although we recommended reselect for creating memoized selectors... most selectors don't need it. Most of the time a selector directly selects a value from the state. There isn't really anything that can be done to speed that up.

You would use a memoized selector, like the ones reselect's `createSelector()` returns, cache the result of computed selectors. The trick is that the cache is invalidated every time state changes. It's exciting but we don't need it just yet.

*Note:* There's something special to take note of in the way selectors are used in your app. They're meant to be used only in `mapStateToProps`... a function that is called every time the state changes. This reveals the hidden trick to how `createSelector()` works &mdash; it recalculates everything when the state changes without having to subscribe to an event. Instead of relying on events, it happens naturally as because of how its implemented. You can see a version of the same trick being used in the `makePostSelectors()` function below.

*Note:* You can see in `makePostSelectors()` that it relies on [functional scope](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch3.md) to create a sort of ephemeral instance. The object that it returns is only useful for the state it was created for. Because of the way redux connects to react, you can be sure that the object is using the current state as long as you use it within `mapStateToProps`.

*Note:* If you find yourself creating "computed selectors" that combine multiple values into dependent values you should use reselect.

Selectors are meant to be used in the `mapStateToProps` function of a component.

```js
// ... snippet from src/routes/Posts/modules/posts.js

// selectors
export const getAppState = (state) => state.postsApp
export const getAll = (state) => (getAppState(state).posts.data || [])
export const getMeta = (state) => (getAppState(state).posts.meta || {})
export const isLoadingPosts = (state) => !!getMeta(state).loading

export const makePostSelectors = (post) => ({
  get: (key) => post.attributes[key],
  getAttributes: () => post.attributes
})
```

1. `getAppState(state)` is a convenience selector for grabbing the part of the global state that pertains to this app. In a redux app all of the state is stored in the same place. You're free to store things as you please but there are a few conventions that redux encourages.

  Commonly each route has a part of the state that it manages. By convention the reducers for a route are added to the store under a common key. In the Posts route we created above we injected our reducer as `postsApp`. This is enforced in the way that `combineReducers()` functions. The end result is that in a reducer you are always working with a select part of the state.

  In a container component, `state` contains the *whole* state, not just the `postsApp` portion. This is a great convenience because it allows your containers to work with the state in unrestricted ways.

  When you're creating selectors in your module you will need to first select `postsApp` from the state. It's customary to make an app-state selector to make it easier to grab the app-specific portion of the state that your module is concerned with.

  ```js
  // a selector
  // convenient way to grab the state related to the posts app
  export const getAppState = (state) => state.postsApp
  ```

  ```js
  // in a container
  import { getAppState } from '../modules/posts'

  // presume state looks like:
  /*
  {
    postsApp: { ...posts },
    ...otherStuff
  }
  */

  const mapStateToProps = (state) => {
    // use a selector to grab part of the state
    const app = getAppState(state)
    return {
      app // --> { ...posts }
    }
  }
  ```

2. `getAll(state)` returns only the posts. We'll see later when we implement the reducer that we're storing the list of posts as an array under `posts.data`. You can see here that we use the `getAppState(state)` selector to grab our posts app. This makes it easy to refactor the app later and "move the app" to a different part of the store with minimal changes to the selectors that rely on it.

  You can also see that we're returning an empty array by default if posts.data is falsey. This prevents errors from happening in components when `posts.data` is undefined.

  ```js
  // selects the data array, defaults to empty array
  // you may prefer calling this function selectAllPosts()
  export const getAll = (state) => (getAppState(state).posts.data || [])
  ```

3. `isLoadingPosts(state)` returns a boolean if the `posts.meta.loading` prop is set to true. We'll see later that this value is managed by our saga for fetching posts. You can see here that we use the `getMeta(state)` selector to pull out the `posts.meta` object before trying to read the `loading` prop. It's common to chain selectors like this.

  ```js
  export const isLoadingPosts = (state) => !!getMeta(state).loading
  ```

4. `makePostSelectors(post)` returns an object containing selector functions bound to an individual post. This is used in cases where a post object is known and you want to make convenient selectors for working with it. We're showing here just two functions, `get(attributeName)` and `getAttributes()` for working with attributes. You may find that it's a good place to put functions for selecting other types of attributes from posts as well, like `isLoading()` or `isNew()`.

  ```js
  // this function expects a single post, not the entire state
  export const makePostSelectors = (post) => ({

    // this makes it easy to get values from a post
    get: (key) => post.attributes[key],
    getAttributes: () => post.attributes

    // <-- you can add other post-specific selectors here as your app grows
  })
  ```

  One benefit of using the `makeThingSelectors(thing)` pattern is that you can call selectors without having to also be holding the state. The returned selectors could be called "bound selectors" because they are bound to the state that was passed into the "maker" function. It's similar in concept to how [`bindActionCreators()`](http://redux.js.org/docs/api/bindActionCreators.html) works. Instead of binding an action creator to the dispatcher we're binding a selector to the state.

  ```js
  // ... usage of makePostSelectors(post) in a container
  // @see https://repl.it/CZ5e/1

  const mapStateToProps = (state) => {
    const post = getAllPosts(state)[0]

    // "get" is a bound selector
    // the component doesn't need access to the state
    // because it can select the values it needs
    // without it

    // use like get('slug')

    const { get } = makePostSelectors(post)

    return {
      get // available as this.props.get() in the component
    }
  }
  ```

### Adding constants and actions

Constants can make modules seem noisy but they're a good way of keeping track of the "things your module does." Each action needs a constant. We'll see later that these constants are used by sagas and reducers for doing things when certain actions are dispatched.

You may like reading up on [how redux-actions works](https://github.com/acdlite/redux-actions).

Actions can seem confusing but they are easily demystified. We review `createAction()` in step two below. It's important to remember that an "action" is just a plain javascript object. Redux does all the hard work of dispatching your actions.

In order to tightly control when the state is updated, you should only dispatch actions from `mapDispatchToProps` in a container. Once you `dispatch(action)`, redux sends it to your middleware where redux-saga might intercept it. If no middleware catches the action it goes to the reducers. Actions are a convenient way to field events in your application.

The hidden benefit of using dispatch instead of a more familiar evented system is the way state changes trigger container components to re-render. You can dispatch an action in one corner of the app everything will update accordingly without need to explicitly subscribe to a global event. Or, actions are like global events except they can only dispatched from a container and and subscribed to by a saga or a reducer.

So, while constants may seem redundant and noisy, they're a key part of an app's internal communication structure.

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

1. We create a constant for each action. Some of the constants aren't used outside this module so we didn't export them. It's important to know that constants need to be unique across your entire app... even if you don't export them. All actions are handled by redux before being sent to your sagas and reducers. This means that your actions may overlap with actions from another part of your app. This is good news and a major advantage of redux if that's what you want. It's also a common bug that's difficult to track down &mdash; make sure your action constants are globally unique. This is why many classic Flux apps recommended keeping all constants in the same file.

  While constants seem ugly, redundant and annoying, they're a key part of what makes redux applications so predictable and easy to refactor.

2. We use [`createAction()`](https://github.com/acdlite/redux-actions#createactiontype-payloadcreator--identity-metacreator) to create action creators. This makes our actions much more predicable as every action looks like this: `{ type, payload }`. You may like reading more about the [FSA-compliant actions](https://github.com/acdlite/flux-standard-action) that createAction produces.

  There's some hidden trickiness to action creators created using `createAction(type)`. The first argument is a action type, usually a constant. The second argument should be a payload creator but we left it blank. It might seem confusing at first but you don't need to define anything else for an action creator that takes 0 or 1 arguments.

  By default you can just use createAction with a constant. like `createAction(ACTION_TYPE)` and it will create a payload from the argument you pass to it. If you need to accept multiple arguments or construct a specific payload you can provide a payloadCreator function.

  ```js
  // use it like fetchAll()
  // returns an object like { type: 'FETCH_ALL_POSTS', payload }
  export const fetchAll = createAction(FETCH_ALL_POSTS)

  // use it like setMeta('loading', true)
  // returns an object like { type: 'SET_POSTS_META_KEY', payload: { key: 'loading', value: true } }
  const setMeta = createAction(SET_POSTS_META_KEY, (key, value) => ({ key, value }))


  // use it like receivePosts(posts)
  // returns an object like { type: 'RECEIVE_ALL_POSTS', payload: [ ...posts ] }
  const receivePosts = createAction(RECEIVE_ALL_POSTS)
  ```

  You don't have to use createAction if you don't want to. You can read about [creating actions creators](http://redux.js.org/docs/basics/Actions.html#action-creators) in the redux manual. One benefit of using createAction instead of writing the action creator yourself is that it starts to encourage treating every action the exact same way. This is a very good thing once your start building your reducers as it completely removes the need to refactor an action creator in most cases.

  It's extremely beneficial to presume that every action consists of only a type and a payload. Only in special cases, like `setMeta(key, value)`, will you need to create the payload from a list of arguments. The `createActions()` function makes it trivial to create a new action creator that accepts a single argument as the optional payload.

  ```js
  // ... equivalent to what createAction does

  // we don't need a payload for this action
  export const fetchAll = () => ({
    type: FETCH_ALL_POSTS
  })

  // sometimes we need to create a payload
  const setMeta = (key, value) => ({
    type: SET_POSTS_META_KEY,
    payload:{ key, value }
  }))

  // other times we can just pass the payload along
  const receivePosts = (payload) => ({
    type: RECEIVE_ALL_POSTS,
    payload
  }))
  ```

*Note:* At this point your app should be able to load without any errors. You will need to add sagas and reducers to make your app *do* something. But at this point your app is able to render itself.

### Adding sagas

For now we only need one saga for fetching a post from the server. You don't call a saga directly, instead a saga watches for specific actions. Below you can see that we watch a list of actions and return the watchers as the `rootSaga`.

For our purposes it's simple enough to just do the `fetch` call right inside the saga. As your app grows you're likely to follow a common pattern and move all of fetching code into functions outside of your saga. As you add more sagas and implement the API across different modules you'll see the benefit of keeping utility functions for fetching entities in a central place. This type of refactoring is easy to do later as your app matures. Your saga is only concerned with extracting values from promises and dispatching actions. It doesn't really matter to the saga how that promise is created.

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

1. `postsUrl` is how you request a list of posts form the WordPress demo API. In a larger app you might have a utility function that constructs a proper URL for you. You may also wish to keep your URL in a config file to more easily control using different URLs for different environments.

2. `*fetchAllPosts()` is a saga. It accepts an action as its argument (but it doesn't use it for anything). Sagas yield effects. In redux-saga you typically yield a `call`, meaning you want to call a function that returns a promise and resume the saga when the promise is fulfilled.

  You  might also yield a `put`, meaning you want to dispatch an action. You can see that we call `fetch` from isomorphic-fetch and we call `response.json()` to get the JSON from the response. You can also see that we put `setMeta` to toggle the loading state and to capture errors. We also put `recievePosts` with the payload from the AJAX call.

  ```js
  // a saga is a generator function
  function *fetchAllPosts (action) {

    // first we need to toggle "loading" to true
    // this dispatches the setMeta action
    yield put(setMeta('loading', true))

    // you should wrap your fetch call in a try/catch
    try {

      // we get the response back after the fetch promise resolves
      // we add _embed to the URL because we want to embed the author
      const response = yield call(fetch, `${postsUrl}?_embed`)

      // fetch returns a response object
      // the json method also returns a promise
      // we use call to get the json after that promise resolves
      // @see https://developer.mozilla.org/en-US/docs/Web/API/Body/json
      const json = yield call(() => response.json())

      // finally we dispatch a RECEIVE_ALL_POSTS action
      // the posts are processed in a reducer
      yield put(receivePosts(json))
    } catch (error) {

      // keep your errors so that you can do something with them in your app
      yield put(setMeta('error', error))
    }

    // Lastly we need to toggle "loading" to false
    yield put(setMeta('loading', false))
  }
  ```

3. `rootSaga` is an arbitrary name. Typically you will want to export a single saga to make it easier to integrate in your route. The easiest way to map actions to sagas is using the `watchActions` function from `src/store/sagas.js`. It works very similarly to [`handleActions`](https://github.com/acdlite/redux-actions#handleactionsreducermap-defaultstate). You may want to read more about [how watchActions works](./redux-sagas-todos.md#watching-for-actions).

  ```js
  export const rootSaga = watchActions({
    [FETCH_ALL_POSTS]: fetchAllPosts
    // <-- you would add other sagas here
  })
  ```

### Adding reducers

Reducers are used to alter the state. You may want to read more about [how reducers work](http://redux.js.org/docs/basics/Reducers.html). We use [`handleActions`](https://github.com/acdlite/redux-actions#handleactionsreducermap-defaultstate) to make writing reducers much easier.

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
  [RECEIVE_ALL_POSTS]: (state, { payload }) => (payload.map((p) => rawPost(p)))
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

Normally it's easier to read your reducers from the bottom up.

1. [`combineReducers()`](http://redux.js.org/docs/api/combineReducers.html) joins all of your reducers together. Here we've only got a `posts` reducer that we want to inject. If we were to add a reducer for `authors` we'd want to add it here.

  ```js
  export default combineReducers({
    posts // <-- receives state.postsApp.posts as "state"
    // <-- you can put other reducers here
  })
  ```

2. `posts` is the "lowest" reducer -- meaning the lowest down in the file. The lowest reducer must handle all actions that are passed to other reducers in the cascade. It's common practice to have reducers cascade into each other. You can see that the posts reducer branches to the `postsMeta` and `postsData` reducers. Each time you branch to another reducer you want to pass it a portion of the state.

  ```js
  // receives state.postsApp.posts as "state"
  const posts = handleActions({

    // when the setMeta action is dispatched...
    [SET_POSTS_META_KEY]: (state, action) => ({
      ...state, // <-- spread the previous state into a new object
      meta: postsMeta(state.meta, action) // <-- replace meta with the result of postsMeta
    }),

    // when the receivePosts action is dispatched...
    [RECEIVE_ALL_POSTS]: (state, action) => ({
      ...state, // <-- spread the previous state into a new object
      data: postsData(state.data, action) // <-- replace data with the result of postsData
    })
  }, { data: [], meta: {} }) // <-- sets the initial state
  ```

3. `postsMeta` handles actions related to the meta data we store about posts. Here we can see that we handle a generic action that allows for setting the value for a key.

  ```js
  // recieves state.postsApp.posts.meta as "state"
  const postsMeta = handleActions({

    // when the setMeta action is dispatched...
    [SET_POSTS_META_KEY]: (state, { payload }) => ({
      ...state, // <-- spread the previous state into a new object
      [payload.key]: payload.value // <-- set the value for a key
    })
  })
  ```

4. `postsData` handles actions related to the posts themselves. You can read about [designing the state shape](http://redux.js.org/docs/advanced/AsyncActions.html#designing-the-state-shape) in a redux application.

  ```js
  // recieves state.postsApp.posts.data as "state"
  const postsData = handleActions({

    // when the receivePosts action is dispatched...
    // process each post in the payload with the rawPost function
    // we'll explain that function below
    [RECEIVE_ALL_POSTS]: (state, { payload }) => (payload.map((p) => rawPost(p)))
  })
  ```

### Normalizing Data
The [redux manual suggests](http://redux.js.org/docs/advanced/AsyncActions.html#note-on-nested-entities) using [normalizr](https://github.com/paularmstrong/normalizr) to flatten objects retrieved from an API. You can see normalizr in use in the [real-world example](https://github.com/reactjs/redux/blob/master/examples/real-world/middleware/api.js). For now we'll simply pull out the important bits from what's returned by the WordPress API and roughly shape them into a format we like.

We're roughly mimicking the [JSONAPI format](http://jsonapi.org/format/) which is different from the structure that normalizr creates. You're welcome to store data however you please. Using JSONAPI as a design inspiration solves some interesting problems with regard to storing objects retrieved from an API. Namely it demonstrates how to store entities separate from each other and keep their linkages. You may want to check out this gist of [how a JSONAPI inspired structure might look](https://gist.github.com/heygrady/62680ca4a79dcd540369037b83630187).

For this demo we're not following best practice. Normally you'd want to store the authors and other related entities in a separate part of the store. To keep things simple we're storing the author in the post entity itself. This optimizes the store for easily selecting posts. A better practice for designing your state shape is explained in detail in the [normalizr docs](https://github.com/paularmstrong/normalizr#the-problem) and in the [redux manual](http://redux.js.org/docs/advanced/AsyncActions.html#designing-the-state-shape).

You may want to read up on [using the WP-API to retrieve a post](http://v2.wp-api.org/reference/posts/). You you can also look at [the JSON that is returned](https://demo.wp-api.org/wp-json/wp/v2/posts/?_embed). If you want to see a nicely formatted version check out this [gist of what the demo wp-api returns for posts](https://gist.github.com/heygrady/b59c8e1018e33acb7cdbd1e32e5d8a42).

```js
// ... snippet from src/routes/Posts/modules/posts.js

// processing a raw post
// `p` is a "post" from the API
const rawPost = (p) => {

  // pull out interesting values from each post
  const { id, slug, date, type, title, author: authorId, excerpt, content, _links: links, _embedded: embedded } = p

  // author is embedded
  const author = embedded.author.find(a => a.id === authorId)

  // pass back a new post entity
  return {
    id, // <-- every entity needs an ID and a type
    type,
    attributes: { // <-- keep attributes here
      slug,
      date,
      title: title.rendered,
      author: { // <-- we're storing it here for simplicity
        id: author.id,
        name: author.name,
        slug: author.slug,
        avatarUrls: author.avatarUrls,
        meta: { links: author._links }
      },
      content: content.rendered, // <-- the api returns rendered html
      excerpt: excerpt.rendered
    },
    meta: { // <-- keep meta-data about an individual post here
      loading: false,
      saving: false,
      links
    }
  }
}
```

# Add a detail route
By this point our list is rendering just fine but when you can't yet view a detail page. To do this we need to add a child route to the Posts route.

## Post detail route
Here's some commands for creating the necessary new files.

```bash
mkdir -p src/routes/Posts/routes/Post/components
mkdir -p src/routes/Posts/routes/Post/containers
touch src/routes/Posts/routes/Post/components/Post.js
touch src/routes/Posts/routes/Post/components/PostView.js
touch src/routes/Posts/routes/Post/containers/PostViewContainer.js
touch src/routes/Posts/routes/Post/index.js
```

### Post route
The post route is very simple. It doesn't need to inject any additional reducers or sagas. When we need then we'll just add them to the existing posts module. You'll notice that the `path` looks like `path: ':slug'`. The colon before "slug" indicates that it is a param. The full path for this route is inherited by the parent. So, the final URL for a post detail page might look like "http://localhost:3000/posts/**some-unique-slug**". We'll be using the slug returned by WordPress itself. Later we'll see that, sadly, the WordPress API doesn't make it easy to locate a post by its slug.

You may want to read more about [using paths in react-router](https://github.com/reactjs/react-router/blob/master/docs/guides/RouteMatching.md#path-syntax).

###### `src/routes/Posts/routes/Post/index.js`
```js
export default (store) => ({
  path: ':slug', // <-- use a param in the path
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const PostView = require('./containers/PostViewContainer').default

      cb(null, PostView)
    }, 'post')
  }
})
```

### Update posts route to include child route
We need to add our detail route to the parent before we go into creating the view and other components. This is easier said than done. There are a few gotchas when setting up nested routes. The most confusing aspect is [how to display the childRoutes view](https://github.com/davezuko/react-redux-starter-kit/issues/808). We'll see that you have to use a layout when you have childRoutes and that takes [extra configuration](https://github.com/davezuko/react-redux-starter-kit/issues/797). We're going to have to make some big changes to our Posts route.

For reference this configuration is often referred to as a [master-detail route](https://github.com/reactjs/react-router/tree/master/examples/master-detail).

We need to configure our Posts list view as the [`IndexRoute`](https://github.com/reactjs/react-router/blob/master/docs/API.md#indexroute-1) and our Posts detail route as the [`childRoute`](https://github.com/reactjs/react-router/blob/master/docs/API.md#childroutes). We could add additional child routes if we desired. A classic example would be an "add" route for adding new posts. For now we'll simply set up our list and detail routes.

*Note:* Some examples show usage of [`getChildRoutes()`](https://github.com/reactjs/react-router/blob/master/docs/API.md#getchildrouteslocation-callback) to asynchronously load routes. **Don't use `getChildRoutes()` for code-splitting**. The react-redux-starter-kit has a note about [using `childRoutes` to load child routes synchonously](https://github.com/davezuko/react-redux-starter-kit/blob/master/src/routes/index.js#L18). In short, asynchronous code-splitting should happen in [`getComponent()`](https://github.com/reactjs/react-router/blob/master/docs/API.md#getcomponentnextstate-callback).

You can see that we're also using code-splitting to load the IndexRoute using [`getIndexRoute()`](https://github.com/reactjs/react-router/blob/master/docs/API.md#getindexroutelocation-callback). In general, code-splitting is for loading components, reducers and saga within routes. Routes themselves should be loaded synchronously. The reasoning might seem odd at first but your app needs to know about all of its routes to properly render a page, but it doesn't need to have all of the components handy for a route that no one has visited.

###### `src/routes/Posts/index.js` (master-detail)
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

Our app will look something like this:

```
// viewing "/"
CoreLayout
  Header
  HomeView

// viewing "/posts"
CoreLayout
  Header
  PostsLayout
    PostsView

// viewing "/posts/:slug"
CoreLayout
  Header
  PostsLayout
    PostView
```

*Note:* We don't need to make any changes to the PostsView to make it work with the new layout. By returning the list view as the IndexRoute we tell react-router to show this view by default. Otherwise it will show a matching child route. In our case that means `/posts` will render the list view and `/posts/:slug` will render the detail view.

Here's some commands for creating the necessary new files.

```bash
mkdir -p src/routes/Posts/layouts
touch src/routes/Posts/layouts/PostsLayout.js
```

###### `src/routes/Posts/layouts/PostsLayout.js`
```js
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

1. `PostsLayout` is just a regular react component.

2. [`children`](https://facebook.github.io/react/tips/children-props-type.html) is the standard way to nest components in react. This is where the view for an index or child route will be rendered.

## Post Detail View
Now we're ready to start creating the components for our view. Because our route is using a param in the path we need to connect our view to the redux store. We're using [react-router-redux](https://github.com/reactjs/react-router-redux) in this app. The biggest different between using vanilla react-router and using it with react-router-redux is that the routers state is stored in redux. This actually makes it far easier to work with the router from within redux connected components. You may want to read up on [accessing the router's state in a container component](https://github.com/reactjs/react-router-redux#how-do-i-access-router-state-in-a-container-component).

### Post view container
The post detail is unique in that it's view is a connected component. There are other ways to arrange your app. You could read the router's state from any component you like. Feel to read the `:slug` param from within a container component other than the view.

You'll notice that we're requesting additional items from our posts module that we created above. We'll be going into that in more detail below.

###### `src/routes/Posts/routes/Post/containers/PostViewContainer.js`
```js
import { connect } from 'react-redux'
import PostView from '../components/PostView'
import { getPostBySlug, makePostSelectors, fetchPostBySlug } from '../../../modules/posts'

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
    fetchPost: () => dispatch(fetchPostBySlug(slug))
  }
}

const PostViewContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(PostView)

export default PostViewContainer
```

### Post view
Here we can see the post detail view. Similar to the [PostList component](#srcroutespostscomponentspostlistjs) above, this post view needs to be a class component. When we navigate to a detail view from the list the post will have already been loaded from the API. However, if someone navigates directly to a post we'll need to fetch that post by it's slug. This is handled in the container above but it's triggered here.

You'll also notice some logic in the `render` method for showing different things based on the application state. When it comes time to actually display the posts itself we use a `Post` component that we will define further below.

###### `src/routes/Posts/routes/Post/components/PostView.js`
```js
import React, { Component, PropTypes } from 'react'
import { Link } from 'react-router'
import Post from './Post'

class PostView extends Component {
  componentDidMount () {
    const { hasPost, isLoading, fetchPost } = this.props
    if (!hasPost && !isLoading) {
      fetchPost()
    }
  }

  render () {
    const { post, slug, hasPost, isLoading, makePostSelectors } = this.props

    let template

    if (!hasPost && !isLoading) {
      template = (<p>Whoops! No post found for "{slug}."</p>)
    } else if (isLoading) {
      template = (<p>Loading...</p>)
    } else {
      template = <Post key={post.id} {...post} {...makePostSelectors(post)} />
    }

    return (
      <div>
        <Link to='/posts'>&lt; Back to all posts</Link>
        {template}
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

### Post
Here we can see the component for the post itself. We've kep this extremely simple for demonstration purposes. From here you can feel free to add things like comments, links to author detail pages and anything else that you would like to retrieve from the WordPress API.

Below you can see that the template looks very similar to [the `Post` component we created for the list view](#srcroutespostscomponentspostjs). The notable difference is using `content` instead of `except` for the HTML content. If you're feeling crafty you could make a component that could be used in either case. For now we'll stick with two separate but similar components. In a real application the `Post` template below will likely be far more robust than the one for the list view.

###### `src/routes/Posts/routes/Post/components/Post.js`
```js
import React, { PropTypes } from 'react'

const createMarkup = (htmlString) => ({ __html: htmlString })
const formatDate = (dateString) => dateString

const Post = ({ get }) => (
  <div>
    <h1>{get('title')}</h1>
    <p>{formatDate(get('date'))} - {get('author').name}</p>
    <div dangerouslySetInnerHTML={createMarkup(get('content'))} />
  </div>
)

Post.propTypes = {
  get: PropTypes.func.isRequired
}

export default Post
```

### Making it work
At this point you should be able to view a detail page except the app will throw errors if you try to. You can define the necessary functions to get the app working if you'd like.

```js
// add this near the top of src/routes/Posts/modules/posts.js
export const getPostBySlug = () => {}
export const FETCH_POST_BY_SLUG = 'FETCH_POST_BY_SLUG'
export const fetchPostBySlug = createAction(FETCH_POST_BY_SLUG)
```

## Fixing our module to work with posts by slug
Now we're in the home stretch. We need to add several new things to our module to manage individual posts. We'll be going through all of the changes in detail below.

For this demo we're going to resort to using [lodash.memoize](https://www.npmjs.com/package/lodash.memoize). You may wish to read up on [how memoize works](https://lodash.com/docs#memoize). We're using lodash.memoize instead of reselect because reselect doesn't

At the start of this demo we [mentioned using reselect](#quick-install-react-redux-starter-kit) for creating memoized selectors. However, [creating a selector that can take arguments](https://github.com/reactjs/reselect/issues/100) is [really confusing](https://github.com/reactjs/reselect/issues/47). It's probably best to use reselect but currently the docs don't cover use cases where you need to pass additional arguments besides the state. This is a philosophical decision on the part of the reselect maintainers. The intention of reselect is to select and memoize values from the state. If a value isn't in the state, reselect makes life difficult. You might be tempted to simply use the [`defaultMemoize()`](https://github.com/reactjs/reselect#defaultmemoizefunc-equalitycheck--defaultequalitycheck) function from reselect directly, but it is also confusing.

You may notice that we're not adding any additional reducers. The WP-API doesn't offer [a direct way to fetch a post by its slug](https://github.com/WP-API/WP-API/issues/456). Instead, we have to perform a filter search, which returns an array of results. This is actually beneficial for our app because it allows us to use the same reducer for receiving multiple posts as well as receiving a single post by slug.

You may notice that we're not using the demo URL below. The WP-API demo has a [peculiar CORS implementation](https://github.com/WP-API/WP-API/issues/2532) that doesn't work in some cases. For the purposes of this demo, we're using the URL of a demo server that we're hosting ourselves. We can't make any guarantees what that server will return in the future.

Lastly you'll notice that our new saga has a peculiar name: `fetchPostBySlugSaga`. This was done to avoid a naming conflict with the `fetchPostBySlug` action creator. It may be wise to follow the redux manual and name the action creator `requestPostBySlug` and the saga `fetchPostBySlug`. Regardless of your naming scheme, you will inevitably run into issues where your actions creators and sagas have similar names because they represent two parts of the same action.

```bash
npm install --save lodash.memoize
```

###### `src/routes/Posts/modules/posts.js` (by slug)
```js
// ... new functions added to src/routes/Posts/modules/posts.js

import memoize from 'lodash.memoize'

// selectors
export const getPostBySlug = memoize((state, slug) => {
  return getAll(state).find(p => p.attributes.slug === slug)
}, (state, slug) => {
  return JSON.stringify(getAll(state)) + slug
})

// constants
export const FETCH_POST_BY_SLUG = 'FETCH_POST_BY_SLUG'

// actions creators
export const fetchPostBySlug = createAction(FETCH_POST_BY_SLUG)

// sagas
const postBySlugUrl = 'http://54.200.88.22/wp-json/wp/v2/posts/?filter[name]='
function *fetchPostBySlugSaga ({ payload }) {
  yield put(setMeta('loading', true))
  try {
    const response = yield call(fetch, `${postBySlugUrl}${payload}&_embed`)
    const json = yield call(() => response.json())
    yield put(receivePosts(json))
  } catch (error) {
    yield put(setMeta('error', error))
  }
  yield put(setMeta('loading', false))
}

// combine sagas
export const rootSaga = watchActions({
  [FETCH_ALL_POSTS]: fetchAllPosts,
  [FETCH_POST_BY_SLUG]: fetchPostBySlugSaga
})
```
