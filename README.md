# lucid-router
A simple (lucid) html5 history-aware router.

- ideal for universal/isomorphic apps
- tiny (adds less than 5kb to your gzipped bundle)
- progressive enhancement
  - fall back on normal redirect behavior when the history API isn't supported, not hash routing
  - routes can be added and removed at any time, so load the heavier sections of your app later when the user is idle -- if they navigate to it before the bundle loads it'll just redirect like normal

More interested in the philosophy than the configuration?  [It's at the bottom.](https://github.com/spicydonuts/lucid-router#philosophy)

Want to dive straight into some examples?
- [Here's a super basic one](https://github.com/spicydonuts/lucid-router/tree/master/examples/basics)
- [And here's one using React](https://github.com/spicydonuts/lucid-router/tree/master/examples/react)


## Configuration
Require the router anywhere it will be used and add routes.  Routes are stored inside the `lucid-router` module, so it is safe to destructure or pass the router's functions around without any binding (see below).
```js
import * as router from 'lucid-router'

router.addRoutes([
  {name: 'profile', path: '/profiles/:profileId'}, // required param
  {name: 'home',    path: '(/:category)'}          // optional param
])
```
(Note: `* as router` is required because `lucid-router` does not have a default export.  Most of the time you will only import the parts your module actually needs -- see the [ES2015 section](https://github.com/spicydonuts/lucid-router/blob/master/README.md#es2015-module-imports) below)

You can also set an `external` property on the route definition.  This flag can be a boolean or a function which takes an object containing the route matching info and returns a boolean.  `false` will allow the normal route transition to occur, while `true` will cancel the location change event and instead navigate the browser directly to the url for a full page load.  This setting is mainly for hybrid classic/SPA apps.
```js
router.addRoutes([
  {
    name:     'elsewhere',
    path:     '/non-spa-route',
    external: true | false | (matchInfo => true | false)
  }
])
```

Also note that routes can be added or removed (see `router.removeRoute(name)`) at any time, allowing different parts of a larger application to asynchronously inject SPA features into an app as it loads.  If a navigation occurs before the necessary route exists, a normal browser redirect will occur.

`getLocation` returns a `RouterLocation` object, or `null` if no routes match.  See the [Flow types](https://github.com/spicydonuts/lucid-router/blob/master/lib/lucid-router.t.js) for more info.
Calling `getLocation` with no parameters will look for a `window` object to pull location info from.  If `window` isn't available (Node.js server) or you need location data for a route you are not currently on, pass the path to match on into `getLocation`.
```js
router.getLocation()
router.getLocation(path)
```


## Subscribing to location changes
This is where the magic happens.  Use this function to call `React.render` or wrap it with a Flux store so your components can subscribe to changes.  Better yet, reserve a spot for it in your global app state and make your whole app a function from `state -> UI`!
```js
router.register(location => console.log(location))
```

Callbacks can be registered and un-registered at any time, during an app's lifecycle (think dynamically loaded sections of a large app).  Unregister by calling the callback returned from `router.register`.


## Navigating
Use `navigate` to perform an immediate transition to a given url (a string).  Use `navigatorFor` to build a callback bound to a particular path (equivalent to `e => navigate(path, e)`):
```js
router.navigate('/path', e)     // => pass the event if you want it cancelled for you
router.navigatorFor('/path')(e) // => same as above, curried style (useful for event binding)
```

Use `navigateToRoute` to navigate using a route name and params object:
```js
router.navigateToRoute('profile', {profileId: 5}, e)
router.navigatorForRoute('profile', {profileId: 5})(e)
```

This functionality is also available without performing a navigation:
```js
router.pathFor('profile', {profileId: 5}) // => returns '/profiles/5'
```


## ES2015 module imports
This is safe and convenient:
```js
import {pathFor, navigatorFor} from 'lucid-router'

const link = pathFor('route-name')
const navigator = navigatorFor(link)
```


## React
Nothing about `lucid-router` is specific to React, but they make a great pair!  I've included a helper component for building anchor tags which you can import and use like so:
```js
import Link from 'lucid-router/link'

class Nav extends React.Component {
  render() {
    return (
      <nav>
        <Link to="my-route" params={{id: 9}}>My Route</Link>
        <Link href="/somewhere/else">Somewhere else</Link>
      </nav>
    )
  }
}
```
The first becomes an `<a>` with an `href` and `onClick` which defer to `pathFor` and `navigatorForRoute`.  The second just sets the `href` with the value provided and calls `navigate` when clicked.  `Link` is just a shortcut for the most common use cases, so you can use it, ignore it, or make your own!


## Philosophy
Another router??  Yup.  I see two problems with the status quo.
Routers are too specialized and they're treated as some separate, almost magical part of getting a 'real' application built.

Too specialized?  What's that mean?
- Angular Router
- Angular UI Router
- Ember Router
- React Router
- Express Router
- Koa Router
- etc...

Every framework has at least one router dedicated to _just_ that library, plus a few more from the community.
I wanted a tool for abstracting the details from routing away from my app, not a tool that depends on the specific library I've chosen to build my app.

The second problem is a little harder to quantify, and it comes from the Rails/Django/MVC patterns of views and layouts.
This system tends to create a separation of _technology_ rather than a separation of _concerns_.
A folder for every controller?  A folder for every view?
No wonder those folders got too big and we needed to abstract common layouts files...

What if we organized it by concern: a folder for accounts, a folder for orders, etc.
Then we have a central "app" that decides which of these to sub-apps to hand off to at any given time.
What if that central hub of your app didn't have to care about routing logic?
At first glance, this may not appear to give you much benefit.
You'll still have code _somewhere_ in your app to decide what to render given the current location information.

The difference is that your application now controls this logic, not a router.
You can render each screen or state of your app without a router.
You can tramsform urls into snapshots of state without an app.
You can render the same way on the server _and_ the client.
This is the power of abstraction -- simplicity and composability.
The "single tool for everything" solutions above sure look powerful at first glance, but in the long run they lock you in to their rules and quirks.

Think of it this way: if you design your app on my router and then decide it sucks, you can replace it with another one or write your own, provided you can convert the replacement's output to simple data.

: ]


## Thanks
Thanks to [url-pattern](https://github.com/snd/url-pattern) for all of the route/pattern work!


## Misc
What about `router5`?

It looks great!  There are three reasons I wrote this router anyway:

1. I didn't know `router5` existed when I initially wrote this for a work app, but even if I had...
2. `router5` is more complicated.  I love keeping things simple.
3. I needed really smooth fallback on regular redirects when html5 history isn't supported, not hash routing fallbacks.  `lucid-router` can also be used on non-SPA apps just fine without losing the routing info.. just mark all your routes as `external`!
