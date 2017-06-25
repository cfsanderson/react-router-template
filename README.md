# This is my version of the [React Router Tutorial](https://github.com/reactjs/react-router-tutorial)

The original tutorial was broken up into separate folders with instructions for each section. I have compiled all of the notes from that original tutorial into the README below. This project will be the result of following along with the tutorial and making my own mods along the way.

---

# #1 Setting up the Project

First you'll need Node.js and the package manager that comes with it: npm.

Once you've got that working, head to the command line where we'll set up our project.

Clone the Tutorial

git clone https://github.com/reactjs/react-router-tutorial
cd react-router-tutorial
cd lessons/01-setting-up
npm install
npm start
Now open up http://localhost:8080

Feel free to poke around the code to see how we're using webpack and npm scripts to run the app.

You should see a "Hello React Router" message in the browser.

Make Some Changes

Open up modules/App.js and change the text to something like "Hello ". The browser automatically reloads with your new code.

# #2 Rendering a Route

At its heart, React Router is a component.

```js
render(<Router/>, document.getElementById('app'))
```

That's not going to display anything until we configure a route.

Open up `index.js` and

1. import `Router`, `Route`, and `hashHistory`
2. render a `Router` instead of `App`

```js
// ...
import { Router, Route, hashHistory } from 'react-router'

render((
  <Router history={hashHistory}>
    <Route path="/" component={App}/>
  </Router>
), document.getElementById('app'))
```

