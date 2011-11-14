# Director

# Overview
Director is a router. Routing is the process of determining what code to run when a URL is requested. Director works on the client and the server. Director is dependency free, on the client it does not require any other libraries (such as jQuery).

* [Client-Side Routing](#client-side)
* [Server-Side HTTP Routing](#http-routing)
* [Server-Side CLI Routing](#cli-routing)
* [API Documentation](#api-documentation)
* [Frequently Asked Questions](#faq)

<a name="client-side"></a>
## Client-side Routing
It simply watches the hash of the URL to determine what to do, for example:

```
http://foo.com/#/bar
```

Client-side routing (aka hash-routing) allows you to specify some information about the state of the application using the URL. So that when the user visits a specific URL, the application can be transformed accordingly. Because of the additional requirements of Client-side routing `director` exposes additional API methods, see the [Client-side documentation](https://github.com/flatiron/director/tree/master/docs/client-side.md) for more details.

Here is a simple example:

```html
  <!html>
  <html>
    <head>
      <script src="/director.js"></script>
      <script>

        var author = function () { /* ... */ },
            books = function () { /* ... */ };

        var routes = {
          '/author': showAuthorInfo,
          '/books': [showAuthorInfo, listBooks]
        };

        var router = Router(routes);

      </script>
    </head>
    <body>
    </body>
  </html>
```

Director works great with your favorite DOM library, such as jQuery.

```html
  <!html>
  <html>
    <head>
      <script src="/director.js"></script>
      <script>

        // 
        // create some functions to be executed when
        // the correct route is issued by the user.
        //
        var author = function () { /* ... */ },
            books = function () { /* ... */ },
            allroutes = function(route) {
              var sections = $('section');
              sections.hide();
              sections.find('data-route[' + route + ']').show();
            };

        //
        // define the routing table.
        //
        var routes = {
          '/author': showAuthorInfo,
          '/books': [showAuthorInfo, listBooks]
        };

        //
        // instantiate the router.
        //
        var router = Router(routes);
      
        //
        // a global configuration setting.
        //
        router.configure({
          on: allroutes
        });

      </script>
    </head>
    <body>
      <section data-route="author">Author Name</section>
      <section data-route="books">Book1, Book2, Book3</section>
    </body>
  </html>
```

You can find a browser-specific build of `director` [here][0] which has all of the server code stripped away.

<a name="http-routing"></a>
## Server-Side HTTP Routing

Director handles routing for HTTP requests similar to `journey` or `express`: 

```js
  //
  // require the native http module, as well as director.
  //
  var http = require('http'),
      director = require('director');

  //
  // create some logic to be routed to.
  //
  function helloWorld(route) {
    this.res.writeHead(200, { 'Content-Type': 'text/plain' })
    this.res.end('hello world from (' + route + ')');
  }

  //
  // define a routing table.
  //
  var router = new director.http.Router({
    '/hello': {
      get: helloWorld
    }
  });

  //
  // stup a server and when there is a request, dispatch the
  // route that was requestd in the request object.
  //
  var server = http.createServer(function (req, res) {
    router.dispatch(req, res, function (err) {
      if (err) {
        res.writeHead(404);
        res.end();
      }
    });
  });

  //
  // You can also do ad-hoc routing, similar to `journey` or `express`.
  // This can be done with a string or a regexp.
  //
  router.get('/bonjour', helloWorld);
  router.get(/hola/, helloWorld);

  //
  // set the server to listen on port `8080`.
  //
  server.listen(8080);
```

<a name="cli-routing"></a>
## CLI Routing

Director supports Command Line Interface routing. Routes for cli options are based on command line input (i.e. `process.argv`) instead of a URL.

``` js
  var director = require('director');
  
  var router = new director.cli.Router();
  
  router.on('create', function () {
    console.log('create something');
  });
  
  router.on(/destroy/, function () {
    console.log('destroy something');
  });
```

<a name="api-documentation"></a>
# API Documentation

* [Constructor](#constructor)
* [Routing Table](#routing-table)
* [Adhoc Routing](#adhoc-routing)
* [Configuration](#configuration)
* [URL Matching](#url-matching)
* [URL Params](#url-params)
* [Async Routing](#async-routing)
* [Route Recursion](#route-recursion)
* [Instance Methods](#instance-methods)

<a name="constructor"></a>
## Constructor

``` js
  var router = Router(routes);
```

<a name="routing-table"></a>
### Routing Table

An object literal that contains nested route definitions. A potentially nested set of key/value pairs. The keys in the object literal represent each potential part of the URL. The values in the object literal contain references to the functions that should be associated with them. *bark* and *meow* are two functions that you have defined in your code.

``` js

  //
  // Assign routes to an object literal.
  //
  var routes = { 
    //
    // a route which assigns the function `bark`.
    //
    '/dog': bark,
    //
    // a route which assigns the functions `meow` and `scratch`.
    //
    '/cat': [meow, scratch]
  };

  var router = Router(routes); // Instantiate the router.

```

<a name="adhoc-routing"></a>
## Adhoc Routing

<a name="configuration"></a>
## Configuration

<a name="url-matching"></a>
## URL Matching

``` js

  var router = Router({
    //
    // given the route '/dog/yella'.
    //
    '/dog': {
      '/:color': {
        //
        // this function will return the value 'yella'.
        //
        on: function (color) { console.log(color) }
      }
    }
  });

```

Routes can sometimes become very complex, `simple/:tokens` don't always suffice. Director supports regular expressions inside the route names. The values captured from the regular expressions are passed to your listener function.

``` js

  var router = Router({ 
    //
    // given the route '/hello/world'.
    //
    '/hello': {
      '/(\\w+)': {
        //
        // this function will return the value 'world'.
        //
        on: function (who) { console.log(who) }
      }
    }
  });

```

``` js

  var router = Router({
    //
    // given the route '/hello/world/johny/appleseed'.
    //
    '/hello': {
      '/world/?([^\/]*)\/([^\/]*)/?': function (a, b) {
        console.log(a, b);
      }
    }

  });

```

<a name="url-params"></a>
## URL Parameters

<a name="async-routing"></a>
## Async Routing

<a name="route-recursion"></a>
## Route Recursion

Can be assigned the value of `forward` or `backward`. The recurse option will determine the order in which to fire the listeners that are associated with your routes. If this option is NOT specified or set to null, then only the listeners associated with an exact match will be fired.

### No recursion, with the URL /dog/angry

``` js

  var routes = {
    '/dog': {
      '/angry': {
        //
        // Only this method will be fired.
        //
        on: growl
      },
      on: bark
    }
  };

  var router = Router(routes);

```

### Recursion set to `backward`, with the URL /dog/angry

``` js

  var routes = {

    '/dog': {
      '/angry': {
        //
        // This method will be fired first.
        //
        on: growl 
      },
      //
      // This method will be fired second.
      //
      on: bark
    }
  };

  var router = Router(routes).configure({ recurse: 'backward' });

```

### Recursion set to `forward`, with the URL /dog/angry

``` js

  var routes = {
    '/dog': {
      '/angry': {
        //
        // This method will be fired second.
        //
        on: growl
      },
      //
      // This method will be fired first.
      //
      on: bark
    }
  };

  var router = Router(routes).configure({ recurse: 'forward' });

```

### Breaking out of recursion, with the URL /dog/angry

``` js

  var routes = {

    '/dog': {
      '/angry': {
        //
        // This method will be fired first.
        //
        on: function() { return false; } 
      },
      //
      // This method will not be fired.
      //
      on: bark 
    }
  };
  
  //
  // This feature works in reverse with recursion set to true.
  //
  var router = Router(routes).configure({ recurse: 'backward' });

```

<a name="instance-methods"></a>
## Instance methods

### configure(settings)
* `on` {Function} or {Array} - A callback or list of callbacks that will fire on every route.
* `after` {Function} or {Array} - A callback or list of callbacks that will fire after every route.
* `recurse` {String} - Determines the order in which to fire the listeners that are associated with your routes. can be set to `backward` or `forward`.
* `resource` {Object} - An object literal of function declarations.
* `notfound` {Function} or {Array} - A callback or a list of callbacks to be called when there is no matching route.

### param(token, matcher)

### on(method, path, route)

### path(path, routesFn)

### dispatch(method, path[, callback])

### mount(routes, path)

<a name="faq"></a>
# Frequently Asked Questions

## What About SEO?

Is using a client side router a problem for SEO? Yes. If advertising is a requirement, you are probably building a "Web Page" and not a "Web Application". Director on the client is meant for script-heavy Web Applications.

## Is Director compatible with X?

Director is known to be Ender.js compatible. However, the project still needs solid cross-browser testing.

# Licence

(The MIT License)

Copyright (c) 2010 Nodejitsu Inc. <http://www.twitter.com/nodejitsu>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

[0]: http://github.com/hij1nx/director