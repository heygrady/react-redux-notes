# Using JSONAPI Server with react-redux-starter-kit
This document covers the creation of a simple JSONAPI server for use with [our example todos app](./react-redux-starter-kit-todos.md). We're going to use the excellent [jsonapi-server](https://github.com/holidayextras/jsonapi-server) to easily manage our API and ensure it's fully conforming with the [JSON API spec](http://jsonapi.org/).

Of course you can use any JSON format that suits the needs of your project but JSON API has become the defacto standard in the [Ember](http://emberjs.com/blog/2015/06/18/ember-data-1-13-released.html) and [Rails](https://wyeworks.com/blog/2015/4/20/rails-api-is-going-to-be-included-in-rails-5/) communities. You can see that there are already many [implementations of the JSON API spec](http://jsonapi.org/implementations/). This makes it attractive for other platforms because your projects will instantly be compatible with other tools. Most frameworks tend to follow the Rails community's lead.

For lack of a better standard, and because we need our API server to *just work*, we're going to use jsonapi-server.

### Node and ES2015
We're going to be writing everything in ES2015 to keep it consistent with the frontend app. This is slightly swimming up stream but [Node is rapidly gaining ES2015 features](http://node.green/) and it's dead-simple to use something like [`babel-preset-node5`](https://www.npmjs.com/package/babel-preset-node5) to only use babel for the parts your version of Node is missing.

You can read more about [using ES6 with npm](http://mammal.io/articles/using-es6-today/).

**NOTE:** You should install the [current version of node](https://nodejs.org/en/download/current/) available for your platform. There is no good reason to wait unless you are using an extremely old package that relies on an obscure deprecated feature of Node. You shouldn't be using [packages that aren't compatible](https://github.com/nodejs/node/issues/2798) with the latest version of Node.

**NOTE:** If you are using a different version of Node, use the proper `babel-preset-nodeX` for your version: [babel-preset-es2015-node4](https://www.npmjs.com/package/babel-preset-es2015-node4), [babel-preset-node5](https://www.npmjs.com/package/babel-preset-node5), [babel-preset-node6](https://www.npmjs.com/package/babel-preset-node6).

## Install jsonapi-server
We're assuming that you've been building something like the [todos app described in the other notes](./react-redux-starter-kit-todos.md). In the previous notes we were doing our work in a `~/Projects/tests/todos-app/` folder.

For the API server we're going to be create a `~/Projects/tests/todos-api/` folder and copying some files from the other folder. *If you're using a slightly different folder structure you may have to pay attention below*.

```bash
# starting in a folder like ~/Projects/tests/

# create boilerplate jsonapi-server app
mkdir todos-api && cd todos-api

# initialize npm and git
npm init --yes
git init

# install jsonapi-server and dev dependencies
npm install --save jsonapi-server
npm install --save-dev \
  babel-register \
  babel-preset-node5 \
  babel-plugin-transform-runtime \
  eslint \
  babel-eslint \
  eslint-plugin-flow-vars \
  eslint-plugin-react \
  eslint-plugin-standard \
  eslint-plugin-promise \
  eslint-config-standard \
  eslint-config-standard-react \
  eslint-watch \
  nodemon \
  mocha
# TODO: need add karma-coverage

# create a bunch of config files
# pay attention: copies some files from the todos-app
echo -e "{\n  \"presets\": [\"node5\"],\n  \"plugins\": [\"transform-runtime\"]\n}" >> .babelrc
cat ../todos-app/.editorconfig >> .editorconfig
cat ../todos-app/.eslintrc >> .eslintrc
cat ../todos-app/.gitignore >> .gitignore
cat .gitignore >> .npmignore
echo lib >> .gitignore

# create the project structure
mkdir src && mkdir src/handlers && mkdir src/resources && mkdir test
touch src/server.js
```

## Configuring our JSONAPI Server
We're going to keep things simple and use the [suggested project structure](https://github.com/holidayextras/jsonapi-server/blob/master/documentation/suggested-project-structure.md) from the jsonapi-server docs. In the previous step we created a `src/` folder and added `server.js`, `handlers/` and `resources/`. Now we need to fill in our `server.js` file.

We need to create our `server.js` file in `src/server.js`. The [suggested project structure](https://github.com/holidayextras/jsonapi-server/blob/master/documentation/suggested-project-structure.md) contains a good base for our project however we'll be using es2015 in this app so the code will need to look slightly different. Here's a rewrite that uses the fancy syntax and passes the same linter we're using for the front-end. Switching the code-base to ES2015 and using the same linting rules as our frontend app makes it easier to switch between contexts.

##### `src/server.js`

```js
import jsonApi from 'jsonapi-server'
import fs from 'fs'
import path from 'path'

jsonApi.setConfig({
  // Put your config here!
  port: 16006,
  base: 'api/v1'
})

// Load all the resources
const isFile = /^[a-z].*\.js$/
fs.readdirSync(path.join(__dirname, 'resources'))
  .filter(filename => isFile.test(filename))
  .map(filename => path.join(__dirname, 'resources', filename))
  .forEach(require)

jsonApi.start()
console.log(`Server listening on http://localhost:${jsonApi._apiConfig.port}${jsonApi._apiConfig.base}`)
```

## Adding a resource
We need to make a resource file. Helpfully, `jsonapi-server` makes it easy to create resources with default data using the `examples` property. You can learn more about defining resources in the [ resource docs](https://github.com/holidayextras/jsonapi-server/blob/master/documentation/resources.md). 

**A resource describes an object and its relationships.**

Here's a simple resource for our todos app.

##### `src/resources/todos.js`

```js
import jsonApi from 'jsonapi-server'
import TodoHandler from '../handlers/TodoHandler'

jsonApi.define({
  resource: 'todos',
  handlers: new TodoHandler(),
  attributes: {
    text: jsonApi.Joi.string(),
    completed: jsonApi.Joi.boolean()
  },
  examples: [
    {
      id: 'e6d4042a-cdd8-4c1c-8558-fa822b58d135',
      type: 'todos',
      text: 'hey',
      completed: false
    },
    {
      id: 'a2194ec8-7dfd-4306-9b4e-030aec49b569',
      type: 'todos',
      text: 'ho',
      completed: false
    },
    {
      id: '6d5fc7c3-15bc-4272-9c3e-125328490266',
      type: 'todos',
      text: 'let\'s go',
      completed: false
    }
  ]
})
```

## Adding a handler
We need to create a handler. As your app matures you'll want to migrate away form the built-in memory handler and use something more permanent. We're going to use the built-in `MemoryHandler` because it makes initial development much simpler... *you don't have to worry about storing data in a DB yet*. For now our handler won't do anything special. The `MemoryHandler` simply stores data in memory. Your data will be deleted every time the server restarts. This is fine for dev but long-term you'll want to explore the persistent data stores available, like [jsonapi-store-relationaldb](https://github.com/holidayextras/jsonapi-store-relationaldb) or [jsonapi-store-mongodb](https://github.com/holidayextras/jsonapi-store-mongodb).

In `src/resources/todos.js` we defined some `examples`. These get loaded into the handler when the app starts so we should always start out with at least those 3 things available.

**A handler manages how data is persisted** in our data store (usually a database). The handler is in charge of reading from the database and writing to the database.

We need to add a handler for our todos resources to use.

##### `src/handlers/TodoHandler.js`

```js
import { MemoryHandler } from 'jsonapi-server'

export default MemoryHandler
```

## Adding `scripts`
We need to add some scripts to our `package.json` to work with our server.

##### `package.json`
```json
// ... snippet from package.json
{
  // ...
  "scripts": {
    "test": "npm run lint:fail -s && npm run build:clean -s && mocha; exit 0",
    "test:dev": "npm run lint:fail -s && npm run build:dev -s && mocha; exit 0",
    "test:watch": "npm run build:clean -s && mocha -w",
    "test:watch:dev": "npm run build:dev -s && mocha -w",
    "test:fail": "mocha",
    "start": "npm run build -s && node ./lib/server.js",
    "start:dev": "npm run lint -s && npm run build:dev -s && DEBUG=jsonApi:validation:*,jsonAPi:requestCounter,jsonApi:handler:* nodemon --watch src ./lib/server.js",
    "lint": "npm run lint:fail -s; exit 0",
    "lint:watch": "esw -w src/**; exit 0",
    "lint:fail": "eslint src/**",
    "clean": "rm -drf lib",
    "build:dev": "npm run clean -s && mkdir lib && echo \"'use strict';\nrequire('babel-register');\nrequire('babel-polyfill');\nmodule.exports = require('../src/server.js').default;\n\" > lib/server.js",
    "build": "npm run lint:fail -s && npm run test:fail -s && babel src --out-dir lib",
    "build:clean": "npm run clean -s && npm run build -s",
    "build:watch": "npm run watch -s",
    "watch": "npm run lint:fail -s && babel src --watch --out-dir lib",
    "prepublish": "npm run build:clean -s"
  },
  // ...
}
```

If you look closely at the `"build:dev"` script in the `package.json` you'll see that for development we actually run our code through the babel transpiler on the fly using [`babel-register`](https://babeljs.io/docs/usage/require/). This can be incredibly slow but should be fast enough for development. It's important to note that it's only "slow" on the first run. Once the server is booted transpiling on the fly isn't really any slower. The benefit for development is that you can work with your fancy new ES2015 code without worrying about creating a new build for every minor change. Having to run a full build just to see your code in development can be painful.

We use the babel transpiled code when we publish to npm, this relies on the `prepublish` script. You can read more about [using ES6 with npm](http://mammal.io/articles/using-es6-today/). For now you don't need to really worry about it at all. The end result is that the `lib/` folder *should never* be committed to git but *should* be published to npm.

## Running our server in development
While we're developing we can work directly on the `src/` files. We use `nodemon` to restart the server for every change and we run `eslint` before starting the server to keep our code clean. You can start your server in development mode with the `npm run start:dev` command.

```bash
npm run start:dev
```

You should see output similar to this:

```
[nodemon] 1.9.1
[nodemon] to restart at any time, enter `rs`
[nodemon] watching: /projects/todos-app/todos-api/src/**/*
[nodemon] starting `node ./lib/server.js`
Server listening on http://localhost:16006/api/v1/
```

Now you should be able to view `http://localhost:16006/api/v1/todos` in your browser and see our example todos.


## Development best practices
Once you get going on developing your server you'll want to make sure you're regularly linting your code and running your tests. Primarily you should be doing this before every commit. But you may also want to run your tests and lint your code *as you develop*. This makes it much easier to catch your mistakes early and can help you get up on the linting rules if you're struggling with the new ES2015 syntax.

Typically this means that you'll want to open multiple tabs in [iTerm2](https://www.iterm2.com/).

- navigate to your `todo-api` folder in your terminal and run `npm run start:dev`
- `⌘t` to open a new new tab and run `npm run lint:watch`
- `⌘t` to open a new new tab and run `npm run test:watch:dev`

Now when you make a change you'll see in real time if your tests pass and your code is properly linted. There are ways to run all of these things as a single command. Fee free to improve the scripts outlined above. Pull requests are welcome.
