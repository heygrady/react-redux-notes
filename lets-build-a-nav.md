# Let's Build a Nav
## A Practical Tutorial

Assuming you've worked through the [React Redux Starter Kit Notes](https://github.com/heygrady/react-redux-notes), this tutorial attempts to answer the question "How do we use all this awesome knowledge to build something on our website?" We will now use our knowledge to build, step-by-step, a main, global navigation area for our website.

### What Do I Need to Do This?
You'll certainly want to be familiar with the [React Redux Starter Kit](https://github.com/davezuko/react-redux-starter-kit) or something similar, and will need a functional instance of WordPress running both the Rest API plugin and the DuxPress API plugin to provide custom endpoints (**only if you are doing Part 3**). If you haven't read [the notes](https://github.com/heygrady/react-redux-notes) about React/Redux and done those tutorials, you might be confused!

### What is this For?
This tutorial will break down a slightly opinionated methodology for front-end developers, especially those new to React/Redux, aimed at building reusable components, containers and modules. The process can be applied to any React/Redux project, but is mainly aimed at building websites. There are three major parts to the methodology:

1. Mock up the UI by creating markup via "dumb" (presentational) components.
2. Hard-code data (props, etc) into "smart" containers which wrap the components.
3. Build modules that handle business logic, API calls, etc. and wire them up to the containers.

If you are working on getting a visual prototype or mock-up of e.g. a web page, you could build the entire thing using parts 1. and 2. and leave 3. for later. In a practical sense, maybe your team works best by having the HTML/CSS gurus work on building basic components and hard-coded containers, and then having your JS sherpas wire things up afterwards.  

Let's do this.

## Part I: Build a Presentational Component
If we were building a traditional site, this might be a good basic structure for your main navigation's markup:

```html
<nav role="navigation">
  <ul>
      <a href="/about/">About</a>
    </li>
    <li>
      <a href="/blog">Blog</a>
    </li>
    <li>
      <a href="/contact">Contact</a>
    </li>
  </ul>
</nav>
```

Creating a presentational component with this markup is very straightforward via the starter kit. From the root of your Starter Kit-based project, in a terminal type `redux g component primaryNav` which creates a boilerplate, stateless functional component:

**src/components/PrimaryNav.js**
```javascript
import React from 'react'
import classes from './PrimaryNav.scss'

export const PrimaryNav = () => (
  <div className={classes['PrimaryNav']}>
    <h1>PrimaryNav</h1>
  </div>
)

export default PrimaryNav
```

Your preferred syntax may vary, but it's simple enough to hard-code the nav markup into this component. In order to do it in a smart fashion, let's structure the component code to accept props, and use React Router to build the links. Your (new) code will look like this:

**src/components/PrimaryNav/PrimaryNav.js**
```javascript
import React, { Component } from 'react'
import { Link } from 'react-router'
import classes from './PrimaryNav.scss'

// TODO: use a stateless functional component
export class PrimaryNav extends Component {

  render () {
    // TODO: this.props should come from a container
    this.props = {
      items: [
        {
          id: 1,
          title: 'About',
          url: '/about'
        },
        {
          id: 2,
          title: 'Contact',
          url: '/contact'
        },
        {
          id: 3,
          title: 'Blog',
          url: '/blog'
        }
      ]
    }

    const { items } = this.props

    return (
      <nav role='navigation'>
        <ul>
        {items.map(item =>
          <li key={item.id}>
            <Link to={item.url} activeClassName={classes.activeRoute}>
              {item.title}
            </Link>
          </li>
        )}
        </ul>
      </nav>
    )
  }
}

export default PrimaryNav

```

Let's go through each part of this code now and talk about how/why it was changed. First, we changed the imports a bit. Before we were simply importing React, now it looks like:

```javascript
import React, { Component } from 'react'
import { Link } from 'react-router'
import classes from './PrimaryNav.scss'
// ...
```

We're now importing React and the `Component` class from React, mainly for convenience. Since we're using React Router's `Link` component, we need to import that. Finally, we're importing CSS classes from a component specific SASS file `PrimaryNav.scss` that we haven't created yet.

Now, we're tweaking the component and render block:

```javascript
// ...
export class PrimaryNav extends Component {

  render () {
    // ...
  }
}
```

This is purely a matter of preference. We imported `Component` so we don't have to type `React.Component`. We also don't have to extend the React.Component class unless we're using certain features of it, but we're doing it here (you could just use a simple functional stateless component if it doesn't need to dispatch actions or have any sort of lifecycle). Note that we're forgetting Prop types at the moment, they will return when we hook this all up with a container.

Now we'll hard-code in props. It's smart to structure your object literal here to mimic the structure of the data you'll be pulling in later. Here, it's an array of menu items:
```javascript
// ...
this.props = {
  items: [
    {
      id: 1,
      title: 'About',
      url: '/about'
    },
    {
      id: 2,
      title: 'Contact',
      url: '/contact'
    },
    {
      id: 3,
      title: 'Blog',
      url: '/blog'
    }
  ]
}
// ...
```

Note that this will go away when we map real data to the props in this component in Part II.

Here's the meat of the component:
```javascript
// ...
const { items } = this.props

return (
  <nav role='navigation'>
    <ul>
    {items.map(item =>
      <li key={item.id}>
        <Link to={item.url} activeClassName={classes.activeRoute}>
          {item.title}
        </Link>
      </li>
    )}
    </ul>
  </nav>
)
// ...
```

First, we extract the `items` array from `this.props`, which we'll be doing to the data we eventually fetch from the API. This allows us to create our final markup for the component. Since our variable `items` is an array of objects, we just use `items.map` to build that array of objects into a series of `li` elements in JSX. Note that we're using the `Link` component from React Router here.

Now just close it out and export the component. Later, we'll be adding PropTypes validation down here:

```javascript
// ...

export default PrimaryNav
```

Now, be sure to create an .scss file with the styles for this component (the Redux CLI may have already created this file for you). You may have a different workflow for handling CSS, in this case we're using css-modules, which ships with the starter kit.

**src/components/PrimaryNav.scss**
```css
.activeRoute {
  font-weight: bold;
  text-decoration: underline;
}
```
Here is the basic style for an "active" link in React Router. You can add any other component styles to this file.

This is the entirety of the basic component, and you can now import it into any view!


## Part II: Wrap the Component in a Container
In the latest version of the starter kit (at the time of this writing) the blueprints for top-level containers has been removed so we can't use the Redux CLI to create one.

**Note!** The AlephSF fork of the starter kit adds a shortcut blueprint to this workflow: `redux g container primaryNav` Using the same name as the component you created above will create a new container with the below boilerplate, appending "Container" to all the names.

Our goal with containers (smart components) is to simply use them to map data from our state and some dispatched actions to the props of our presentational component (via the Connect API). That means we don't want or need to export a component with a render inside it like in a presentational component. Here's how our boilerplate will look:

**src/containers/PrimaryNavContainer/PrimaryNavContainer.js**

```javascript
import { connect } from 'react-redux'
import PrimaryNav from 'components/PrimaryNav'

const mapStateToProps = (state) => {
  return {}
}

const mapDispatchToProps = (dispatch) => {
  return {}
}

const PrimaryNavContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(PrimaryNav)

export default PrimaryNavContainer
```

Some important things to note here. First, we're not importing React at all because there is no JSX to worry about. Second, we are using the magic of the `connect` API from the `react-redux` library to glue together this PrimaryNav container with the PrimaryNav component. Finally, we're returning the results of that connection as `PrimaryNavContainer`, a named constant to be exported wherever we like. This is easier to read than exporting the results of `connect()` directly.

This container doesn't currently do much, as you can see; It is mapping nothing to props from either state or dispatch. Eventually, (in Part III) we will use selectors here to map specific, useful parts of state to props in a shape that is validated by PropTypes in the component.

For now, we can hard-code the state here, and move it out of the component. First, cut it from the component we created in Part I:

**src/components/PrimaryNav/PrimaryNav.js**
```javascript
import React, { Component } from 'react'
import { Link } from 'react-router'
import classes from './PrimaryNav.scss'

export class PrimaryNav extends Component {
  render () {

    // we've cut out the manual setting of props from here!

    const { items } = this.props

    return (
      <nav role='navigation'>
        <ul>
        {items.map(item =>
          <li key={item.id}>
            <Link to={item.url} activeClassName={classes.activeRoute}>
              {item.title}
            </Link>
          </li>
        )}
        </ul>
      </nav>
    )
  }
}

export default PrimaryNav
```

Now, you can add it to state in a hacky-but-effective way:

**src/containers/PrimaryNavContainer/PrimaryNavContainer.js**

```javascript
import { connect } from 'react-redux'
import PrimaryNav from 'components/PrimaryNav'

const mapStateToProps = (state) => {
  return {
    // adding this manually to state just for this component
    items: [
      {
        id: 1,
        title: 'About',
        url: '/about'
      },
      {
        id: 2,
        title: 'Contact',
        url: '/contact'
      },
      {
        id: 3,
        title: 'Blog',
        url: '/blog'
      }
    ]
  }
}

const mapDispatchToProps = (dispatch) => {
  return {}
}

const PrimaryNavContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(PrimaryNav)

export default PrimaryNavContainer
```

At this point, the component and container are connected. The component is blisfully and completely unaware of the source of data in the state, it only cares about rendering props into markup. Let's add in some validation to the component in the form of PropTypes now.

**src/components/PrimaryNav/PrimaryNav.js**
```javascript
import React, { Component, PropTypes } from 'react' // We're importing PropTypes now
import { Link } from 'react-router'
import classes from './PrimaryNav.scss'

export class PrimaryNav extends Component {

  render () {
    const { items } = this.props

    return (
      <nav role='navigation'>
        <ul>
        {items.map(item =>
          <li key={item.id}>
            <Link to={item.url} activeClassName={classes.activeRoute}>
              {item.title}
            </Link>
          </li>
        )}
        </ul>
      </nav>
    )
  }
}

// Now we add in propTypes as a property of the Component class
PrimaryNav.propTypes = {
  items: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number,
    title: PropTypes.string,
    url: PropTypes.string
  }))
}

export default PrimaryNav
```

You can now render the `PrimaryMenuContainer` container anywhere in your app and style it as needed. For example, if you wanted to drop it into the Core Layout file so you could start styling it:

**src/layouts/CoreLayout/CoreLayout.js**
```javascript
import React from 'react'
import classes from './CoreLayout.scss'
import '../../styles/core.scss'
import PrimaryNavContainer from 'containers/PrimaryNavContainer'

export const CoreLayout = ({ children }) => (
  <div className='container text-center'>
    <PrimaryNavContainer />
    <div className={classes.mainContainer}>
      {children}
    </div>
  </div>
)

CoreLayout.propTypes = {
  children: React.PropTypes.element.isRequired
}

export default CoreLayout
```

Now you have a hard-coded component and container without any need to fetch data from an API. When we do need to fetch that data asynchronously, we'll add in some actions and logic to do so and will connect to the API via the module, in Part III.

## Part III: Wire It All Up With a Module
In the `react-redux-notes`, there's [a great little snippet](https://github.com/heygrady/react-redux-notes/blob/master/redux-sagas-todos.md#empty-module) that represents a good boilerplate for a module in our React/Redux applications. We're calling this module `Nav` instad of `PrimaryNav` mainly because it will probably be used to fetch data for other navigation elements as well as the primary menu.

This is important because, for this project, we're going to build a module for every route in the API. This seems like a good idea at the time of writing. Here's the boilerplate nav module after we put in our snippet:

**Remember to add in redux-saga** because we're using it for async stuff: `npm install --save redux-saga`

**src/modules/Nav.js**
```javascript
import { combineReducers } from 'redux'

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

At this point, we'll be making some decisions and also building logic that will tie our API calls together with the container and component that we created in Parts I and II.  Note that we're going to assume that you have a WordPress instance running the Rest API and the DuxPress plugin at `http://localhost:8080`. The DuxPress plugin exposes the endpoints we need at the `menus` route. Let's take a look at some sample JSON returned from an endpoint that gets all menus and locations:

```json
{
  "locations": {
    "header": 59,
    "footer": 61
  },
  "menus": [
    {
      "id": 59,
      "name": "Primary Nav",
      "slug": "primary-nav",
      "items": [
        {
          "url": "/about",
          "title": "About",
          "target": "",
          "description": "",
          "classes": [""]
        },
        {
          "url": "/blog",
          "title": "Blog",
          "target": "",
          "description": "",
          "classes": [""]
        },
        {
          "url": "/contact",
          "title": "Contact",
          "target": "",
          "description": "",
          "classes": [""]
        }
      ]
    },
    {
      "id": 61,
      "name": "Secondary Nav",
      "slug": "secondary-nav",
      "items": [
        {
          "url": "/terms",
          "title": "Terms",
          "target": "",
          "description": "",
          "classes": [""]
        },
        {
          "url": "/press",
          "title": "Press",
          "target": "",
          "description": "",
          "classes": [""]
        }
      ]
    }
  ]
}
```

**DuxPress-Specific Notes:** WordPress allows you to create menus and then to attach menus to locations in the theme. We're basically mirroring that here by outputting a `locations` object which lists all the possible locations for a menu, and then the UID of the menu attached to each location which is matched by the content of the menu itself in the `menus` array. In this case, we have a menu location in the header (wherever that may be), and one in the footer. In the header location, we're attaching a menu with the ID of `59`. Let's consider the header menu location to be the place for our PrimaryNav. The menu with ID 59 is, aptly, named "Primary Nav" in WordPress.

### Create Selectors
The selectors will allow us to easily traverse the state and spit out the information we want. The endpoint we're fetching data from (as above) pulls in all the site menus at once and their locations, so we should create selectors that will get that data for us easily. Selectors are basically helper functions that become a simple shorthand to get stuff from state. The first few selectors should likely point at the specific part of state that you want to grab information for. In this case, we'll be dumping all of this information into an object in state named "menuData", so let's assume that.  

The first selector is going to grab all the menu information (which is going to go into an object called `menuReducer`) into `appState`. This is kind of a standard for us:

```javascript
const getAppState = (state) => state.menuReducer
```

Simple as pie; We pass in the whole state and get back a variable with just the menu stuff. You could `export` this, but we won't because we're not going to use it outside this module currently, maybe ever.

The next two selectors grab our menu locations object and our array of menus using the getAppState selector we just created:

```javascript
const getLocations = (state) => (getAppState(state).menuData.locations || {})  // Create an empty object if nothing comes through here. Important!
const getMenus = (state) => (getAppState(state).menuData.menus || []) // Ditto on empty array.
```

Still no need to export these, probably. We'll maybe never need to spit out all locations or all menus in a container or component (we could easily change this later, though). Within the module, though, we can now use these selectors. Once the API fetch has been run and our JSON (as above) has been injected into state, running `getLocations(state)` will return:

```javascript
{
  "header": 59,
  "footer": 61
}
```

Similarly, `getMenus` will give us our array. This is useful, but how would we really use this in the primaryNav container/component? It would be awesome if I was able to pass in a location's name to a function and have it spit out the data for a menu at this location, right? Assuming we have a location named `header` in WordPress, and that menu #59 is supposed to be there, I'd love to be able to call a function like `getMenuByLocName('header')` and get back:

```json
{
  "id": 59,
  "name": "Primary Nav",
  "slug": "primary-nav",
  "items": [
    {
      "url": "/about",
      "title": "About",
      "target": "",
      "description": "",
      "classes": [""]
    },
    {
      "url": "/blog",
      "title": "Blog",
      "target": "",
      "description": "",
      "classes": [""]
    },
    {
      "url": "/contact",
      "title": "Contact",
      "target": "",
      "description": "",
      "classes": [""]
    }
  ]
}
```

This is _exactly_ what selectors are there for. We wanna keep the little logic stuff in the module here. First, we should make it possible to select a menu from our array of menus via its ID. This is old-school array traversal with a new-school syntax (in the form of `Array.prototype.find` in ES6), and it looks like this:

```javascript
const getMenuById = (state, menuId) => {
  return getMenus(state).find(menu => menu.id === menuId)
}
```

If you're not used to it, this is calling the `.find` method on getMenus (which is an array of all the menus) and returning the first (only) object in the array that matches the menuId that we passed in. Now let's create a simple selector to get the Menu ID by passing in a location name:

```javascript
const getMenuIdByLocName = (state, locName) => {
  return getLocations(state)[locName]
}
```

Let's combine what these things do into a single curried selector. Curried selectors are covered in the sagas notes.

**Note:** If this were an expensive operation (and it might be), we could use `reselect` to build a memoized selector, but we'll keep it all pretty simple here.

Here's the selector that we want to export for use in our container:

```javascript
export const getMenuByLocName = (state) => (locName) => {
  const locs = getLocations(state)
  const menus = getMenus(state)
  const menuId = locs[locName]
  return menuId ? menus.find(menu => menu.id === menuId) : {} // default is empty object
}
```

This is a pretty simple application of selectors, but it demonstrates the major concepts! Once the state is wired up properly, we should be able to use this selector in the container like `getMenuByLocName('header')` and things will be peachy! If the menu object returned was complicated, we could add in more selectors, but it's not, so we probably won't.

**Note:** You have to add in `redux-actions` via `npm install --save redux-actions` before proceeding.

### Create Constants and Action Creators
Let's first change up the imports at the top of our module to include some stuff we need:

```javascript
import { combineReducers } from 'redux' // this was already there
import { createAction, handleActions } from 'redux-actions' // creating and handling actions, below

```

Now we can go about the business of grabbing data from the API and pushing it into our application state so that the selectors can actually use it. We'll be using Sagas for the asynchronous calls, but first let's call out the constants and action creators. This is about as simple as we can make it, we only have one action creators and one constant.

```javascript
// constants
const RECEIVE_MENUS = 'RECEIVE_MENUS'

// action creators
const receiveMenus = createAction(RECEIVE_MENUS)
```

In any application, we'll probably be creating a lot more here, and exporting actions for use elsewhere. An example would be in a module that needs to make actions available to dispatch from i.e. a component that might trigger some async transaction or other logic. At this point, we only need to handle one action, which is going to be to handle and reduce the data we get from the API (via a saga) to get it into state for use by our selectors. We are naming this a "receive" action, and could use this action to do lots of things with the data.

### Create a simple saga
Adding a couple tools for our saga in the module will make imports look like this:

```javascript
import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'
import { put, call } from 'redux-saga/effects' // add this for the saga

```

Our saga is going to do one very specific job, and you may already know that Sagas can be used for much more complicated operations, but here's what it looks like here:

```javascript
// sagas
const jsonUrl = 'http://localhost:8080/wp-json/duxpress-api/v2/menus' // this endpoint should go in a config or env variable one day
export function *fetchMenuData (action) {
  try {
    // simple ES6 fetch
    const response = yield call(fetch, `${jsonUrl}`)

    // convert the json and store it
    const json = yield call(() => response.json())

    // here's our action
    yield put(receiveMenus(json))
  } catch (error) {
    // any problems?
    console.log(error)
  }
}
```

This could typically be accomplished with a single line of code, but it sets a pattern that we can extend to much more complex and powerful applications.

### Wrap it up with Reducers
Everything that you've done to this point has led up to this: reducing our data into a useable state. Your last few lines of code in the module look like this:

```javascript
// reducers
const menuData = handleActions({
  [RECEIVE_MENUS]: (state, { payload }) => {
    // return the new state
    const { locations, menus } = payload
    return { locations, menus }
  }
}, { locations: {}, menus: [] })

// combine reducers
export default combineReducers({
  menuData
})
```

In the first part of this snippet, you're creating a reducer called `menuData` using `handleActions`, and you can see it's basically a switch statement. At the moment, there's only one option, which is to `RECEIVE_MENUS`. This passes in the existing `state`, and then a `payload` and then returns a **new** state (friendly reminder that in Redux we never, ever mutate state, but instead return a new one).

Note that we're constructing both the `locations` object and the `menus` array that we reviewed when we made our selectors here. This is easy because the API returns it in the format we need, but you might have to build in a shim here to structure things properly here. note that we're returning an empty object and array for `locations` and `menus` respectively for when this fires without any data. This is a good practice in general, and can be used to set up the whole default structure of the store here even when there is no data to be had.

Finally, we're combining our reducers here. There's only one, but we could do a bunch!

The final module for this example looks like this:

**src/modules/Nav.js**

```javascript
import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'
import { put, call } from 'redux-saga/effects'

// selectors
const getAppState = (state) => state.menuReducer
const getLocations = (state) => (getAppState(state).menuData.locations || {})
const getMenus = (state) => (getAppState(state).menuData.menus || [])
const getMenuById = (state, menuId) => {
  return getMenus(state).find(menu => menu.id === menuId)
}
const getMenuIdByLocName = (state, locName) => {
  return getLocations(state)[locName]
}

export const getMenuByLocName = (state) => (locName) => {
  const menuId = getMenuIdByLocName(state, locName)
  const menu = getMenuById(menuId)
  return menu
}

// constants
const RECEIVE_MENUS = 'RECEIVE_MENUS'

// action creators
const receiveMenus = createAction(RECEIVE_MENUS)

// sagas
const jsonUrl = 'http://localhost:8080/wp-json/duxpress-api/v2/menus'
export function *fetchMenuData (action) {
  try {
    const response = yield call(fetch, `${jsonUrl}`)
    const json = yield call(() => response.json())
    yield put(receiveMenus(json))
  } catch (error) {
    console.log(error)
  }
}

// reducers
const menuData = handleActions({
  [RECEIVE_MENUS]: (state, { payload }) => {
    const { locations, menus } = payload
    return { locations, menus }
  }
}, { locations: {}, menus: [] })

// combine reducers
export default combineReducers({
  menuData
})
```

### Implement Redux Sagas
To make sagas work in a convenient reusable way, we will take one extra step here. There is more on this in the sagas-specific tutorial, but here are the basic minimal steps.

First, we need to create a `sagas.js` file in the same folder where our `createStore` and `reducers` stuff is, at `src/store`:

```javascript
import createSagaMiddleware from 'redux-saga'

export const sagaMiddleware = createSagaMiddleware()
// We'll use this in our last step below
export const runSaga = (saga) => sagaMiddleware.run(saga)

// Lots of other stuff can go here

export default sagaMiddleware
```

As your app grows in complexity, you will be able to add more and more exports to this file, but this is all we need to get started here. Read the excellent WordPress-specific implementation of posts into the Redux state if you want to delve deeper.

Next, we have to implement the sagaMiddleware in `src/store/createStore.js`, which we can do by adding this import to the top:

```javascript
import sagaMiddleware from './sagas'
```

And then changing the `middleware` const under "Middleware Configuration" to read:

```javascript
const middleware = [thunk, routerMiddleware(history), sagaMiddleware]
```

Finally, we want to add our `menuReducer` in the store, so in `src/store/reducers.js` we import it from the module:

```javascript
import menuReducer from 'modules/Nav' // it's the default export

```

Then add it to the spread of sync reducers like so:

```javascript
export const makeRootReducer = (asyncReducers) => {
  return combineReducers({
    // Add sync reducers here
    router,
    menuReducer, // Hi!
    ...asyncReducers
  })
}
```

Now, if you reload your app and are using the Redux Dev Tools, you should see a nice new part of state, called `menuData`, with the default state you set up; an empty `locations` object and an empty `menus` array. Let's fill them up and finish this thing!

### Wire everything up
If you're feeling sharp today, you may have noticed that our module is currently not being used at all by the application. There are no actions exported for dispatch, no sagas or reducers being run, no nothing. That's what we'll do now, but there are some caveats here. Many modules that will fetch data from external sources will do so when **something at the component level dispatches an action**, or when a route is changed, or something else. This example shows us doing something that is **global** in nature, i.e. a primary navigation element on a website. It'll be triggered once, and probably not from the component level.

Discussions are ongoing about best practices here, but right now, for simplicity, we're doing something pretty basic: Just triggering the `fetchMenuContent` saga once when the application loads using `runSaga`, which we just created above. You could really do that anywhere, but let's put it in the AppContainer because it feels "ok". That means we have to import our module and `runSaga`, and it all looks like this:

**src/containers/AppContainer.js**

```javascript
import React, { Component, PropTypes } from 'react'
import { Router } from 'react-router'
import { Provider } from 'react-redux'
import { runSaga } from '../store/sagas' // Convenience method
import { fetchMenuData } from 'modules/Nav' // Our Module

class AppContainer extends Component {
  static propTypes = {
    history: PropTypes.object.isRequired,
    routes: PropTypes.object.isRequired,
    store: PropTypes.object.isRequired
  }

  render () {
    const { history, routes, store } = this.props
    runSaga(fetchMenuData) // throw this thing anywhere, I guess
    return (
      <Provider store={store}>
        <div style={{ height: '100%' }}>
          <Router history={history} children={routes} />
        </div>
      </Provider>
    )
  }
}

export default AppContainer
```

So here we are, just throwing `runSaga` into the `render` block. This is just fine for now, and if you refresh your app in the browser and are using the Redux Dev Tools, you can view state and will see something awesome: The saga has filled `menuData` with actual menu data. Huzzah!

Now we refactor the container to use our selectors to get this stuff out of state and into props, instead of hard-coding it. That part is pretty easy:

**src/containers/PrimaryNavContainer/PrimaryNavContainer.js**

```javascript
import { connect } from 'react-redux'
import PrimaryNav from 'components/PrimaryNav'
import { getMenuByLocName } from 'modules/Nav' // importing our selector

const mapStateToProps = (state) => {
  // We'll use the selector here and get the menu
  const headerMenu = getMenuByLocName(state, 'primary')
  return {
    items: headerMenu['items'] // we could use another selector to just get 'items'
  }
}
const mapDispatchToProps = (dispatch) => {
  return {}
}

const PrimaryNavContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(PrimaryNav)

export default PrimaryNavContainer
```

This wires up the state to the component, and if you reload your application you'll probably get the red screen of death, your browser screaming bloody murder that it can't map undefined!  Well, that's our last step...

While our presentational components are dumb, they shouldn't be stupid. If we're running any methods that will error out when trying to process values that aren't there (such as undefinedArray.map) inside our component, then we should add in some logic to handle that if it happens, just like in the old days.

In this case, `mapStateToProps` runs anytime the state changes, which means it could run a lot of times (with nothing yet from the saga!) before it has anything to pass into the component. There are a couple approaches here but it makes the most sense to add in the fallbacks at the component level, since that's the point of failure:

**src/components/PrimaryNav/PrimaryNav.js**

```javascript
import React, { Component } from 'react'
import { Link } from 'react-router'
import classes from './PrimaryNav.scss'

export class PrimaryNav extends Component {
  render () {
    // There's no guarantee that items will be defined!
    const { items } = this.props

    //simplest possible route here: If items is falsy, return null
    if (!items) {
      return null  // React won't like it if you return undefined
    } else {
      return (
        <nav role='navigation'>
          <ul>
          {items.map(item =>
            <li key={item.id}>
              <Link to={item.url || ''} activeClassName={classes.activeRoute}>
                {item.title}
              </Link>
            </li>
          )}
          </ul>
        </nav>
      )
    }
  }
}

PrimaryNav.propTypes = {
  items: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number,
    title: PropTypes.string,
    url: PropTypes.string
  }))
}

export default PrimaryNav
```

**React Router Note:** React's `Link` component, used here, has the `to` prop listed as required and it throws all sorts of weirdness if nothing is there, so we're putting in a fallback of an empty string if item.url is falsy.

Reload your app, and you'll see the WordPress menu!

## Conclusions
This is a long way of breaking down an easy thing, but there are some serious pitfalls that you will only counter with experience. Hope you enjoyed this!
