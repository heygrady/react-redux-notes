# React-Redux Notes
Here are my notes from getting to know React Redux a lot better.

### How did we get here?
I've been a web developer for more than a decade. I've seen JavaScript take a massive leap forward several times. First with [Prototype.js](http://prototypejs.org/) then [jQuery](https://jquery.com/) and more recently [Angular](https://angularjs.org/) and [Ember](http://emberjs.com/). But these days everyone is excited about the possibilities of using [React-Redux](http://redux.js.org/docs/basics/UsageWithReact.html). Getting into the Redux ecosystem can be daunting as there are many new tools to learn. It's reminiscent of learning [SCSS](http://sass-lang.com/) for the first time, where many front-end web develpers were first getting used to build tools; even for the simplest of projects. The days of writing HTML, CSS and JavaScript without a build tool are long gone, just like most web developers can't imagine writing plain CSS anymore. Now, in order to build even a single page brocure site, you should be using a build tool or you're doing it wrong. You can get a feel for how complex it's starting to get by reading the "[State of the Art JavaScript in 2016](https://medium.com/javascript-and-opinions/state-of-the-art-javascript-in-2016-ab67fc68eb0b#.6wdwcsm93)" posted in February. It basically lays out all of the tools a modern web developer should be using and not surprisingly those are all the exact same tools you'll see in use in a React-Redux application.

Anyone familiar with getting started on Angular back in the early 1.x days can attest that the biggest hurdle was getting it started. The "best" way to work with Angular was to use build tools, but Angular didn't ship with a CLI or a starter kit. So getting started was really hard. Eventually there was a good [Yeoman generator for Angular](https://github.com/yeoman/generator-angular#readme) available. This made it dead simple to understand all of the core concepts of Angular and start applying them without having to get lost in the underlying tooling. In the early Angular days, building JavaScript bundles was still an insane mess. Read this [epic history of why requireJS is bad](https://gist.github.com/david-mark/2845842) from the man who accidentally created the core idea (also see the [offical requireJS history](http://requirejs.org/docs/history.html) by the creator of RequireJS). In Angular you were still bending over backwards to overcome the weirdest part of working with JavaScript: everything ran in the global scope! One could argue that the vasy majority of Angular 1.x was written to overcome that one messy hurdle.

Ember takes all of that a huge step further into the future with their impressive [ember-cli](http://ember-cli.com/) tool. Used with [Ember Data](https://guides.emberjs.com/v2.5.0/models/) and [JSONAPI](http://jsonapi.org/) (especially with a [Rails 5 API](http://emberigniter.com/modern-bridge-ember-and-rails-5-with-json-api/)). Ember shares a number of core concepts and tooling with React-Redux, including a focus on components and using Babel to bring ES6 modules to the development lifecycle. The biggest problem with Ember is that it makes it [enormously difficult to use NPM packages](http://stackoverflow.com/questions/26544578/how-to-use-third-party-npm-packages-with-ember-cli-app). They have their reasons for this decision, but it's not a restriction that exists in a typical React-Redux App.

The amount of changes between ES5 and ES6 are huge. Now we're supposed to call it ES2015 and we're to expect an ES2016 this year, aka ES7. [Babel](https://babeljs.io/) is helping the JavaScript community avoid the huge [debacle that was Python 3](http://www.donationcoder.com/forum/index.php?topic=37718.0). Essentially Python 3 was very different from Python 2 and that made it nearly impossible to adopt because the old code wouldn't work without changes and the new code would run without an upgrade. People chose not to upgrade. You could find similar trepidation within the Ruby community when they were upgrading from 1.8 to 1.9 and 2.x. That problem eventually lead to the wide adoption of [RVM](http://code.tutsplus.com/articles/why-you-should-use-rvm--net-19529).

The differences between ES5 and ESNext are huge and most browsers [don't support ES6 yet](https://kangax.github.io/compat-table/es6/) (although [Chrome](http://blog.chromium.org/2016/04/es6-es7-in-browser.html) and [Node](https://nodejs.org/en/blog/release/v6.0.0/) are [very close](http://node.green/) now). In browser-land, the [vast array of environments](http://caniuse.com/) your code might run within makes the problems of Python and Ruby seem laughably simple. How does it work? Like SCSS before it, Babel completely side-steps the problem: it converts your code to plain-old JavaScript first. Babel isn't just smoothing over versioning issues for browsers. Now you can even use [babel to write npm modules](http://jamesknelson.com/writing-npm-packages-with-es6-using-the-babel-6-cli/).

With the change over to ES2015 and beyond, JavaScript development is making a dramitic shift towards modernization. A React-Redux application takes full advantage of that new power. Redux makes older tools like Angular and Ember,seem obsolete. In many cases the conventions of React and Redux completely replace the need for more complicaed frameworks. Of course the downside is that Redux development is not on rails. There is no massive framework that holds your hand through the whole process. Instead, Redux gets out of your way and forces you to get to know a new way to think about your apps. And thankfully there is a concensus emerging on the right way to get started on your app.

If you didn't realize it already, [reactive programming is nothing new](http://blog.salsitasoft.com/why-now/).

## React Redux Starter Kit
If you're new to Redux and want to get started there's really no better place than the [react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit) in conjunction with the [redux-cli](https://github.com/SpencerCDixon/redux-cli) tool. If you've been digging into Redux and noticing that it's missing a framework and a CLI, then you just found it. Although it's deceptively named "starter kit", it's the missing app framework and CLI for React-Redux projects. The biggest advantage of the starter kit is that it wires up the Router and the application bootstrapping in a smart way. From there it's easy to start putting your app together. Having the hard work of getting Webpack and Babel wired up with all of the best practices is a massive time saver. You're not likely to need to mess with the defaults until much later. This allows you to get to the hard work of building your app without cluttering your mind with infrastructure.

Because the Redux ecosystem is heavily inspired by the Node ecosystem (a react-redux app uses NPM to manage packages) there isn't ever going to be a full framework like there is for Ember. Instead, building a react-redux app is an exercise in assempling the right tools for your application. The magic in Redux is that it's very simple. The pain is that the simplicity requires you to bring all of the tools yourself. Much of what you need is easy to find in discreet packages that are well maintained and solve yor problems very well. But you're going to have to research to find each of those little tools and stitch them together yourself. In practice this isn't very difficult and is actually more and more of a blessing as your app develops. But for starting out it can be daunting.

The [Redux Todo tutorial](http://redux.js.org/docs/basics/index.html) currently stops at "use middleware to solve async needs." There are dozens of middleware solutions that try to solve the problem of asyncronous actions in a variety of different ways. The classic [redux-thunk](https://github.com/gaearon/redux-thunk) middleware is just the simplistic beginning. Once you start building an app you're going to want to move away from the boilerplate redux you learned in the Todo tutorial and you're going to want to pick the middle that will make building your app as painless as possible.

The biggest missing piece from the starter kit is a strong opinion about middleware. Right now it seems like the community is coalescing around [redux-sagas](http://yelouafi.github.io/redux-saga/index.html). Each project is free to make its own choice in this regard. You'll likely feel extremely opinionated yourself once you get started on your second app.

### Getting Started with the Starter Kit
What you may not realize is that you *must* watch the videos before doing anything with React-Redux.

1. [Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) - By the creator of Redux, Dan Abramov. These videos are more essential than you may realize. It's nearly impossible to understand Redux without watcing them.
2. [Official Redux documentation](http://redux.js.org/) - Also by the creator of Redux. Writen in a similar style to the videos and covers all of the same code. After watching the videos you will want to try to build the apps. You can do this by reading the code in the manual.
3. [Starting out with react-redux-starter-kit](https://suspicious.website/2016/04/29/starting-out-with-react-redux-starter-kit/) - Blog post about how to use the react-reduc-starter-kit. It covers all of the basics of the project layout. If you skim the [write-up on fractal project structures](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure) you'll see many of the same concepts.

### High-level Concepts
It's important to get a picture of how a starter-kit app is strucutred because it makes it clear how to apply the core principles of React-Redux. At it's core React-Redux is a marriage of components (React) with a store (Redux). Most people get a little lost at this point so here's a quick example using a plain React components.

There's a good [write-up about "smart" and "dumb" components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.vsnf17nol), written by the creator of Redux. In short, a "smart" component knows about the outside world. A "dumb" component only cares about itself.

#### Components should be simple templates
The code below should look familiar from the [components/Link.js](http://redux.js.org/docs/basics/ExampleTodoList.html#-components-link-js) in the classic React-Redux Todo example. The template, the redux manual calls it a [presentational component](presentational-components) because all it does is draw itself with some variables. It doesn't concern itself with were a variable came from.

Read the code below and ask yourself "where do `active` and `onClick` come from?" If you're able to follow along you'll note that those properties "come from the outside." The `Link` component below is "dumb" becuase it doesn't care how those properties are defined, it just blindly tries to use them to render its template.

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

*Note:* Starting in React 0.14 it's common practice to define components as [stateless functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) using a [fat-arrow function](http://www.2ality.com/2012/04/arrow-functions.html). Below is the simplest example. Notice how the component itself is one line. You should only need to use the [fancier ES6 class syntax](http://facebook.github.io/react/blog/2015/01/27/react-v0.13.0-beta-1.html#es6-classes) from React 0.13 in special cases. There's a great write-up on the Babel blog on [the ES6 way to write React components](https://babeljs.io/blog/2015/06/07/react-on-es6-plus). You may want to read up on how to [use destructuring to simulate named parameters](http://www.2ality.com/2015/01/es6-destructuring.html#simulating-named-parameters-in-javascript).

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

*Note:* PropTypes is explained in the massive code block at the top of the [React manual page for resusable components](http://facebook.github.io/react/docs/reusable-components.html).

### Containers connect to the store



http://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux/34623840#34623840