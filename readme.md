# React-Redux Notes
Here are my notes from getting to know React Redux a lot better.

### How did we get here?
I've been a web developer for more than a decade. I've seen JavaScript take a massive leap forward several times over the years. First with [Prototype.js](http://prototypejs.org/) then [jQuery](https://jquery.com/) and more recently with [Angular](https://angularjs.org/) and [Ember](http://emberjs.com/). JavaScript has been maturing slowly for what seems like forever. But these days everyone is excited about the possibilities of ES6. And the premier way to build apps in with ES6 is [React-Redux](http://redux.js.org/docs/basics/UsageWithReact.html).

Getting into the Redux ecosystem can be daunting as there are many new tools to learn, even if you've been trying to keep up. It's reminiscent of learning [SCSS](http://sass-lang.com/) for the first time where suddenly there's a build step where there wasn't one before. The days of writing HTML, CSS and JavaScript without a build tool are long gone. Now in order to create even a simple brochure site you should be using a build tool or you're doing it wrong. You can get a feel for how complex it's starting to get by reading the "[State of the Art JavaScript in 2016](https://medium.com/javascript-and-opinions/state-of-the-art-javascript-in-2016-ab67fc68eb0b#.6wdwcsm93)" article from February. It basically lays out all of the tools a modern web developer should be using and, not surprisingly, those are all the exact same tools you'll see in use in a React-Redux application.

Anyone who got started on Angular back in the early 1.x days can attest that the biggest hurdle was... getting it started. The "best" way to work with Angular was to use build tools but Angular didn't ship with a CLI or a starter kit. So getting started was really hard! Eventually there was a good [Yeoman generator for Angular](https://github.com/yeoman/generator-angular#readme) available. This made it dead simple to understand all of the core concepts of Angular and start applying them without having to get lost in the underlying tooling.

In the early Angular days JavaScript build tools were still an insane mess. How messy? Read this [epic anti-history of requireJS](https://gist.github.com/david-mark/2845842) from the man who accidentally inspired the [official requireJS history](http://requirejs.org/docs/history.html). In Angular 1.x you were still bending over backwards to overcome the weirdest part of working with JavaScript: everything ran in the global scope! One could argue that the vast majority of Angular 1.x was written to overcome that one messy hurdle. It was barely a step up from the [IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/) that were the best practice in the jQuery days.

With ES6 and Babel we're now able to use the new `import` syntax for [JavaScript modules](http://www.2ality.com/2014/09/es6-modules-final.html). It encapsulates all of your code, similar in concept to an IFFE, and gives you tight control over how you include and export code in your project. It makes JavaScript feel like am moderns language. It brings the power of Node and NPM to browser development. With Webpack it gets even better. If you've got the right setup you can focus on your code and trust that everything will just work.

Ember takes all of that a huge step further into the future with their impressive [ember-cli](http://ember-cli.com/) tool. Used with [Ember Data](https://guides.emberjs.com/v2.5.0/models/) and [JSONAPI](http://jsonapi.org/) (especially with a [Rails 5 API](http://emberigniter.com/modern-bridge-ember-and-rails-5-with-json-api/)) Ember is a powerful and productive framework. Ember shares a number of core concepts and tooling with React-Redux, including a focus on components and using Babel to bring ES6 modules to the development life cycle. The biggest problem with Ember is that it makes it [enormously difficult to use NPM packages](http://stackoverflow.com/questions/26544578/how-to-use-third-party-npm-packages-with-ember-cli-app). They have their reasons for this decision, but it's not a restriction that exists in a typical React-Redux App.

### ES6 + Babel is a game changer
ES6 has arrived. Now we're [supposed to call it ES2015](http://meta.stackoverflow.com/questions/297774/rename-es6-to-es2015) and we're to expect an [ES2016](http://www.2ality.com/2016/01/ecmascript-2016.html) this year, aka ES7. [Babel](https://babeljs.io/) is helping the JavaScript community avoid the huge [debacle that was Python 3](http://www.donationcoder.com/forum/index.php?topic=37718.0) (seriously, skim these: [1](http://blog.startifact.com/posts/alex-gaynor-on-python-3.html), [2](https://alexgaynor.net/2013/dec/30/about-python-3/), [3](https://plus.google.com/+IanBicking/posts/iEVXdcfXkz7) and realize just how well-executed the ES6 roll-out has been). Python 3 was different enough from Python 2 that it was nearly impossible to adopt. The old code wouldn't work on the new engine. Many people in the Python community simply chose not to upgrade. You could find similar trepidation within the Ruby community when they were upgrading from 1.8 to 1.9 and 2.x. That problem eventually lead to the wide adoption of [RVM](http://code.tutsplus.com/articles/why-you-should-use-rvm--net-19529). These days if you're not using RVM to manage multiple versions of Ruby on your machine you're doing it wrong. There's even a Node clone of RVM called [NVM](https://github.com/creationix/nvm) that manages multiple versions of Node for those that need it (you probably don't).

Upgrading a language can be excruciatingly painful for a community of developers. We all remember the pains of trying to support IE6, IE7, IE8, IE9... isn't this new version of JS going to break everything?

The differences are huge and most browsers [don't support ES6 yet](https://kangax.github.io/compat-table/es6/) (although [Chrome](http://blog.chromium.org/2016/04/es6-es7-in-browser.html) and [Node](https://nodejs.org/en/blog/release/v6.0.0/) are [*very close*](http://node.green/) now). In browser-land, the [complicated matrix of (un)supported features](http://caniuse.com/) makes the problems of Python and Ruby seem laughably simple. How does it work? Like SCSS before it, Babel completely side-steps the problem: it converts your shiny new ES6 code to plain-old JavaScript first. Babel isn't just smoothing over language compatibility issues for browsers. You can even use [babel to write npm modules](http://jamesknelson.com/writing-npm-packages-with-es6-using-the-babel-6-cli/).

**If you're not using Babel you're doing it wrong.**

*Note:* Yes, [TypeScript](https://www.typescriptlang.org/) exists. [Don't use it](https://www.quora.com/Should-I-go-with-Babel-or-TypeScript-for-writing-web-apps).

*Note:* Babel is highly configurable and is smart enough only to "transpile" what's needed for your target system. If you're using Babel for Node-only development you can use something like [babel-preset-node6](https://www.npmjs.com/package/babel-preset-node6) which only touches code that Node 6 doesn't yet support.

### Reactive Programming is better than OOP
With the change over to ES2015 and beyond, JavaScript development is making a dramatic shift towards modernization. A React-Redux application takes full advantage of that new power. Often then best way to do something in Redux is to simply write ES6-styled JavaScript in a smart way. Redux makes older OOP frameworks like Angular and Ember seem obsolete because ES6 makes JavaScript much easier to write -- with ES6 you don't need a framework to help you fight the deficiencies in the language. In many cases the conventions-based-approach of React and Redux can completely replace the need for more complicated OOP frameworks. Of course the downside is that Redux development is *mostly* about conventions, not frameworks. There is no massive framework that holds your hand through the whole process. Instead, Redux gets out of your way and forces you to get to know a new way to think about writing apps.

React are Redux are based on functional programming (FP) techniques (also called reactive programming) while Angular and Ember are based on object-oriented programming (OOP) techniques.

If you didn't realize it already, [reactive programming is nothing new](http://blog.salsitasoft.com/why-now/), but it is fairly new to JavaScript. The biggest hurdle is rearranging your thinking away from OOP frameworks like Ember and embracing functional programming instead. Bear in mind it's possible that [React-Redux is getting it all wrong](http://staltz.com/why-react-redux-is-an-inferior-paradigm.html).

# React Redux Starter Kit
If you're new to Redux and want to get started there's really no better place than the [react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit) in conjunction with the [redux-cli](https://github.com/SpencerCDixon/redux-cli) tool. If you've been digging into Redux and noticing that it's missing an app framework and a CLI, then you just found it! Although it's deceptively named "starter kit", it's the missing app framework and CLI for React-Redux projects. The biggest advantage of the starter kit is that it wires up the [router](https://github.com/reactjs/react-router-redux) and bootstraps your application better than you would all by yourself. It sets up Webpack and Babel for you and shows you a great way to organize your code. From there it's easy to start putting your app together.

Having the hard work of getting Webpack and Babel working up with all of the best practices is a *massive* time saver. You're not likely to need to mess with the defaults until much later in your dev process. This allows you to get to the hard work of building your app without getting lost in the tooling.

Because the Redux ecosystem is heavily inspired by the Node ecosystem (a React-Redux app uses NPM to manage packages) there isn't ever going to be a full framework like there is for Ember. Instead, building a React-Redux app is an exercise in assembling the right tools for your application. Sadly, beyond the foundations in the starter-kit you're going to have to research to find each of those little tools and stitch them together yourself. Much of what you need is easy to find on NPM. In practice tis level of control actually becomes a blessing as your app matures. However, for starting out it can be daunting.

### NPM is your friend
If you're completely new to the Node ecosystem you might be worried about things like how all of your code fits together. It becomes clearer with practice. You might enjoy reading about [ES6 modules in depth](https://hacks.mozilla.org/2015/08/es6-in-depth-modules/). The short version is that you don't have to worry about your code colliding with other code. If you're looking at a file that uses `import` then you know exactly what code is going to run in that file. If you look at the `export` declarations in a file then you'll know exactly what will be available when you import it. When you need a package, install it with NPM.

One thing to note is that the files in your project are all imported relative to each other. This is extremely powerful because it allows you to organize your code in any way you want. The only restriction is that if you want your code to run you have to import it *somewhere*. It's hard to really grasp how powerful it is to have no restrictions on how your organize your code. There's no *wrong* place to put your files.

- Don't bother with Bower unless it's the only way to get something you need (sometimes bower packages include code that npm packages don't). For the most part a Bower packer can be installed with NPM.
- Don't bother with [alternative package managers](http://andrewhfarmer.com/javascript-frontend-package-managers/).
- Don't worry about Node vs Browser. If you're trying to write universal code that runs in the browser and on the server, remember this simple rule: With very few exceptions, *all JavaScript code is universal*.
- The one big exception is, duh, [Node-specific APIs](https://nodejs.org/dist/latest/docs/api/). When in doubt, only use functions you can find on [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript). The Node-specific API won't work in the browser but the vast majority of what Node offers is powered by V8, the same JavaScript engine used in Chrome. If you're constantly running into issues where your code isn't working in a browser because it's Node-specific you are probably doing something *very wrong*.
- If you're trying to evaluate if a package is likely to work for your project a good rule of thumb is "does it work with the filesystem or not?" That's usually a good filter. Some NPM packages are clearly designed to work with the server and they're usually easy to spot. 

## Getting Started with the Starter Kit
What you may not realize is that you *must* watch the videos before doing anything with React-Redux. You have to watch the videos before you can even read the manual. **If you haven't watched the videos you shouldn't be reading this right now!**

1. **Watch [Getting Started with Redux](https://egghead.io/series/getting-started-with-redux)** - By the creator of Redux, Dan Abramov. These videos are more essential than you may realize. It's nearly impossible to understand Redux without watching them.

2. **Read the [Official Redux documentation](http://redux.js.org/)** - Also by the creator of Redux. Written in a similar style to the videos and covers much of the same code. After watching the videos you will want to try to build the example Todo app. You can do this by reading the code in the manual.

3. **Read [Starting out with react-redux-starter-kit](https://suspicious.website/2016/04/29/starting-out-with-react-redux-starter-kit/)** - Blog post about how to use the react-reduc-starter-kit. It covers all of the basics of the project layout. If you skim the [write-up on fractal project structures](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure) you'll see many of the same concepts. The starter-kit example shows an example of using Redux-thunk to use a remote API. It's worth getting to know that example to better understand the problem that middleware is trying to solve.

4. **Quit thinking about universal/isomorphic and native apps for now.** - You're getting ahead of yourself. Once you get the hang of Redux the other stuff is easy and already solved for you. Seriously. If you have a fully functioning React-Redux application it can be made "universal" in a matter of minutes if it isn't already. Porting it to [React Native](https://facebook.github.io/react-native/) will be different than what you're imagining if you've never tried. Regardless, these are things that can wait until later.

5. **Read the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)** - This style guide is a nice primer on the day-to-day ES6 features you need to use and how to use them. It's possible to lint your code using the Airbnb rules which -- because of how well it's documented -- can help you casually learn about best practices. However, the starter kit uses the [standard](http://standardjs.com/rules.html) rules for the linter and you should too. The biggest difference is that the standard rules **don't use semicolons.**

6. **[Build the Todo example](http://redux.js.org/docs/basics/Actions.html)** - The manual links to a [complete Todo app](http://redux.js.org/docs/introduction/Examples.html#todos) built to match what's documented in the manual. Because the Redux API is so transparent, much of the concepts only make sense once you start to write code. This means that, unlike other frameworks where you can read the API for hours, Redux can be best learned by actually writing some code.

The [Redux Todo tutorial](http://redux.js.org/docs/basics/index.html) currently stops at "use middleware to solve your async needs." There are dozens of middleware solutions that try to solve the problem of asynchronous actions in a variety of different ways. The classic [redux-thunk](https://github.com/gaearon/redux-thunk) middleware is just the simplistic beginning. Once you start building an app you're going to want to move away from the plain Redux you learned in the Todo tutorial and pick the middleware that fits your working style.

## Middleware blues
The biggest missing piece from the starter kit is a strong opinion about [middleware](http://redux.js.org/docs/advanced/Middleware.html). Don't get slowed down if you don't understand middleware right now. What you need to know is that you will use someone else's middleware, you likely won't write your own. The Redux community is coalescing around [redux-sagas](http://yelouafi.github.io/redux-saga/index.html) as the middleware of choice. Each project is free to make its own choice in this regard and Redux seems to have left this gap wide open on purpose. You'll likely feel extremely opinionated yourself once you get started on your *second* app.

When we get there you'll want to choose between [redux-saga](https://github.com/yelouafi/redux-saga), [redux-effects](https://github.com/redux-effects/redux-effects), [redux-side-effects](https://github.com/gregwebs/redux-side-effect), and [redux-loop](https://github.com/raisemarketplace/redux-loop). Unless of course you're in love with [Redux-thunk](https://github.com/gaearon/redux-thunk).

We're going to be [using Redux-Sagas](http://riadbenguella.com/from-actions-creators-to-sagas-redux-upgraded/) and [you should too](http://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux/34623840#34623840). But there are other options for [Redux effects middleware](https://blog.hivejs.org/building-the-ui-2/) if the Sagas middleware isn't right for you.

## High-level Concepts
It's important to get a picture of how a typical starter-kit app is structured because it makes it clear how to apply the core principles of React-Redux. At it's core React-Redux is a marriage of components to a store (React deals with components, Redux deals with the store). Most people get a little lost at this point so we'll try to dive into a quick example starting with a plain React component (see below).

There's a good [write-up about "smart" and "dumb" components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.vsnf17nol), by the creator of Redux. In short, a "smart" component knows about the outside world. A "dumb" component only cares about itself.

*Note:* It's now passé to refer to presentational components as "dumb." It's not cool to pass judgment, they're just components, there's no need to be rude about it ;) You actually have to know about words like dumb and smart and duck because they're still used all over the Redux ecosystem. We need to know the old terms while we wait for the documentation to catch up to trends.

**A redux app is made of:**
- components (dumb)
- containers (smart)
- modules (duck)

**A duck is made of:**
- constants
- actions
- reducers

*Note:* Everything we're showing here is from the [Todo example in the Redux manual](http://redux.js.org/docs/basics/ExampleTodoList.html)... although slightly rewritten to show how experienced developers apply those core concepts. You'll be less confused if you've studied that tutorial.

## Components should be simple templates
A dumb component is essentially a plain template. The code below should look familiar from the [`components/Link.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-components-link-js) file in the classic [React-Redux Todo example](https://github.com/reactjs/redux/tree/master/examples/todos). The redux manual calls it a [presentational component](http://redux.js.org/docs/basics/UsageWithReact.html#presentational-and-container-components) because all it does is render itself. It doesn't concern itself with were a variable came from.

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

1. The `Link` component is a stateless functional component. You can define these as simple functions that accept `props` argument. Here we're destructuring the props argument into `active`, `children`, and `onClick`. We're using a fat-arrow function to be cool.

  ```js
  const Link = ({ active, children, onClick }) => {
    //...
  }
  ```
2. We're using the `active` prop to return a `span` or an `a[href]`.

3. The `onClick` attribute of the `a[href]` calls the `onclick()` property on the `Link`. The `onclick()` prop must be passed in from the outside.

4. If you're defining a component that **only** needs to return a template you can use parenthesis to wrap your JSX template. This allows for multiline templates to be used with a one-line [fat-arrow function with an implied return](http://stackoverflow.com/questions/28889450/when-should-i-use-return-in-es6-arrow-functions).

  ```jsx
  const SimpleLink = (onClick, children) => (
    <a href="#"
       onClick={e => {
         e.preventDefault()
         onClick()
       }}
    >
      {children}
    </a>
  )
  ```

*Note:* Starting in React 0.14 it's common practice to define components as [stateless functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) using a [fat-arrow function](http://www.2ality.com/2012/04/arrow-functions.html).

*Note:* You may want to read up on how to [use destructuring to simulate named parameters](http://www.2ality.com/2015/01/es6-destructuring.html#simulating-named-parameters-in-javascript).

#### Example if a barebones component
*Note:* Below is the simplest example of a component. Notice how the component itself is just one line.

```jsx
import React, { PropTypes } from 'react'

// create a stateless functional component
// - use a fat-arrow function
// - use descructing to pull name out of props
// - return a JSX template
const DumbComponent = ({ name }) => <div>How dumb am {name}?</div>

// specify the properties you need
// (you can leave off `.isRequired` if it's not required)
DumbComponent.propTypes = {
  name: PropTypes.string.isRequired
}

// export the component so someone can put it in a page somewhere
export default DumbComponent
```

*Note:* PropTypes is explained in the massive code block at the top of the [React manual page for reusable components](http://facebook.github.io/react/docs/reusable-components.html).

#### Example of a component using the class syntax
*Note:* You should only need to use the [fancier ES6 class syntax](http://facebook.github.io/react/blog/2015/01/27/react-v0.13.0-beta-1.html#es6-classes) introduced in React 0.13 in special cases. There's a great write-up on the Babel blog on [the ES6 way to write React components](https://babeljs.io/blog/2015/06/07/react-on-es6-plus) but it doesn't cover the new syntax.

```jsx
import React, { Component, PropTypes } from 'react'

// create a class component
// - use an es6 class
class DumbComponent extends Component {
  
  // commonly you are using a class
  // to fire an action when the component mounts
  // or hook up DOM events that are hard to express inline
  // @see https://gist.github.com/koistya/934a4e452b61017ad611
  componentDidMount() {
    this.props.fetchInitialState()
  }
  
  // you need to define a render function
  render() {

    // props are available as a property of this
    const { name } = this.props

    return <div>How dumb am {name}?</div>
  }
}

// specify the properties you need
// it's still prettier to define them down here
// @see https://babeljs.io/blog/2015/06/07/react-on-es6-plus
DumbComponent.propTypes = {
  name: PropTypes.string.isRequired
}

// export the component so someone can put it in a page somewhere
export default DumbComponent
```

*Note:* If you're trying to read about React, ignore how they use the old `React.createClass()` syntax in the manual. Never use something like `React.createClass()` unless, inexplicably, you're not allowed to use ES6. But really... you should be using ES6 no matter what by this point. There is no reason not to be on the ES6 train.

## Containers connect to the store
If dumb components don't know where the data comes from, who does?! The answer is "smart" components. The Redux manual calls them [container components](http://redux.js.org/docs/basics/UsageWithReact.html#presentational-and-container-components). A container component is the bridge between React and Redux. The library that connects them is called [React-Redux](https://github.com/reactjs/react-redux). Because a container component is gluing together two different things it needs to *map* the concepts of *one to the other*. A container component maps the concepts for interacting with a a React component with the concepts for interacting with the Redux store.

A container hooks a component into Redux. React, even without Redux, has several advanced methods that allow components to deeply manage their own state. If you've read a React tutorial you've probably read about the [component state](https://facebook.github.io/react/docs/component-api.html#setstate). Forget about that! Redux stores the state for you. Redux enforces a strict data flow in order to know precisely when components need to update. The container can dispatch actions and read from the store. The container handles events from the component and passes data to it. The interface for this is deceptively simple and best explained with some code. The code below should look familiar from the ['containers/FilterLink.js`](http://redux.js.org/docs/basics/ExampleTodoList.html#-containers-filterlink-js) in the classic [React-Redux Todo example](https://github.com/reactjs/redux/tree/master/examples/todos).

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

1. `FilterLink` is a container component. It uses the `connect()` function to map values from the Redux `state` and `dispatch()` to properties on the `Link` component. There's a reason the two arguments are "map state" and "map dispatch" to props.

  ```js
  const FilterLink = connect(
    mapStateToProps,
    mapDispatchToProps
  )(Link)
  ```

2. `mapStateToProps` and `mapDispatchToProps` are callback functions that are called from within `[connect()](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)`. Under the hood they are using the `[subscribe()](http://redux.js.org/docs/api/Store.html#subscribe)` method from Redux and `componentDidMount()` and `setState()` in React. You may want to check out [what the connect function does](https://github.com/reactjs/react-redux/blob/master/src/components/connect.js#L75). Whatever the magic to goes on under the hood, it's important to know that `connect()` ensures that when the state changes the component will be rendered with the updated props from the store.

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

### Containers in a nutshell
A container maps values from the Redux store to properties on a React component. It reads from the store when a component loaded and after the store has been updated. In order to closely monitor when the store has been updated, Redux only allows you to dispatch actions. From a container it is not really possible to know what an action does because Redux separates that functionality into reducers.

If the concept of reducers seems totally foreign right now then you completely understand why smart components only dispatch actions and read from the store. Writing to the store is a different task.

Containers are all about reading from the store and sending actions. Work is done elsewhere.


## The Store is immutable by convention
If you've been reading up on React and Redux you've probably come across libraries such as [Immutable.js](https://facebook.github.io/immutable-js/). Forget about it. Redux completely replaces the need for a library like Immutable.js. You can still use it with Redux (and many people do) but Immutable simply enforces what Redux defines by convention. *Hint:* If you're worried about your state being mutated in unexpected ways you should be writing better tests.

There are a great number of fascinating advantages that surface from the simple decision of Redux to store all of the application state in a single immutable object. Redux is a methodology and minimal toolkit for working with the store. This is the next part of React-Redux that can seem very daunting. Redux's simplicity becomes it's great power. Utilizing barebones JavaScript and good conventions is the spirit of Redux. There is no need for something like Immutable when you're doing Redux correctly.

### The Store is what you make of it
Everything uses the same store. If you put something in the store a container can read from it. There are no restrictions of which part of the store you can write to or read from. At it's base level it might feel a little disorganized or even insecure. But the store is what you make of it. We'll see that people commonly organize things into modules and use some basic Redux functionality that makes things a little less scary.

We'll see later that there are some conventions emerging about how to maintain data that you recieved from an API. Those patterns can be stamped into other parts of your app because of how well Redux encapulates functionality.

## Modules control specific parts of the store
In the Redux world it's now popular to group Action Types, Action Creators and Reducers into a single file called a "module" or a "[duck](https://github.com/erikras/ducks-modular-redux)." It's actually fine to do it the old way if you want. Both methods make sense for different use cases. If you wanted to separate your constants, actions and reducers into separate files and folders you can feel free to do so but many people first starting out are finding it easier to lump them together.

This starts make more sense with some code. The following is a combination of the [`actions.js`](http://redux.js.org/docs/basics/Actions.html#-actions-js) and the [`reducers.js`](http://redux.js.org/docs/basics/Reducers.html#-reducers-js) files in the classic [React-Redux Todo example](https://github.com/reactjs/redux/tree/master/examples/todos). The action creator and reducer have been rewritten using [redux-actions](https://github.com/acdlite/redux-actions) for simplicity. For brevity we're only looking at the `visibilityFilter` example from the link component we're exploring. A lot changed here but it will make sense in a second.

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
  [SET_VISIBILITY_FILTER]: (state, { payload }) => payload
}, 'SHOW_ALL')

export default combineReducers({
  todos,
  visibilityFilter
});
```

Let's dig into what's going on here:

1. We have an action creator named `setVisibilityFilter()`. In the `FilterLink` container we use our action creator function to create an action object that we dispatch when someone clicks the `Link`. An action creator is a simple function that returns an object with a `type` and a `payload`.

  ```js
  export const setVisibilityFilter = createAction(SET_VISIBILITY_FILTER)
  console.log(setVisibilityFilter)
  /* -->
  function(payload) {
    return { type: SET_VISIBILITY_FILTER, payload }
  }
  */
  ```

2. Action creators are easy to write by hand but using `createAction()` makes it even easier. By default the function creates an action creator that accepts a payload. The vast majority of the time this is all an action needs to do -- marry a payload to an action type. You can see that the `FilterLink` container sends the `ownProps.filter` payload to the `setVisibilityFilter(payload)` action creator. Compare this to [the long-hand version in the Todos example](https://github.com/reactjs/redux/blob/master/examples/todos/actions/index.js#L10-L15).

  ```js
  const actionObject = setVisibilityFilter(payload)
  console.log(actionObject)
  // --> { type: 'SET_VISIBILITY_FILTER', payload: 'some string' }
  ```

2. The action actually gets dispatched from the container. Redux uses this `dispatch(action)` construct to allow any container to call an action while still maintaining control over the when the store gets updated. If you're used to MVC you could think of a component as the view, the container as the controller and the actionCreators as the model. Well... sort of. Action creators cover a portion of that you normally get with a model. We'll be rounding this out more as we go along.

  ```js
  // from the FilterLink container
  const mapDispatchToProps = (dispatch, ownProps) => {
    return {
      onClick: () => {
        dispatch(setVisibilityFilter(ownProps.filter))
        // same as dispatch({ type: 'SET_VISIBILITY_FILTER', payload: ownProps.filter })
      }
    }
  }
  ```

3. After an action is dispatched it is handled by a reducer. In the example you can see that the `handleActions(map, initialState)` function creates a reducer that can handle an action named `SET_VISIBILITY_FILTER`. Probably the single ugliest thing about Redux is the reliance on all-caps constants. It's absolutely awful to have these long descriptive strings screaming at you. If you can squint and look past them, reducers start to get really simple.

  ```js
  export const visibilityFilter = handleActions({
    [SET_VISIBILITY_FILTER]: (state, { payload }) => payload
  }, 'SHOW_ALL')
  console.log(visibilityFilter);
  /* -->
  function(state = 'SHOW_ALL', action) {
    switch() {
      case SET_VISIBILITY_FILTER: return action.payload
      default: return state
    }
  }
  */
  ```

4. Our reducer for `SET_VISIBILITY_FILTER` accepts the `state` and the `action` and returns the new state. In this case the state for `visibilityFilter` is just a simple string. We're returning whatever was passed in as the payload as the new state. What's really subtle is that `combineReducers()` is what's pulling out the part of the state that the `visibilityFilter(state, action)` reducer cares about. This makes it possible for our reducer to simply return the `action.payload` as the new state instead of trying to store it in the `state.visibilityFilter` property. That's slightly confusing because where you store something in the state determines how read something. Not explicitly stating how our state is stored is actually a hidden power of Redux. The confusing part simply goes away once you've worked with it for a while.

5. [`combineReducers()`](http://redux.js.org/docs/api/combineReducers.html) is a key part of how the Redux state can allow every component to share a state without causing massive collisions. A reducer is in charge of managing the state that's passed to it. `combineReducers()` calls each reducer it is passed with only the part of the state matching its key. In this case the reducer created by `combineReducers()` will pass `state.visibilityFilter` to the `visibilityFilter(state, action)` function. That clever tick ensures that the reducer won't accidentally overwrite the wrong part of the state. This is also the beginning of some of the magic reusability that Redux enables. Once you begin to master reducers you will be able to mix and match them all you want. The Redux authors refer to this as composability (it's what the [`compose()`](http://redux.js.org/docs/api/compose.html) function is for).

  ```js
  // how we wrote it in the example above
  export default combineReducers({
    todos,
    visibilityFilter
  });

  // a toy example replacing combineReducers
  // check out the real deal: https://github.com/reactjs/redux/blob/master/src/combineReducers.js
  export default function(state, action) {
    // we have an object whose keys match the name of a reducer
    const reducers = {
      todos,
      visibilityFilter
    }

    // we loop through the keys
    // calling a function for each key
    Object.keys(reducers).map(key => {

      // capture the reducer and subState by key
      const reducer = reducers[key]
      const subState = state[key]

      // return a new state
      // replace the key with the new sub state from the reducer
      return {
        ...state,
        key: reducer(subState, action)
      }
    })
  }
  ```

*Note:* If you're noticing that the use of `ALL_CAPS` constants is annoying you are not alone. For now we'll stick with them but we'll quickly see that you don't use them for very much once you start building things.

# Next

1. Next we'll build the Todo app from the manual with react-redux-starter-kit and redux-cli.
1. We'll use Redux-saga to manage our asynchronous side effects.
1. We'll make it work with a JSONAPI server for persisting todos to the database.