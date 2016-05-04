# React-Redux Notes
Here are my notes from getting to know React Redux a lot better.

### How did we get here?
I've been a web developer for more than a decade. I've seen JavaScript take a massive leap forward several times over the years. First with [Prototype.js](http://prototypejs.org/) then [jQuery](https://jquery.com/) and more recently with [Angular](https://angularjs.org/) and [Ember](http://emberjs.com/). JavaScript has been maturing slowly for what seems like forever. But these days everyone is excited about the possibilities of using ES6 and the premier way to build apps in 2016 is [React-Redux](http://redux.js.org/docs/basics/UsageWithReact.html).

Getting into the Redux ecosystem can be daunting as there are many new tools to learn, even if you've been trying to keep up. It's reminiscent of learning [SCSS](http://sass-lang.com/) for the first time where suddenly there's a build step where there wasn't one before. The days of writing HTML, CSS and JavaScript without a build tool are long gone. Now in order to build even a simple brochure site you should be using a build tool or you're doing it wrong. You can get a feel for how complex it's starting to get by reading the "[State of the Art JavaScript in 2016](https://medium.com/javascript-and-opinions/state-of-the-art-javascript-in-2016-ab67fc68eb0b#.6wdwcsm93)" posted in February. It basically lays out all of the tools a modern web developer should be using and, not surprisingly, those are all the exact same tools you'll see in use in a React-Redux application.

Anyone who got started on Angular back in the early 1.x days can attest that the biggest hurdle was... getting it started. The "best" way to work with Angular was to use build tools but Angular didn't ship with a CLI or a starter kit. So getting started was really hard! Eventually there was a good [Yeoman generator for Angular](https://github.com/yeoman/generator-angular#readme) available. This made it dead simple to understand all of the core concepts of Angular and start applying them without having to get lost in the underlying tooling.

In the early Angular days building JavaScript bundles was still an insane mess. How messy? Read this [epic history of why requireJS is bad](https://gist.github.com/david-mark/2845842) from the man who accidentally created the core idea behind AMD (also see the [official requireJS history](http://requirejs.org/docs/history.html) by the creator of RequireJS). In Angular you were still bending over backwards to overcome the weirdest part of working with JavaScript: everything ran in the global scope! One could argue that the vast majority of Angular 1.x was written to overcome that one messy hurdle. It was barely a step up from the [IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/) that were the best practice in the jQuery days.

Ember takes all of that a huge step further into the future with their impressive [ember-cli](http://ember-cli.com/) tool. Used with [Ember Data](https://guides.emberjs.com/v2.5.0/models/) and [JSONAPI](http://jsonapi.org/) (especially with a [Rails 5 API](http://emberigniter.com/modern-bridge-ember-and-rails-5-with-json-api/)) Ember is a powerful and productive framework. Ember shares a number of core concepts and tooling with React-Redux, including a focus on components and using Babel to bring ES6 modules to the development life cycle. The biggest problem with Ember is that it makes it [enormously difficult to use NPM packages](http://stackoverflow.com/questions/26544578/how-to-use-third-party-npm-packages-with-ember-cli-app). They have their reasons for this decision, but it's not a restriction that exists in a typical React-Redux App.

### ES6 is a game changer
ES6 has arrived. Now we're [supposed to call it ES2015](http://meta.stackoverflow.com/questions/297774/rename-es6-to-es2015) and we're to expect an [ES2016](http://www.2ality.com/2016/01/ecmascript-2016.html) this year, aka ES7. [Babel](https://babeljs.io/) is helping the JavaScript community avoid the huge [debacle that was Python 3](http://www.donationcoder.com/forum/index.php?topic=37718.0) (seriously, read the links at the top of [that thread](http://www.donationcoder.com/forum/index.php?topic=37718.0) and realize just how well-executed the ES6 roll-out has been). Essentially Python 3 was very different from Python 2 and that made it nearly impossible to adopt because old code wouldn't work on the new engine. Many people in the Python community chose not to upgrade. You could find similar trepidation within the Ruby community when they were upgrading from 1.8 to 1.9 and 2.x. That problem eventually lead to the wide adoption of [RVM](http://code.tutsplus.com/articles/why-you-should-use-rvm--net-19529). Upgrading a language can be excruciatingly painful for a community of developers. We all remember the pains of trying to support IE6, IE7, IE8, IE9...

The differences between ES5 and ESNext are huge and most browsers [don't support ES6 yet](https://kangax.github.io/compat-table/es6/) (although [Chrome](http://blog.chromium.org/2016/04/es6-es7-in-browser.html) and [Node](https://nodejs.org/en/blog/release/v6.0.0/) are [very close](http://node.green/) now). In browser-land, the [complicated matrix of (un)supported features](http://caniuse.com/) makes the problems of Python and Ruby seem laughably simple. How does it work? Like SCSS before it, Babel completely side-steps the problem: it converts your shiny new ES6 code to plain-old JavaScript first. Babel isn't just smoothing over language compatibility issues for browsers. You can even use [babel to write npm modules](http://jamesknelson.com/writing-npm-packages-with-es6-using-the-babel-6-cli/).

With the change over to ES2015 and beyond, JavaScript development is making a dramatic shift towards modernization. A React-Redux application takes full advantage of that new power. Often then best way to do something in Redux is to simply write ES6-styled JavaScript in a smart way. Redux makes older tools like Angular and Ember seem obsolete because ES6 makes JavaScript much easier to write. In many cases the conventions-based-approach of React and Redux completely replace the need for more complicated frameworks. Of course the downside is that Redux development is *mostly* about conventions, not frameworks. There is no massive framework that holds your hand through the whole process. Instead, Redux gets out of your way and forces you to get to know a new way to think about writing apps. Thankfully there is a consensus emerging on the right way to get started.

If you didn't realize it already, [reactive programming is nothing new](http://blog.salsitasoft.com/why-now/), but it is fairly new to JavaScript. The biggest hurdle is rearranging your thinking away from OOP frameworks like Ember and embracing functional programming instead. Bear in mind it's possible that [React-Redux is getting it all wrong](http://staltz.com/why-react-redux-is-an-inferior-paradigm.html).

# React Redux Starter Kit
If you're new to Redux and want to get started there's really no better place than the [react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit) in conjunction with the [redux-cli](https://github.com/SpencerCDixon/redux-cli) tool. If you've been digging into Redux and noticing that it's missing a framework and a CLI, then you just found it! Although it's deceptively named "starter kit", it's the missing app framework and CLI for React-Redux projects. The biggest advantage of the starter kit is that it wires up the [router](https://github.com/reactjs/react-router-redux) and bootstraps your application better than you would all by yourself. From there it's easy to start putting your app together. Having the hard work of getting Webpack and Babel working up with all of the best practices is a *massive* time saver. You're not likely to need to mess with the defaults until much later in your dev process. This allows you to get to the hard work of building your app without getting lost in the tooling.

Because the Redux ecosystem is heavily inspired by the Node ecosystem (a React-Redux app uses NPM to manage packages) there isn't ever going to be a full framework like there is for Ember. Instead, building a React-Redux app is an exercise in assembling the right tools for your application. Sadly, beyond the foundations in the starter-kit you're going to have to research to find each of those little tools and stitch them together yourself. Much of what you need is easy to find on NPM. In practice this isn't very difficult. It actually becomes more and more of a blessing as your app matures. However for starting out it can be daunting.

The [Redux Todo tutorial](http://redux.js.org/docs/basics/index.html) currently stops at "use middleware to solve async needs." There are dozens of middleware solutions that try to solve the problem of asynchronous actions in a variety of different ways. The classic [redux-thunk](https://github.com/gaearon/redux-thunk) middleware is just the simplistic beginning. Once you start building an app you're going to want to move away from the boilerplate redux you learned in the Todo tutorial and you're going to want to pick the middleware that will make building your app as painless as possible.

### Middleware blues
The biggest missing piece from the starter kit is a strong opinion about middleware. Right now it seems like the community is coalescing around [redux-sagas](http://yelouafi.github.io/redux-saga/index.html). Each project is free to make its own choice in this regard and Redux seems to have left this gap wide open on purpose. You'll likely feel extremely opinionated yourself once you get started on your *second* app.

When we get there you'll want to choose between [redux-saga](https://github.com/yelouafi/redux-saga), [redux-effects](https://github.com/redux-effects/redux-effects), [redux-side-effects](https://github.com/gregwebs/redux-side-effect), and [redux-loop](https://github.com/raisemarketplace/redux-loop).

## Getting Started with the Starter Kit
What you may not realize is that you *must* watch the videos before doing anything with React-Redux. You have to watch the videos before you even read the manual. **If you haven't watched the videos you shouldn't be reading this right now!**

1. **[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux)** - By the creator of Redux, Dan Abramov. These videos are more essential than you may realize. It's nearly impossible to understand Redux without watching them.
2. **[Official Redux documentation](http://redux.js.org/)** - Also by the creator of Redux. Written in a similar style to the videos and covers all of the same code. After watching the videos you will want to try to build the example Todo app. You can do this by reading the code in the manual.
3. **[Starting out with react-redux-starter-kit](https://suspicious.website/2016/04/29/starting-out-with-react-redux-starter-kit/)** - Blog post about how to use the react-reduc-starter-kit. It covers all of the basics of the project layout. If you skim the [write-up on fractal project structures](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure) you'll see many of the same concepts.
4. **Quit thinking about universal/isomorphic) and native apps for now.** You're getting ahead of yourself. Once you get the hang of Redux the other stuff is easy and already solved for you.

## High-level Concepts
It's important to get a picture of how a typical starter-kit app is structured because it makes it clear how to apply the core principles of React-Redux. At it's core React-Redux is a marriage of components (React) with a store (Redux). Most people get a little lost at this point so here's a quick example using a plain React component.

There's a good [write-up about "smart" and "dumb" components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.vsnf17nol), written by the creator of Redux. In short, a "smart" component knows about the outside world. A "dumb" component only cares about itself.

### Components should be simple templates
A dumb component is essentially a plain template. It's React 101. The code below should look familiar from the `[components/Link.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-components-link-js)` in the classic [React-Redux Todo example](https://github.com/reactjs/redux/tree/master/examples/todos). The redux manual calls it a [presentational component](http://redux.js.org/docs/basics/UsageWithReact.html#presentational-and-container-components) because all it does is draw itself. It doesn't concern itself with were a variable came from.

Read the code below and ask yourself "where do `active` and `onClick` come from?" If you're able to follow along you'll note that those properties "come from the outside." The `Link` component below is "dumb" because it doesn't care how those properties are defined, it just tries to use them to render its template.

```jsx
import React, { PropTypes } from 'react'

const Link = ({ active, children, onClick }) => {
  if (active) {
    return <span>{children}</span>
  }

  return (
    <a href="#"
       onClick={e => {
         e.preventDefault()
         onClick()
       }}
    >
      {children}
    </a>
  )
}

Link.propTypes = {
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func.isRequired
}

export default Link
```

*Note:* Starting in React 0.14 it's common practice to define components as [stateless functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) using a [fat-arrow function](http://www.2ality.com/2012/04/arrow-functions.html). Below is the simplest example. Notice how the component itself in the example below is one line. You should only need to use the [fancier ES6 class syntax](http://facebook.github.io/react/blog/2015/01/27/react-v0.13.0-beta-1.html#es6-classes) introduced in React 0.13 in special cases. There's a great write-up on the Babel blog on [the ES6 way to write React components](https://babeljs.io/blog/2015/06/07/react-on-es6-plus) but it doesn't cover the new syntax. You may want to read up on how to [use destructuring to simulate named parameters](http://www.2ality.com/2015/01/es6-destructuring.html#simulating-named-parameters-in-javascript).

```jsx
import React, { PropTypes } from 'react'

// create a stateless functional component
// - use a fat-arrow function
// - use descructing to pull name out of props
// - return a JSX template
const DumbComponent = ({ name }) => (<div>How dumb am {name}?</div>)

// specify the properties you need
// (you can leave off `.isRequired` if it's not required)
DumbComponent.propTypes = {
  name: PropTypes.string.isRequired
}

// export the component so someone can put it in a page somewhere
export default DumbComponent
```

*Note:* PropTypes is explained in the massive code block at the top of the [React manual page for reusable components](http://facebook.github.io/react/docs/reusable-components.html).

### Containers connect to the store
If dumb components don't know where the data comes from, who does?! The answer is "smart" components. The Redux manual calls them [container components](http://redux.js.org/docs/basics/UsageWithReact.html#presentational-and-container-components). A container component is the bridge between React and Redux. The library that connects them is called [React-Redux](https://github.com/reactjs/react-redux). Because a container component is gluing together two different things it needs to map the concepts of one to the other. A container component maps the concepts for interacting with a a React component with the concepts for interacting with the Redux store.

React, even without Redux, has several advanced methods that allow components to deeply manage their own state. If you've read a React tutorial you've probably read about the state. Redux stores the state for you and enforces a strict data flow. A container hooks a component into Redux so that it can easily make changes to and read from the store. The interface for this is deceptively simple and best explained with some code. The code below should look familiar from the `[containers/FilterLink.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-containers-filterlink-js)` in the classic [React-Redux Todo example](https://github.com/reactjs/redux/tree/master/examples/todos).

```js
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../redux/modules/todos'
import Link from '../components/Link'

const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)

export default FilterLink
```

It's important to go over the steps that are involved here.

1. `FilterLink` is a container component. It uses the `connect()` function to map values from the Redux `state` and `dispatch` to properties on the `Link` component.

```js
const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)
```

2. `mapStateToProps` and `mapDispatchToProps` are callback functions that are called from within `[connect()](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)`. Under the hood they are using the `[subscribe()](http://redux.js.org/docs/api/Store.html#subscribe)` method from Redux and `componentDidMount()` and `setState()` in React. You may want to check out [what the connect function does](https://github.com/reactjs/react-redux/blob/master/src/components/connect.js#L75) for you but it's enough that you're glad you don't have to write it.

3. The `FilterLink` container is wrapping the `Link` component. A container's entire purpose is to wrap a component in order to connect it to the Redux store. To do that the container will map values from the state to properties on the component. It also maps dispatched actions to functions on the component.

4. `mapStateToProps` is where you can pull values out of the state and define them as properties on the connected component. If you look at the example, the wrapped component will be receiving an `active` property that returns true if the container's `filter` prop is equal to the `visibilityFilter` from the state. The example is designed to show that the container can have props, like `ownProps.filter` that the wrapped component never knows about.

```js
const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}
```

5. `mapDispatchToProps` is where you can create functions that dispatch actions to the store. In Redux there is a strict separation of concerns. In another framework you'd be tempted to have your template actions immediately do something. But in Redux we dispatch every action using a strict flow. This makes it easier to reuse functionality and to creatively mix and match functionality once you get the hang of things.

```js
const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}
```

#### Containers in a nutshell
A container maps values from the Redux store to properties on a React component. It reads from the store when a component loaded and after the store has been updated. In order to closely monitor when the store has been updated, Redux only allows you to dispatch actions. From a container it is not really possible to know what an action does because Redux separates that functionality into reducers.

If the concept of reducers seems totally foreign right now then you completely understand why smart components only dispatch actions and read from the store. Writing to the store is a different task.


### The Store is immutable by convention
If you've been reading up on React you've probably come across libraries such as [Immutable.js](https://facebook.github.io/immutable-js/). Forget about it. Redux completely replaces the need for a library like Immutable.js. You can still use it with Redux (and many people do) but with Redux, Immutable simply enforces the convention by applying a noisy API where one isn't needed. *Hint:* If you're worried about your state being mutated in unexpected ways you should be writing better tests.

There are a great number of fascinating advantages that surface from the simple decision of Redux to store all of the application state in a single immutable object. Redux simply plays traffic cop, controlling how you make store changes and update your interface. This is the next part of React-Redux that can seem very daunting. Everything uses the same store. On the plus side, making two components that share a state is easier than ever and getting your components to update when changes occur is now baked in. On the negative side, it feels a little wild and woolly to overlap the state of every component in your application.

### Modules control specific parts of the store
In the Redux world it's now popular to group Action Types, Action Creators and Reducers into a single file called a "module" or a "[duck](https://github.com/erikras/ducks-modular-redux)." It's actually fine to do it the old way if you want. Both methods make sense for different use cases. If you wanted to separate your constants, actions and reducers into separate files and folders you can feel free to do so but many people first starting out are finding it easier to lump them together.

This starts make more sense with some code. The following is a combination of the `[actions.js](http://redux.js.org/docs/basics/Actions.html#-actions-js)` and the `[reducers.js](http://redux.js.org/docs/basics/Reducers.html#-reducers-js)` in the classic [React-Redux Todo example](https://github.com/reactjs/redux/tree/master/examples/todos). The action creator and reducer have been rewritten using [redux-actions](https://github.com/acdlite/redux-actions) for simplicity. For brevity we're only looking at the `visibilityFilter` example from the link component we're exploring.

```js
import { combineReducers } from 'redux'
import { createAction, handleActions } from 'redux-actions'

// ...for brevity
import todos from './todos'

// Action Types (Constants)
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'

// Action Creators
export const setVisibilityFilter = createAction(SET_VISIBILITY_FILTER)

// Reducers
export const visibilityFilter = handleActions({
  SET_VISIBILITY_FILTER: (state, { payload }) => payload
}, 'SHOW_ALL')

export default combineReducers({
  todos,
  visibilityFilter
});
```

Let's dig into what's going on here:

1. We have an action creator named `setVisibilityFilter()`. In the `FilterLink` container we use that function to create an action object that we dispatch when someone clicks it. In the container you can see that we're calling the action creator function with an argument, like this: `setVisibilityFilter(ownProps.filter)`

2. Action creators are easy to write by hand but using `createAction()` makes it even easier. By default the function creates an action creator that accepts a payload. The vast majority of the time this is all an action needs to do -- marry a payload to an action type. You can see that the `FilterLink` container sends the `ownProps.filter` payload to the `setVisibilityFilter()` action creator. Compare this to [the long-hand version in the Todos example](https://github.com/reactjs/redux/blob/master/examples/todos/actions/index.js#L10-L15).

2. The action actually gets dispatched from the container. Redux uses this `dispatch(action)` construct to allow any container to call an action while still maintaining control over the when the store gets updated. If you're used to MVC you could think of a component as the view, the container as the controller and the actionCreators as the model. Well... sort of. Action creators cover a portion of that you normally get with a model. We'll be rounding this out more as we go along.

3. Once an action is dispatched it is picked up by a reducer. In the example you can see that the `handleActions(map, initialState)` function creates a reducer that can handle a actions named `SET_VISIBILITY_FILTER`. Probably the single ugliest thing about Redux is the reliance on all-caps constants. It's absolutely awful to have these long descriptive strings screaming at you. If you can squint and look past them, reducers start to get really simple.

4. Our reducer for `SET_VISIBILITY_FILTER` accepts the state and the action and returns the new state. In this case the state for `visibilityFilter` is just a simple string. We're returning whatever was passed in as the payload as the new state. What's really subtle is that `combineReducers` is what's pulling out the part of the state that the `visibilityFilter(state, action)` reducer cares about. This makes it possible for our reducer to simply return the `action.payload` as the new state.

5. `[combineReducers](http://redux.js.org/docs/api/combineReducers.html)` is a key part of how the Redux state can allow every component to share a state without causing massive collisions. A reducer is in charge of managing the state that's passed to it. `combineReducers()` calls each reducer it is passed with only the part of the state matching its key. In this case the reducer created by `combineReducers()` will pass `state.visibilityFilter` to the `visibilityFilter(state, action)` function. That clever tick ensures that the reducer won't accidentally overwrite the wrong part of the state. This is also the beginning of some of the magic reusability that Redux enables. Once you begin to master reducers you will be able to mix and match them all you want. The Redux authors refer to this as composability (it's what the `[compose()](http://redux.js.org/docs/api/compose.html)` function is for).

6. The `SET_VISIBILITY_FILTER` constant becomes nearly useless when used in a module. Currently it's best to keep using them to help your code look as much like vanilla Redux as possible. This is probably the next thing to have disappear in a future version of Redux.



http://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux/34623840#34623840