Make sure your server is running with `npm start` and then visit
[http://localhost:8080](http://localhost:8080)

You should get the same screen as before, but this time with some junk
in the URL. We're using `hashHistory`--it manages the routing history
with the hash portion of the url. It's got that extra junk to shim some
behavior the browser has natively when using real urls.  We'll change
this to use real urls later and lose the junk, but for now, this works
great because it doesn't require any server-side configuration.

## Adding More Screens

Create two new components at:

- `modules/About.js`
- `modules/Repos.js`

```js
// modules/About.js
import React from 'react'

export default React.createClass({
  render() {
    return <div>About</div>
  }
})
```

```js
// modules/Repos.js
import React from 'react'

export default React.createClass({
  render() {
    return <div>Repos</div>
  }
})
```

Now we can couple them to the app at their respective paths.

```js
// insert into index.js
import About from './modules/About'
import Repos from './modules/Repos'

render((
  <Router history={hashHistory}>
    <Route path="/" component={App}/>
    {/* add the routes here */}
    <Route path="/repos" component={Repos}/>
    <Route path="/about" component={About}/>
  </Router>
), document.getElementById('app'))
```

Now visit [http://localhost:8080/#/about](http://localhost:8080/#/about) and
[http://localhost:8080/#/repos](http://localhost:8080/#/repos)

---

# #3 Navigating with Link

Perhaps the most used component in your app is `Link`. It's almost
identical to the `<a/>` tag you're used to except that it's aware of
the `Router` it was rendered in.

Let's create some navigation in our `App` component.

```js
// modules/App.js
import React from 'react'
import { Link } from 'react-router'

export default React.createClass({
  render() {
    return (
      <div>
        <h1>React Router Tutorial</h1>
        <ul role="nav">
          <li><Link to="/about">About</Link></li>
          <li><Link to="/repos">Repos</Link></li>
        </ul>
      </div>
    )
  }
})
```

Now visit [http://localhost:8080](http://localhost:8080) and click the links, click back, click
forward. It works!

---

# #4 Nested Routes

The navigation we added to `App` should probably be present on every
screen. Without React Router, we could wrap that `ul` into a
component, say `Nav`, and render a `Nav` on every one of our screens.

This approach isn't as clean as the application grows. React Router
provides another way to share UI like this with nested routes, a trick
it learned from [Ember](http://emberjs.com) (/me tips hat).

## Nested UI and Nested URLs

Have you ever noticed your app is just a series of boxes inside boxes
inside boxes? Have you also noticed your URLs tend to be coupled to that
nesting? For example given this url, `/repos/123`, our
components would probably look like this:

```js
<App>       {/*  /          */}
  <Repos>   {/*  /repos     */}
    <Repo/> {/*  /repos/123 */}
  </Repos>
</App>
```

And our UI something like:

```
         +-------------------------------------+
         | Home Repos About                    | <- App
         +------+------------------------------+
         |      |                              |
Repos -> | repo |  Repo 1                      |
         |      |                              |
         | repo |  Boxes inside boxes          |
         |      |  inside boxes ...            | <- Repo
         | repo |                              |
         |      |                              |
         | repo |                              |
         |      |                              |
         +------+------------------------------+
```

React Router embraces this by letting you nest your routes, which
automatically becomes nested UI.

## Sharing Our Navigation

Let's nest our `About` and `Repos` components inside of `App` so that we
can share the navigation with all screens in the app. We do it in two
steps:

First, let the `App` `Route` have children, and move the other routes
underneath it.

```js
// index.js
// ...
render((
  <Router history={hashHistory}>
    <Route path="/" component={App}>
      {/* make them children of `App` */}
      <Route path="/repos" component={Repos}/>
      <Route path="/about" component={About}/>
    </Route>
  </Router>
), document.getElementById('app'))
```

Next, render children inside of `App`.

```js
// modules/App.js
// ...
  render() {
    return (
      <div>
        <h1>React Router Tutorial</h1>
        <ul role="nav">
          <li><Link to="/about">About</Link></li>
          <li><Link to="/repos">Repos</Link></li>
        </ul>

        {/* add this */}
        {this.props.children}

      </div>
    )
  }
// ...
```

Alright, now go click the links and notice that the `App` component
continues to render while the child route's component gets swapped
around as `this.props.children` :)

React Router is constructing your UI like this:

```js
// at /about
<App>
  <About/>
</App>

// at /repos
<App>
  <Repos/>
</App>
```

## By Small and Simple Things are Great Things Brought to Pass

The best way to build large things is to stitch small things together.

This is the real power of React Router, every route can be developed
(even rendered!) as an independent application. Your route configuration
stitches all these apps together however you'd like.  Applications
inside of Applications, boxes inside of boxes.

What happens if you move the `About` route outside of `App`?

Okay, now put it back.

---

# #5 Active Links

One way that `Link` is different from `a` is that it knows if the path
it links to is active so you can style it differently.

## Active Styles

Let's see how it looks with inline styles, add `activeStyle` to your
`Link`s.

```js
// modules/App.js
<li><Link to="/about" activeStyle={{ color: 'red' }}>About</Link></li>
<li><Link to="/repos" activeStyle={{ color: 'red' }}>Repos</Link></li>
```

Now as you navigate, the active link is red.

## Active Class Name

You can also use an active class name instead of inline-styles.

```js
// modules/App.js
<li><Link to="/about" activeClassName="active">About</Link></li>
<li><Link to="/repos" activeClassName="active">Repos</Link></li>
```

We don't have a stylesheet on the page yet though. Lets add one-extra
point if you can add a `link` tag from memory.

```html
// index.html
<link rel="stylesheet" href="index.css" />
```

And the CSS file:

```css
.active {
  color: green;
}
```

You'll need to manually refresh the browser since Webpack isn't building
our `index.html`.

## Nav Link Wrappers

Most links in your site don't need to know they are active, usually just
primary navigation links need to know. It's useful to wrap those so you
don't have to remember what your `activeClassName` or `activeStyle` is
everywhere.

We will use a spread operator here, the three dots. It clones our props
and in this use case it clones `activeClassName` to our desired component for
us to benefit from.

Create a new file at `modules/NavLink.js` that looks like this:

```js
// modules/NavLink.js
import React from 'react'
import { Link } from 'react-router'

export default React.createClass({
  render() {
    return <Link {...this.props} activeClassName="active"/>
  }
})
```

Now you can go change your links to `NavLink`s.

```js
// modules/App.js
import NavLink from './NavLink'

// ...

<li><NavLink to="/about">About</NavLink></li>
<li><NavLink to="/repos">Repos</NavLink></li>
```

Oh, how beautiful upon the renders is the composability of components.

---

# #6 URL Params

Consider the following URLs:

```
/repos/reactjs/react-router
/repos/facebook/react
```

These URLs would match a route path like this:

```
/repos/:userName/:repoName
```

The parts that start with `:` are URL parameters whose values will be
parsed out and made available to route components on
`this.props.params[name]`.

## Adding a Route with Parameters

Let's teach our app how to render screens at `/repos/:userName/:repoName`.

First we need a component to render at the route, make a new file at
`modules/Repo.js` that looks something like this:

```js
// modules/Repo.js
import React from 'react'

export default React.createClass({
  render() {
    return (
      <div>
        <h2>{this.props.params.repoName}</h2>
      </div>
    )
  }
})
```

Now open up `index.js` and add the new route.

```js
// ...
// import Repo
import Repo from './modules/Repo'

render((
  <Router history={hashHistory}>
    <Route path="/" component={App}>
      <Route path="/repos" component={Repos}/>
      {/* add the new route */}
      <Route path="/repos/:userName/:repoName" component={Repo}/>
      <Route path="/about" component={About}/>
    </Route>
  </Router>
), document.getElementById('app'))
```

Now we can add some links to this new route in `Repos.js`.

```js
// Repos.js
import { Link } from 'react-router'
// ...
export default React.createClass({
  render() {
    return (
      <div>
        <h2>Repos</h2>

        {/* add some links */}
        <ul>
          <li><Link to="/repos/reactjs/react-router">React Router</Link></li>
          <li><Link to="/repos/facebook/react">React</Link></li>
        </ul>

      </div>
    )
  }
})
```

Now go test your links out. Note that the parameter name in the route
`path` becomes the property name in the component. Both `repoName` and
`userName` are available on `this.props.params` of your component. You
should probably add some prop types to help others and yourself out
later.

---

# #7 More Nesting

Notice how the list of links to different repositories goes away when we
navigate to a repository? What if we want the list to persist, just like
the global navigation persists?

Try to figure that out before reading on.

...

First, nest the `Repo` route under the `Repos` route. Then go render
`this.props.children` in `Repos`.

```js
// index.js
// ...
<Route path="/repos" component={Repos}>
  <Route path="/repos/:userName/:repoName" component={Repo}/>
</Route>
```

```js
// Repos.js
// ...
<div>
  <h2>Repos</h2>
  <ul>
    <li><Link to="/repos/reactjs/react-router">React Router</Link></li>
    <li><Link to="/repos/facebook/react">React</Link></li>
  </ul>
  {/* will render `Repo.js` when at /repos/:userName/:repoName */}
  {this.props.children}
</div>
```

## Active Links

Let's bring in our `NavLink` from before so we can add the `active`
class name to these links:

```js
// modules/Repos.js
// import it
import NavLink from './NavLink'

// ...
<li><NavLink to="/repos/reactjs/react-router">React Router</NavLink></li>
<li><NavLink to="/repos/facebook/react">React</NavLink></li>
// ...
```

Notice how both the `/repos` link up top and the individual repo links are
both active? When child routes are active, so are the parents.

---

# #8 Index Routes

When we visit `/` in this app it's just our navigation and a blank page.
We'd like to render a `Home` component there. Lets create a `Home`
component and then talk about how to render it at `/`.

```js
// modules/Home.js
import React from 'react'

export default React.createClass({
  render() {
    return <div>Home</div>
  }
})
```

One option is to see if we have any children in `App`, and if not,
render `Home`:

```js
// modules/App.js
import Home from './Home'

// ...
<div>
  {/* ... */}
  {this.props.children || <Home/>}
</div>
//...
```

This would work fine, but its likely we'll want `Home` to be attached to
a route like `About` and `Repos` in the future. A few reasons include:

1. Participating in a data fetching abstraction that relies on matched
   routes and their components.
2. Participating in `onEnter` hooks
3. Participating in code-splitting

Also, it just feels good to keep `App` decoupled from `Home` and let the
route config decide what to render as the children. Remember, we want to
build small apps inside small apps, not big ones!

Let's add a new route to `index.js`.

```js
// index.js
// new imports:
// add `IndexRoute` to 'react-router' imports
import { Router, Route, hashHistory, IndexRoute } from 'react-router'
// and the Home component
import Home from './modules/Home'

// ...

render((
  <Router history={hashHistory}>
    <Route path="/" component={App}>

      {/* add it here, as a child of `/` */}
      <IndexRoute component={Home}/>

      <Route path="/repos" component={Repos}>
        <Route path="/repos/:userName/:repoName" component={Repo}/>
      </Route>
      <Route path="/about" component={About}/>
    </Route>
  </Router>
), document.getElementById('app'))
```

Now open [http://localhost:8080](http://localhost:8080) and you'll see the new component is
rendered.

Notice how the `IndexRoute` has no path. It becomes
`this.props.children` of the parent when no other child of the parent
matches, or in other words, when the parent's route matches exactly.

Index routes can twist people's brains up sometimes. Hopefully it will
sink in with a bit more time. Just think about a web server that looks
for `index.html` when you're at `/`. Same idea, React Router looks for
an index route if a route's path matches exactly.

---

# #9 Index Links

Have you noticed in our app that we don't have any navigation to get
back to rendering the `Home` component?

Lets add a link to `/` and see what happens:

```js
// in App.js
// ...
<li><NavLink to="/">Home</NavLink></li>
// ...
```

Now navigate around. Notice anything weird? The link to `Home` is always
active! As we learned earlier, parent routes are active when child routes
are active. Unfortunately, `/` is the parent of everything.

For this link, we want it to only be active when the index route is
active. There are two ways to let the router know you're linking to the
"index route" so it only adds the active class (or styles) when the
index route is rendered.

## IndexLink

First let's use the `IndexLink` instead of `NavLink`

```js
// App.js
import { IndexLink } from 'react-router'

// ...
<li><IndexLink to="/" activeClassName="active">Home</IndexLink></li>
```

Fixed! Now this link is only "active" when we're at the index route. Go
ahead and click around to see.

## `onlyActiveOnIndex` Property

We can use `Link` as well by passing it the `onlyActiveOnIndex` prop
(`IndexLink` just wraps `Link` with this property for convenience).

```js
<li><Link to="/" activeClassName="active" onlyActiveOnIndex={true}>Home</Link></li>
```

That's fine, but we already abstracted away having to know what the
`activeClassName` is with `Nav`.

Remember, in `NavLink` we're passing along all of our props to `Link` with
the `{...spread}` syntax, so we can actually add the prop when we render
a `NavLink` and it will make its way down to the `Link`:

```js
<li><NavLink to="/" onlyActiveOnIndex={true}>Home</NavLink></li>
```

---

# #10 Clean URLs with Browser History

The URLs in our app right now are built on a hack: the hash. It's the
default because it will always work, but there's a better way.

Modern browsers let JavaScript manipulate the URL without making an http
request, so we don't need to rely on the hash (`#`) portion of the url
to do routing, but there's a catch (we'll get to it later).

## Configuring Browser History

Open up `index.js` and import `browserHistory` instead of `hashHistory`.

```js
// index.js
// ...
// bring in `browserHistory` instead of `hashHistory`
import { Router, Route, browserHistory, IndexRoute } from 'react-router'

render((
  <Router history={browserHistory}>
    {/* ... */}
  </Router>
), document.getElementById('app'))
```

Now go click around and admire your clean URLs.

Oh yeah, the catch. Click on a link and then refresh your browser. What
happens?

```
Cannot GET /repos
```

## Configuring Your Server

Your server needs to deliver your app no matter what URL comes in,
because your app, in the browser, is manipulating the URL. Our current
server doesn't know how to handle the URL.

The Webpack Dev Server has an option to enable this. Open up
`package.json` and add `--history-api-fallback`.

```json
    "start": "webpack-dev-server --inline --content-base . --history-api-fallback"
```

We also need to change our relative paths to absolute paths in
`index.html` since the URLs will be at deep paths and the app, if it
starts at a deep path, won't be able to find the files.

```html
<!-- index.html -->
<!-- index.css -> /index.css -->
<link rel="stylesheet" href="/index.css">

<!-- bundle.js -> /bundle.js -->
<script src="/bundle.js"></script>
```

Stop your server if it's running, then `npm start` again. Look at those
clean URLs :)

---

# #11 Production-ish Server

None of this has anything to do with React Router, but since we're
talking about web servers, we might as well take it one step closer to
the real-world. We'll also need it for server rendering in the next
section.

Webpack dev server is not a production server. Let's make a production
server and a little environment-aware script to boot up the right server
depending on the environment.

Let's install a couple modules:

```
npm install express if-env compression --save
```

First, we'll use the handy `if-env` in `package.json`.  Update your
scripts entry in package.json to look like this:

```json
// package.json
"scripts": {
  "start": "if-env NODE_ENV=production && npm run start:prod || npm run start:dev",
  "start:dev": "webpack-dev-server --inline --content-base . --history-api-fallback",
  "start:prod": "webpack && node server.js"
},
```

In the root directly, go open up `webpack.config.js` and add the publicPath '/' as per below:
```
// webpack.config.js
  output: {
    path: 'public',
    filename: 'bundle.js',
    publicPath: '/'
  },
```

When you run `npm start` it checks if the value of our `NODE_ENV` environment variable is
`production`. If yes, it runs `npm run start:prod`, if not, it runs
`npm run start:dev`.

Now we're ready to create a production server with Express and add a new file at root dir. Here's a
first attempt:

```js
// server.js
var express = require('express')
var path = require('path')

var app = express()

// serve our static stuff like index.css
app.use(express.static(__dirname))

// send all requests to index.html so browserHistory in React Router works
app.get('*', function (req, res) {
  res.sendFile(path.join(__dirname, 'index.html'))
})

var PORT = process.env.PORT || 8080
app.listen(PORT, function() {
  console.log('Production Express server running at localhost:' + PORT)
})
```

Now run:

```sh
NODE_ENV=production npm start
# For Windows users:
# SET "NODE_ENV=production" && npm start
```

Congratulations! You now have a production server for this app. After
clicking around, try navigating to [http://localhost:8080/package.json](http://localhost:8080/package.json).
Whoops.  Let's fix that. We're going to shuffle around a couple files and
update some paths scattered across the app.

1. make a `public` directory.
2. Move `index.html` and `index.css` into it.

Now let's update `server.js` to point to the right directory for static
assets:

```js
// server.js
// ...
// add path.join here
app.use(express.static(path.join(__dirname, 'public')))

// ...
app.get('*', function (req, res) {
  // and drop 'public' in the middle of here
  res.sendFile(path.join(__dirname, 'public', 'index.html'))
})
```

We also need to tell webpack to build to this new directory:

```js
// webpack.config.js
// ...
output: {
  path: 'public',
  // ...
}
```

And finally (!) add it to the `--content-base` argument to `npm run start:dev` script:

```json
"start:dev": "webpack-dev-server --inline --content-base public --history-api-fallback",
```

If we had the time in this tutorial, we could use the `WebpackDevServer`
API in a JavaScript file instead of the CLI in an npm script and then
turn this path into config shared across all of these files. But, we're
already on a tangent, so that will have to wait for another time.

Okay, now that we aren't serving up the root of our project as public
files, let's add some code minification to Webpack and gzipping to
express.

```js
// webpack.config.js

// make sure to import this
var webpack = require('webpack')

module.exports = {
  // ...

  // add this handful of plugins that optimize the build
  // when we're in production
  plugins: process.env.NODE_ENV === 'production' ? [
    new webpack.optimize.DedupePlugin(),
    new webpack.optimize.OccurrenceOrderPlugin(),
    new webpack.optimize.UglifyJsPlugin()
  ] : [],

  // ...
}
```

And compression in express:

```js
// server.js
// ...
var compression = require('compression')

var app = express()
// must be first!
app.use(compression())
```

Now go start your server in production mode:

```
NODE_ENV=production npm start
```

You'll see some UglifyJS logging and then in the browser, you can see
the assets are being served with gzip compression.

---

# #12 Navigating Programatically

While most navigation happens with `Link`, you can programmatically
navigate around an application in response to form submissions, button
clicks, etc.

Let's make a little form in `Repos` that programmatically navigates.

```js
// modules/Repos.js
import React from 'react'
import NavLink from './NavLink'

export default React.createClass({

  // add this method
  handleSubmit(event) {
    event.preventDefault()
    const userName = event.target.elements[0].value
    const repo = event.target.elements[1].value
    const path = `/repos/${userName}/${repo}`
    console.log(path)
  },

  render() {
    return (
      <div>
        <h2>Repos</h2>
        <ul>
          <li><NavLink to="/repos/reactjs/react-router">React Router</NavLink></li>
          <li><NavLink to="/repos/facebook/react">React</NavLink></li>
          {/* add this form */}
          <li>
            <form onSubmit={this.handleSubmit}>
              <input type="text" placeholder="userName"/> / {' '}
              <input type="text" placeholder="repo"/>{' '}
              <button type="submit">Go</button>
            </form>
          </li>
        </ul>
        {this.props.children}
      </div>
    )
  }
})
```

There are two ways you can do this, the first is simpler than the
second.

First we can use the `browserHistory` singleton that we passed into
`Router` in `index.js` and push a new URL into the history.

```js
// modules/Repos.js
import { browserHistory } from 'react-router'

// ...
  handleSubmit(event) {
    // ...
    const path = `/repos/${userName}/${repo}`
    browserHistory.push(path)
  },
// ...
```

There's a potential problem with this though. If you pass a different
history to `Router` than you use here, it won't work. It's not very
common to use anything other than `browserHistory`, so this is
acceptable practice. If you're concerned about it, you can make a module
that exports the history you want to use across the app, or...

You can also use the `router` that `Router` provides on "context".
First, you ask for context in the component, and then you can use it:

```js
export default React.createClass({

  // ask for `router` from context
  contextTypes: {
    router: React.PropTypes.object
  },

  // ...

  handleSubmit(event) {
    // ...
    this.context.router.push(path)
  },

  // ..
})
```

This way you'll be sure to be pushing to whatever history gets passed to
`Router`. It also makes testing a bit easier since you can more easily
stub context than singletons.

---

# #13 Server Rendering

Alright, first things first. Server rendering, at its core is a simple
concept in React.

```js
render(<App/>, domNode)
// can be rendered on the server as
const markup = renderToString(<App/>)
```

It's not rocket science, but it also isn't trivial. First I'm going to
just throw a bunch of webpack shenanigans at you with little
explanation, then we'll talk about the Router.

Since node doesn't (and shouldn't) understand JSX, we need to compile
the code somehow. Using something like `babel/register` is not fit for
production use, so we'll use webpack to build a server bundle, just like
we use it to build a client bundle.

Make a new file called `webpack.server.config.js` and put this stuff in
there:

```js
var fs = require('fs')
var path = require('path')

module.exports = {

  entry: path.resolve(__dirname, 'server.js'),

  output: {
    filename: 'server.bundle.js'
  },

  target: 'node',

  // keep node_module paths out of the bundle
  externals: fs.readdirSync(path.resolve(__dirname, 'node_modules')).concat([
    'react-dom/server', 'react/addons',
  ]).reduce(function (ext, mod) {
    ext[mod] = 'commonjs ' + mod
    return ext
  }, {}),

  node: {
    __filename: true,
    __dirname: true
  },

  module: {
    loaders: [
      { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader?presets[]=es2015&presets[]=react' }
    ]
  }

}
```

Hopefully some of that makes sense, we aren't going to cover what all of
that stuff does, it's sufficient to say that now we can run our
`server.js` file through webpack and then run it.

Now we need to make some scripts to build server bundle before we try to
run our app.  Update your `package.json` script config to look like
this:

```
"scripts": {
  "start": "if-env NODE_ENV=production && npm run start:prod || npm run start:dev",
  "start:dev": "webpack-dev-server --inline --content-base public/ --history-api-fallback",
  "start:prod": "npm run build && node server.bundle.js",
  "build:client": "webpack",
  "build:server": "webpack --config webpack.server.config.js",
  "build": "npm run build:client && npm run build:server"
},
```

Now when we run `NODE_ENV=production npm start` both the client and
server bundles get created by Webpack.

Okay, let's talk about the Router. We're going to need our routes split
out into a module so that both the client and server entries can require
it. Make a file at `modules/routes` and move your routes and components
into it.

```js
// modules/routes.js
import React from 'react'
import { Route, IndexRoute } from 'react-router'
import App from './App'
import About from './About'
import Repos from './Repos'
import Repo from './Repo'
import Home from './Home'

module.exports = (
  <Route path="/" component={App}>
    <IndexRoute component={Home}/>
    <Route path="/repos" component={Repos}>
      <Route path="/repos/:userName/:repoName" component={Repo}/>
    </Route>
    <Route path="/about" component={About}/>
  </Route>
)
```

```js
// index.js
import React from 'react'
import { render } from 'react-dom'
import { Router, browserHistory } from 'react-router'
// import routes and pass them into <Router/>
import routes from './modules/routes'

render(
  <Router routes={routes} history={browserHistory}/>,
  document.getElementById('app')
)
```

Now open up `server.js`. We're going to bring in two modules from React
Router to help us render on the server.

If we tried to render a `<Router/>` on the server like we do in the
client, we'd get an empty screen since server rendering is synchronous
and route matching is asynchronous.

Also, most apps will want to use the router to help them load data, so
asynchronous routes or not, you'll want to know what screens are going
to render before you actually render so you can use that information to
load asynchronous data before rendering. We don't have any data loading
in this app, but you'll see where it could happen.

First we import `match` and `RouterContext` from react router, then
we'll match the routes to the url, and finally render.

```js
// ...
// import some new stuff
import React from 'react'
// we'll use this to render our app to an html string
import { renderToString } from 'react-dom/server'
// and these to match the url to routes and then render
import { match, RouterContext } from 'react-router'
import routes from './modules/routes'

// ...

// send all requests to index.html so browserHistory works

app.get('*', (req, res) => {
  // match the routes to the url
  match({ routes: routes, location: req.url }, (err, redirect, props) => {
    // `RouterContext` is what the `Router` renders. `Router` keeps these
    // `props` in its state as it listens to `browserHistory`. But on the
    // server our app is stateless, so we need to use `match` to
    // get these props before rendering.
    const appHtml = renderToString(<RouterContext {...props}/>)

    // dump the HTML into a template, lots of ways to do this, but none are
    // really influenced by React Router, so we're just using a little
    // function, `renderPage`
    res.send(renderPage(appHtml))
  })
})

function renderPage(appHtml) {
  return `
    <!doctype html public="storage">
    <html>
    <meta charset=utf-8/>
    <title>My First React Router App</title>
    <link rel=stylesheet href=/index.css>
    <div id=app>${appHtml}</div>
    <script src="/bundle.js"></script>
   `
}

var PORT = process.env.PORT || 8080
app.listen(PORT, function() {
  console.log('Production Express server running at localhost:' + PORT)
})
```

And that's it. Now if you run `NODE_ENV=production npm start` and visit
the app, you can view source and see that the server is sending down our
app to the browser. As you click around, you'll notice the client app
has taken over and doesn't make requests to the server for UI. Pretty
cool yeah?!


Our callback to match is a little naive, here's what a production
version would look like:

```js
app.get('*', (req, res) => {
  match({ routes: routes, location: req.url }, (err, redirect, props) => {
    // in here we can make some decisions all at once
    if (err) {
      // there was an error somewhere during route matching
      res.status(500).send(err.message)
    } else if (redirect) {
      // we haven't talked about `onEnter` hooks on routes, but before a
      // route is entered, it can redirect. Here we handle on the server.
      res.redirect(redirect.pathname + redirect.search)
    } else if (props) {
      // if we got props then we matched a route and can render
      const appHtml = renderToString(<RouterContext {...props}/>)
      res.send(renderPage(appHtml))
    } else {
      // no errors, no redirect, we just didn't match anything
      res.status(404).send('Not Found')
    }
  })
})
```

Server rendering is really new. There aren't really "best practices"
yet, especially when it comes to data loading, so this tutorial is done,
dropping you off at the bleeding edge.

---

# What's Next?

Thanks for sticking with this tutorial all the way to the end!

If you want to learn more:

1. Browse and run the [examples](https://github.com/reactjs/react-router/tree/latest/examples)
- Read the [docs](https://github.com/reactjs/react-router/tree/latest/docs)
- Read the source.

Happy routing!
